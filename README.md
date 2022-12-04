# Ansible-wordpress

Here, we will install WordPress in a remote server using Ansible.

For that, first we have to launch an amazon Linux instance from AWS and install Ansible in the server using the below command:

```
sudo amazon-linux-extras install ansible2 -y
```

Verify the installation by entering the below command:

```
ansible --version
```

Next, create a key file inside your working directory. 

```
vim aws.pem
```

Add your key in it and change the file permission:

```
chmod 400 aws.pem
```

Now, we will have to create a hosts file and add the remote instance which we prefer to install WordPress into it.

```
vim hosts
```

```
[your-group-name]    
Instance-IP-address ansible_user="ec2-user" ansible_port=22 ansible_ssh_private_key_file="aws.pem"
```

Check connectivity to the remote instance from the host instance by:

```
ansible -i hosts your-group-name -f 1 -m ping
```

Or

```
ansible -i hosts all -f 1 -m ping
```

## Virtualhost file creation

Create a virtualhost file in the working directory.

```
vim virtualhost.conf.tmpl
```

Add the below contents in it:

```
<virtualhost *:{{ httpd_port }}>
  
  servername {{ httpd_domain }}  
  documentroot /var/www/html/{{ httpd_domain }}  
  directoryindex index.html index.php

</virtualhost>
```

## httpd config file

Create a defauklt httpd config file in the working directory:

```
vim httpd.conf.template
```

Add the below contents in it:

```
# 
# This is the main Apache HTTP server configuration file.  It contains the 
# configuration directives that give the server its instructions. 
# See <URL:http://httpd.apache.org/docs/2.4/> for detailed information. 
# In particular, see 
# <URL:http://httpd.apache.org/docs/2.4/mod/directives.html> 
# for a discussion of each configuration directive. 
# 
# Do NOT simply read the instructions in here without understanding 
# what they do.  They're here only as hints or reminders.  If you are unsure 
# consult the online docs. You have been warned. 
# 
# Configuration and logfile names: If the filenames you specify for many 
# of the server's control files begin with "/" (or "drive:/" for Win32), the 
# server will use that explicit path.  If the filenames do *not* begin 
# with "/", the value of ServerRoot is prepended -- so 'log/access_log' 
# with ServerRoot set to '/www' will be interpreted by the 
# server as '/www/log/access_log', where as '/log/access_log' will be 
# interpreted as '/log/access_log'. 
# 
# ServerRoot: The top of the directory tree under which the server's 
# configuration, error, and log files are kept. 
# 
# Do not add a slash at the end of the directory path.  If you point 
# ServerRoot at a non-local disk, be sure to specify a local disk on the 
# Mutex directive, if file-based mutexes are used.  If you wish to share the 
# same ServerRoot for multiple httpd daemons, you will need to change at 
# least PidFile. 
# 
ServerRoot "/etc/httpd" 
# 
# Listen: Allows you to bind Apache to specific IP addresses and/or 
# ports, instead of the default. See also the <VirtualHost> 
# directive. 
# 
# Change this to Listen on specific IP addresses as shown below to 
# prevent Apache from glomming onto all bound IP addresses. 
# 
#Listen 12.34.56.78:80 
Listen {{ httpd_port }} 
# 
# Dynamic Shared Object (DSO) Support 
# 
# To be able to use the functionality of a module which was built as a DSO you 
# have to place corresponding `LoadModule' lines at this location so the 
# directives contained in it are actually available _before_ they are used. 
# Statically compiled modules (those listed by `httpd -l') do not need 
# to be loaded here. 
# 
# Example: 
# LoadModule foo_module modules/mod_foo.so 
# 
Include conf.modules.d/*.conf 
# 
# If you wish httpd to run as a different user or group, you must run 
# httpd as root initially and it will switch. 
# 
# User/Group: The name (or #number) of the user/group to run httpd as. 
# It is usually good practice to create a dedicated user and group for 
# running httpd, as with most system services. 
# 
User apache 
Group apache 
# 'Main' server configuration 
# 
# The directives in this section set up the values used by the 'main' 
# server, which responds to any requests that aren't handled by a 
# <VirtualHost> definition.  These values also provide defaults for 
# any <VirtualHost> containers you may define later in the file. 
# 
# All of these directives may appear inside <VirtualHost> containers, 
# in which case these default settings will be overridden for the 
# virtual host being defined. 
# 
# 
# ServerAdmin: Your address, where problems with the server should be 
# e-mailed.  This address appears on some server-generated pages, such 
# as error documents.  e.g. admin@your-domain.com 
# 
ServerAdmin root@localhost 
# 
# ServerName gives the name and port that the server uses to identify itself. 
# This can often be determined automatically, but we recommend you specify 
# it explicitly to prevent problems during startup. 
# 
# If your host doesn't have a registered DNS name, enter its IP address here. 
# 
#ServerName www.example.com:80 
# 
# Deny access to the entirety of your server's filesystem. You must 
# explicitly permit access to web content directories in other 
# <Directory> blocks below. 
# 
<Directory /> 
    AllowOverride none 
    Require all denied 
</Directory> 
# 
# Note that from this point forward you must specifically allow 
# particular features to be enabled - so if something's not working as 
# you might expect, make sure that you have specifically enabled it 
# below. 
# 
# 
# DocumentRoot: The directory out of which you will serve your 
# documents. By default, all requests are taken from this directory, but 
# symbolic links and aliases may be used to point to other locations. 
# 
DocumentRoot "/var/www/html" 
# 
# Relax access to content within /var/www. 
# 
<Directory "/var/www"> 
    AllowOverride None 
    # Allow open access: 
    Require all granted 
</Directory> 
# Further relax access to the default document root: 
<Directory "/var/www/html"> 
    # 
    # Possible values for the Options directive are "None", "All", 
    # or any combination of: 
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews 
    # 
    # Note that "MultiViews" must be named *explicitly* --- "Options All" 
    # doesn't give it to you. 
    # 
    # The Options directive is both complicated and important.  Please see 
    # http://httpd.apache.org/docs/2.4/mod/core.html#options 
    # for more information. 
    # 
    Options Indexes FollowSymLinks 
    # 
    # AllowOverride controls what directives may be placed in .htaccess files. 
    # It can be "All", "None", or any combination of the keywords: 
    #   Options FileInfo AuthConfig Limit 
    # 
    AllowOverride None 
    # 
    # Controls who can get stuff from this server. 
    # 
    Require all granted 
</Directory> 
# 
# DirectoryIndex: sets the file that Apache will serve if a directory 
# is requested. 
# 
<IfModule dir_module> 
    DirectoryIndex index.html 
</IfModule> 
# 
# The following lines prevent .htaccess and .htpasswd files from being 
# viewed by Web clients. 
# 
<Files ".ht*"> 
    Require all denied 
</Files> 
# 
# ErrorLog: The location of the error log file. 
# If you do not specify an ErrorLog directive within a <VirtualHost> 
# container, error messages relating to that virtual host will be 
# logged here.  If you *do* define an error logfile for a <VirtualHost> 
# container, that host's errors will be logged there and not here. 
# 
ErrorLog "logs/error_log" 
# 
# LogLevel: Control the number of messages logged to the error_log. 
# Possible values include: debug, info, notice, warn, error, crit, 
# alert, emerg. 
# 
LogLevel warn 
<IfModule log_config_module> 
    # 
    # The following directives define some format nicknames for use with 
    # a CustomLog directive (see below). 
    # 
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined 
    LogFormat "%h %l %u %t \"%r\" %>s %b" common 
    <IfModule logio_module> 
      # You need to enable mod_logio.c to use %I and %O 
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio 
    </IfModule> 
    # 
    # The location and format of the access logfile (Common Logfile Format). 
    # If you do not define any access logfiles within a <VirtualHost> 
    # container, they will be logged here.  Contrariwise, if you *do* 
    # define per-<VirtualHost> access logfiles, transactions will be 
    # logged therein and *not* in this file. 
    # 
    #CustomLog "logs/access_log" common 
    # 
    # If you prefer a logfile with access, agent, and referer information 
    # (Combined Logfile Format) you can use the following directive. 
    # 
    CustomLog "logs/access_log" combined 
</IfModule> 
<IfModule alias_module> 
    # 
    # Redirect: Allows you to tell clients about documents that used to 
    # exist in your server's namespace, but do not anymore. The client 
    # will make a new request for the document at its new location. 
    # Example: 
    # Redirect permanent /foo http://www.example.com/bar 
    # 
    # Alias: Maps web paths into filesystem paths and is used to 
    # access content that does not live under the DocumentRoot. 
    # Example: 
    # Alias /webpath /full/filesystem/path 
    # 
    # If you include a trailing / on /webpath then the server will 
    # require it to be present in the URL.  You will also likely 
    # need to provide a <Directory> section to allow access to 
    # the filesystem path. 
    # 
    # ScriptAlias: This controls which directories contain server scripts. 
    # ScriptAliases are essentially the same as Aliases, except that 
    # documents in the target directory are treated as applications and 
    # run by the server when requested rather than as documents sent to the 
    # client.  The same rules about trailing "/" apply to ScriptAlias 
    # directives as to Alias. 
    # 
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/" 
</IfModule> 
# 
# "/var/www/cgi-bin" should be changed to whatever your ScriptAliased 
# CGI directory exists, if you have that configured. 
# 
<Directory "/var/www/cgi-bin"> 
    AllowOverride None 
    Options None 
    Require all granted 
</Directory> 
<IfModule mime_module> 
    # 
    # TypesConfig points to the file containing the list of mappings from 
    # filename extension to MIME-type. 
    # 
    TypesConfig /etc/mime.types 
    # 
    # AddType allows you to add to or override the MIME configuration 
    # file specified in TypesConfig for specific file types. 
    # 
    #AddType application/x-gzip .tgz 
    # 
    # AddEncoding allows you to have certain browsers uncompress 
    # information on the fly. Note: Not all browsers support this. 
    # 
    #AddEncoding x-compress .Z 
    #AddEncoding x-gzip .gz .tgz 
    # 
    # If the AddEncoding directives above are commented-out, then you 
    # probably should define those extensions to indicate media types: 
    # 
    AddType application/x-compress .Z 
    AddType application/x-gzip .gz .tgz 
    # 
    # AddHandler allows you to map certain file extensions to "handlers": 
    # actions unrelated to filetype. These can be either built into the server 
    # or added with the Action directive (see below) 
    # 
    # To use CGI scripts outside of ScriptAliased directories: 
    # (You will also need to add "ExecCGI" to the "Options" directive.) 
    # 
    #AddHandler cgi-script .cgi 
    # For type maps (negotiated resources): 
    #AddHandler type-map var 
    # 
    # Filters allow you to process content before it is sent to the client. 
    # 
    # To parse .shtml files for server-side includes (SSI): 
    # (You will also need to add "Includes" to the "Options" directive.) 
    # 
    AddType text/html .shtml 
    AddOutputFilter INCLUDES .shtml 
</IfModule> 
# 
# Specify a default charset for all content served; this enables 
# interpretation of all content as UTF-8 by default.  To use the 
# default browser choice (ISO-8859-1), or to allow the META tags 
# in HTML content to override this choice, comment out this 
# directive: 
# 
AddDefaultCharset UTF-8 
<IfModule mime_magic_module> 
    # 
    # The mod_mime_magic module allows the server to use various hints from the 
    # contents of the file itself to determine its type.  The MIMEMagicFile 
    # directive tells the module where the hint definitions are located. 
    # 
    MIMEMagicFile conf/magic 
</IfModule> 
# 
# Customizable error responses come in three flavors: 
# 1) plain text 2) local redirects 3) external redirects 
# 
# Some examples: 
#ErrorDocument 500 "The server made a boo boo." 
#ErrorDocument 404 /missing.html 
#ErrorDocument 404 "/cgi-bin/missing_handler.pl" 
#ErrorDocument 402 http://www.example.com/subscription_info.html 
# 
# 
# EnableMMAP and EnableSendfile: On systems that support it, 
# memory-mapping or the sendfile syscall may be used to deliver 
# files.  This usually improves server performance, but must 
# be turned off when serving from networked-mounted 
# filesystems or if support for these functions is otherwise 
# broken on your system. 
# Defaults if commented: EnableMMAP On, EnableSendfile Off 
# 
#EnableMMAP off 
EnableSendfile on 
# Enable HTTP/2 by default 
# 
# https://httpd.apache.org/docs/2.4/mod/core.html#protocols 
<IfModule mod_http2.c> 
    Protocols h2 h2c http/1.1 
</IfModule> 
# Supplemental configuration 
# 
# Load config files in the "/etc/httpd/conf.d" directory, if any. 
IncludeOptional conf.d/*.conf
```

## Create a file to declare variables

Next, we havr to create a file to declare variables required for the WordPress installation:

```
vim main.vars
```

Add the below contents in it:

```
---
httpd_port: 80
httpd_domain: "wordpress.domain.com"
httpd_owner: "apache"
httpd_group: "apache"
    
mysql_packages:
  - mariadb-server
  - MySQL-python
mysql_root_password: "mysqlroot123"
mysql_extra_user: "bloguser"
mysql_extra_user_password: "bloguser123"
mysql_extra_database: "blogdb"
    
wp_url: "https://wordpress.org/wordpress-6.0.tar.gz"
```

## wp-config.php file creation

Create a wp-config.php file in the working directory and add contents as follows:

```
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', '{{  mysql_extra_database }}' );

/** Database username */
define( 'DB_USER', '{{ mysql_extra_user }}' );

/** Database password */
define( 'DB_PASSWORD', '{{ mysql_extra_user_password }}' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

Next, we will have to create two test files test.php and test.html in the working directory. For that, use the below commands:

```
echo "<?php phpinfo(); ?>" > test.php 
```

and 

```
echo "<h1><center>test.html page</center></h1>" > test.html
```

## Playbook file creation

Create a playbook file:

```
vim playbook.yml
```

Add the contents:

```
---
- name: "Wordpress Installation using Ansible"
  hosts: all
  become: true
  vars_files:
    - main.vars
  tasks:
    
    - name: "Lamp - Apache Webserver Installation"
      yum:
        name: httpd
        state: present
            
    - name: "Lamp - Installing PHP packages"
      shell: amazon-linux-extras install php7.4 -y
    
    - name: "listing PHP package in client instance"
      package_facts:
        manager: auto
        strategy: all

    - name: "checking for PHP package"
      when: "'php-fpm' in ansible_facts.packages"
      debug:
        msg: "php is available in server"

    - name: "Installing php if not available"
      when: "'php-fpm' not in ansible_facts.packages" 
      shell: amazon-linux-extras install php7.4 -y    

    - name: "Lamp - Creating httpd.conf from template"
      template:
        src: httpd.conf.tmpl
        dest: /etc/httpd/conf/httpd.conf
        owner: "{{ httpd_owner }}"
        group: "{{ httpd_group }}"
            
    - name: "Lamp - Creating Virtualhost Configuration File {{ httpd_domain }}"
      template:
        src: virtualhost.conf.tmpl 
        dest: /etc/httpd/conf.d/default.conf
        owner: "{{ httpd_owner }}"
        group: "{{ httpd_group }}"
            
    - name: "Lamp - Creating Documentroot For the virtualhost  {{ httpd_domain }}"
      file:
        path: "/var/www/html/{{ httpd_domain }}"
        state: directory
        owner: "{{ httpd_owner }}"
        group: "{{ httpd_group }}"
            
    - name: "Lamp - Copying test page into documentroot /var/www/html/{{ httpd_domain }}"
      copy:
        src: "{{ item }}"
        dest: "/var/www/html/{{ httpd_domain }}"
        owner: "{{ httpd_owner }}"
        group: "{{ httpd_group }}"
      with_items:
        - test.php
        - test.html
                
    - name: "Lamp - Restarting Apache and PHP service"
      service:
        name: "{{ item }}"
        state: restarted
        enabled: true
      with_items:
        - "httpd"
        - "php-fpm"
               
    - name: "Lamp - MariaDB-Server Installation"
      yum:
        name: "{{ mysql_packages }}"
        state: present
    
    - name: "Lamp - Restart/enable MariaDB-Server"
      service:
        name: mariadb
        state: restarted
        enabled: true
            
    - name: "Lamp - MariaDB-Server Setting Root Password"
      ignore_errors: true
      mysql_user:
        login_user: "root"
        login_password: ""
        user: "root"
        password: "{{ mysql_root_password }}"
        host_all: true
            
    - name: "Lamp - MariaDB-Server Removing Anonymous Users"
      mysql_user:
        login_user: "root"
        login_password: "{{ mysql_root_password }}"
        user: ""
        host_all: true
        state: absent
            
    - name: "Lamp - MariaDB-Server Removing Test Database"
      mysql_db:
        login_user: "root"
        login_password: "{{ mysql_root_password }}"
        name: "test"
        state: absent
            
            
    - name: "Lamp - MariaDB-Server Creating Extra Database {{ mysql_extra_database }} "
      mysql_db:
        login_user: "root"
        login_password: "{{ mysql_root_password }}"
        name: "{{ mysql_extra_database }}"
        state: present
            
    - name: "Lamp - MariaDB-Server Creating Extra User {{ mysql_extra_user }} "
      mysql_user:
        login_user: "root"
        login_password: "{{ mysql_root_password }}"
        user: "{{ mysql_extra_user }}"
        host: "%"
        password: "{{ mysql_extra_user_password }}"
        priv: '{{ mysql_extra_database }}.*:ALL'
        state: present
            
            
    - name: "Wordpress - Downloading WordPress archive file"
      get_url:
        url: "{{ wp_url }}"
        dest: /tmp/wordpress.tar.gz
    
    - name: "Wordpress - Extrating Wordpress archive"
      unarchive:
        src: /tmp/wordpress.tar.gz
        dest: /tmp/
        remote_src: true
            
    - name: "WordPress - Copying files to documentroot /var/www/html/{{ httpd_domain }}"
      copy:
        src: /tmp/wordpress/
        dest: "/var/www/html/{{ httpd_domain }}"
        owner: "{{ httpd_owner }}"
        group: "{{ httpd_group }}"
        remote_src: true
            
    - name: "Wordpress - Creating wp-config.php from template"
      template:
        src: wp-config.php.tmpl
        dest: "/var/www/html/{{ httpd_domain }}/wp-config.php"
        owner: "{{ httpd_owner }}"
        group: "{{ httpd_group }}"
            
    - name: "Post-installation restart"
      service:
        name: "{{ item }}"
        state: restarted
        enabled: true
      with_items:
        - "mariadb"
        - "httpd"
        - "php-fpm"
        
    - name: "Post-Installation Cleanup"
      file:
        path: "{{ item }}"
        state: absent
      with_items:
        - "/tmp/wordpress.tar.gz"
        - "/tmp/wordpress/"
```

## Conclusuion

Now, execute the below command to check syntax of the playbook file:

```
ansible-playbook -i hosts playbook.tmpl --syntax-check
```

If everuthing is clear, use the below command to execute the playbook file and to install wordpress in the remote server:

```
ansible-playbook -i hosts playbook.tmpl
```

After, the command is executed, the output will look like:




























