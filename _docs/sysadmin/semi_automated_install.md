---
title: Semi-Automated Install
category: System Administrator
order: 2
---

_FIXME: WE WILL RENAME THIS FILE TO "MANUAL SETUP OF LIVE SERVER" (OR
SIMILAR), ONCE WE HAVE REMOVED ALL OF THE MANY OUTDATED FILES NAMED
"SETUP"._


1. [Install Ubuntu 14.04 server edition](sysadmin/server_os)


2. After installing the operating system, clone the git repository:  

   ```
   mkdir -p /usr/local/submitty  
   git clone https://github.com/Submitty/Submitty.git /usr/local/submitty/GIT_CHECKOUT_Submitty
   ```

3. Run the automated portion of the install  

   ```
   cd /usr/local/submitty/GIT_CHECKOUT_Submitty  
   ./.setup/install_system.sh
   ```
   Unless you have a database server already set up, it is practical to use localhost.
   We use hsdbu for the database user, but any userid that is not used for something else will suffice.
   Do not enable debugging unless you are developing code on a non-production machine.

4. Clean up Apache configs

   You may wish to first disable or remove the default configurations
   to prevent unintended access to the web server (don’t do this if
   the default site is already in use)

   ```
   rm /etc/apache2/sites-enabled/000-default.conf 
   rm /etc/apache2/sites-enabled/default-ssl.conf
   rm /etc/apache2/sites-available/000-default.conf
   rm /etc/apache2/sites-available/default-ssl.conf
   ```

   You may also want to comment out the directory specific portions of
   ``` /etc/apache2/apache2.conf ``` so that you do not risk
   configuration conflicts with your other configurations.  (Things
   that begin with Directory and end with /Directory).


   Edit ``` /etc/apache2/conf-enabled/security.conf ``` and set the
   following options to limit the information the server gives to
   potential hackers:

   ```
   ServerTokens Prod
   ServerSignature Off
   ```
5. From checked out GIT repository:
  
    ```cp Docs/sample_apache_config /etc/apache2/sites-available/submit-ssl.conf```  
    ```cp Docs/hwgrading.conf /etc/apache2/sites-available/hwgrading.conf```  
    ```cp Docs/cgi.conf /etc/apache2/sites-available/cgi.conf```  

    and update the ip address for the VirtualHost, ServerAdmin,
    ServerName, DocumentRoot, paths, and SSL key locations as needed.
    Use

    ```a2ensite submit-ssl```  
    ```a2ensite hwgrading```  
    ```a2ensite cgi```  

    to enable the configurations.

6. Edit PHP Settings
   Edit `/etc/php5/cgi/php.ini`  and find the entry for disable_functions and prepend the list of disabled functions with:
   ```
popen,pclose,proc_open,chmod,php_real_logo_guid,php_egg_logo_guid,php_ini_scanned_files,php_ini_loaded_file,readlink,symlink,link,set_file_buffer,proc_close,proc_terminate,proc_get_status,proc_nice,getmyuid,getmygid,getmyinode,putenv,get_current_user,magic_quotes_runtime,set_magic_quotes_runtime,import_request_variables,ini_alter,stream_socket_client,stream_socket_server,stream_socket_accept,stream_socket_pair,stream_get_transports,stream_wrapper_restore,mb_send_mail,openlog,syslog,closelog,pfsockopen,posix_kill,apache_child_terminate,apache_get_modules,apache_get_version,apache_lookup_uri,apache_reset_timeout,apache_response_headers,virtual,system,phpinfo,exec,shell_exec,passthru,
   ```

7. Edit the sites in `/etc/apache2/sites-available/` to point to your SSL certificate(s).
   (Using snake oil certificates will work, but you may run into problems with browsers not trusting them, so we recommend getting certificates from a recognized Certificate Authority.)

8. You need to either comment out the system call to validate.auth.pl in **/var/local/submitty/bin/authonly.pl** and
   **new.svn.user.pl** or populate **/var/local/submitty/instructors/valid** with a list of valid userids.  
   If all valid users already have an account on the machine, one way to do this is: 

   ```
   ls /home > /var/local/sumitty/instructors/valid`
   ```

9. Set up ssh keys for hwcron to be able to connect to the subversion server without a password  

   ```
   su hwcron
   ssh-keygen -t rsa -b 4096 
   ssh-copy-id hwcron@<svnhost>    
   ```

10. Add the postgres user to the shadow group on your database machine so that
    it can access the system file for authentication

    ```
    adduser postgres shadow
    ```

11. Edit `/root/bin/top.txt` and `/root/bin/bottom.txt` to make sure they
    are pointed at the right path, using the right SSL keys, and have
    any customizations you need.  (edit `SSLCertificateFile`,
    `SSLCertificateKeyFile`, and comment out `SSLCertificatChainFile` in
    `bottom.txt` if you are using a snakeoil certificate)


12. Edit `/root/bin/gen.middle` to make sure `$dir` is set correctly, the
    `AuthName` reflects what you use, and that the "require group" line
    has the right group name for your course.

13. Test apache config with:  `apache2ctl -t` 
    If everything looks ok, restart apache with:  `service apache2 restart'