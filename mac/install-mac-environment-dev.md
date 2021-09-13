#install brew for mac

we need to install Homebrew. Homebrew is a package manager that every MacOS user should have. Installing it won’t hurt anything and you’ll definitely need it for this guide. The newest version of Homebrew has been built natively for the M1 Architecture. When you run this, you should see `arm` files getting downloaded. This is how you’ll know if a package is compiled natively for M1 / ARM Apple Silicon.

This Homebrew installation method is automated, but if you’re security minded (and should be), you can download the installer script and examine it before running it.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

https://brew.sh/

#### #Installing Nginx, PHP and Mysql on Apple M1, MacOS 11 Big Sur

This guide will walk you through installing and configuring nginx, PHP and Mysql optimized for MacOS Big Sur on Apple Silicon - M1 Arm processors.

### #OpenSSL

We’ll need to install OpenSSL to use SSL certificates in nginx. This doesn’t break any existing SSL functionality when installed via Homebrew.

```
brew install openssl
```

### #MySQL

Install MySQL.

```
brew install mysql
brew services start mysql  // this command install the latest version of mysql
brew services list
or 
// if you want to use any version on mysql 
brew services start mysql@5.7 
```

after you already install mysql they will show the instruction for you to export path via echo command or you can go to export path in .zshrc 

```
vi ~/.zshrc
```

Add this code

```
export PATH="/opt/homebrew/opt/mysql@5.7/bin:$PATH"
```

Next, update your my.cnf

```
vi /opt/homebrew/etc/my.cnf
```

add this code user=root as default and no password

```
# Default Homebrew MySQL server config
[mysqld]
# Only allow connections from localhost
bind-address = 127.0.0.1
user=root
```

Now, secure using the password password and then restart.

```
mysql_secure_installation // you can change new password if you want but defualt no password
brew services restart mysql
```

Finally command test to login mysql 

```
mysql -uroot
```

### #PHP

Don’t use the default homebrew core tap for PHP. Use shivammathur/php recomment.

```
brew tap shivammathur/php
brew install shivammathur/php/php@7.4
or 
brew install php@7.4 // if you want to use other version of php you can change php@7.x.x
```

after you already install php they will show the instruction for you to export path via `echo` command  or you can go to export path in .zshrc 

```
vi ~/.zshrc
```

Add this code

```
export PATH="/opt/homebrew/opt/php@7.4/bin:$PATH"
```

You can verify the new binary is M1 native by running `file`. The important thing to note is that it is `arm64`.

```
hostname:~ username$ file /opt/homebrew/bin/php
/opt/homebrew/bin/php: Mach-O 64-bit executable `arm64`
```

Next, set PHP 7.4 as your default php CLI version.

```
brew unlink php
brew link --overwrite --force php@7.4
```

Now, for each version update the php-fpm you will need a unique port. Change the ports of each php-fpm to match its php version number. For example, [php@7.4](mailto:php@7.4) I use port 9074.

Also, you will want php-fpm to run with `your user account` and not `_www`.

##### #Noted 

if you want to change unique php port from 9000 to other port number but if not just leave default 9000

```
vi /opt/homebrew/etc/php/7.4/php-fpm.d/www.conf
```

```
# default
user = _www
group = _www
listen = 127.0.0.1:9000

# change to 
user = <your_username>
group = staff
listen = 127.0.0.1:9074
```

Optionally, before starting php-fpm, if you want to make edits to a php.ini file now is the time. For example, you might want to increase the upload_max_filesize and post_max_size to 10M.

```
/opt/homebrew/etc/php/7.2/php.ini
/opt/homebrew/etc/php/7.3/php.ini
/opt/homebrew/etc/php/7.4/php.ini
/opt/homebrew/etc/php/8.0/php.ini
```

Once you are ready, start up php-fpm for each version.

```
sudo brew services start php@7.4
```

Check that you have processes running and validate your ports are correct.

```
sudo lsof -Pn | grep php-fpm
```

### #Redis

```
brew install redis
brew services start redis
redis-server
```

Install redis extension 

```
sudo pecl install redis
```

This command will automatically to add `extension=redis.so`  to php.ini

```
php -v // to see php version
php -m // to see php extension
php --ini // to see php direct path via load php.ini file.
sudo lsof -i -n -P | grep 80 or  xxxx // to see which http use port ex: nginx  use port 80 
```

### #Nginx

```
brew install nginx
brew services start nginx
sudo nginx
```

Now, test the install is working. by defualt port of nginx is 8080

```
curl http://localhost:8080
```

Next let’s change the default settings.

```
vi /opt/homebrew/etc/nginx/nginx.conf  // or //
code /opt/homebrew/etc/nginx/nginx.conf // if you use vscode ide
```

add the  `server_names_hash_bucket_size 64;` inside any place in  `http {}`

inside server `server {}`

From 

```
listen 8080;
server_name  localhost;
index index.html;
```

To

```
 listen       80;
 server_name  localhost;
 root "/Users/your user name/Sites/";
 index index.html index.htm index.php;

#access_log  logs/host.access.log  main;

location / {
autoindex on;
index  index.html index.htm;
try_files $uri $uri/ =404;
}
```

Next, add a FastCGI gateway to php-fpm on the default server. The latest version of php installed is best. For other servers, you can set the version of PHP to the project requirement.

the same inside server 

```
location ~ \.php$ {
  fastcgi_pass   127.0.0.1:9000; // if you use other php port, you change ex: 127.0.0.1:9998;
  fastcgi_index  index.php;
  fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name; // important need to add
  include        fastcgi_params;	
  #fastcgi_pass unix:/run/php/php7.0-fpm.sock;
}
```

Optionally, add the Universal Coded Character Set for Unicode  inside `server {}`

```
charset utf-8;
```

Note full code

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server_names_hash_bucket_size 64;

    server {
        listen       80;
        server_name  localhost;
        root "/Users/your user name/Sites/";
        index index.html index.htm index.php;

        #access_log  logs/host.access.log  main;

        location / {
            autoindex on;
            index  index.html index.htm;
            try_files $uri $uri/ =404;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;	
            #fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include servers/*;
}
```

Don't forget to add index.php in your document root 

```
root "/Users/your user name/Sites/"; 
//this is depend on you location ducoment root you want to store your project, make sure you directory is correctly
```

Ex: create a folder name Sites in you /Users/abc/Sites/index.php

```
<?php
phpinfo();
?>
```

> just make sure you local if you use default brew directory if you don't want to use like above code.
>
> Next, we edit the real index.html file used by nginx. So, replace the index.html with an index.php file. Then, and some php code to make sure everything is working.
>
> mv /opt/homebrew/var/www/index.html /opt/homebrew/var/www/index.php
> vi /opt/homebrew/var/www/index.php

Now reload nginx:

```
sudo nginx -s reload
brew services restart nginx
curl http://localhost
```

To add more servers you can go to the nginx servers directory, /opt/homebrew/etc/nginx/servers, and add them there as individual files. Here is a basic template.

inside the end of file nginx.conf you will saw include code     `include servers/*;` so this is a place you want to add more servce for your any project, inside the /opt/homebrew/etc/nginx/ directory you can create folder name `servers`  to store all vhost server for your project. Note if don't saw servers folder name just create new one.

```
code /opt/homebrew/etc/nginx/ // open vscode to create new folder name for your project
```

Next, create any file name you want or name a file to your project name ex: xxx.conf, xxx.com.

```
code /opt/homebrew/etc/nginx/servers/xxx.conf
or 
vi /opt/homebrew/etc/nginx/servers/xxx.conf
```

Finally add code inside file name. if you have multi project just do the same below:

```
server {
    listen 80;
    server_name xxx.com; // this is server name 
    root "/Users/your username /Sites/hj-gadmin-api/public/"; // this is laravel directory
    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
		autoindex on;
    }
    
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000; // php port 9000 if you change php port just change here
        fastcgi_index  index.php;     // load php index
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name; // load document root
        include        fastcgi_params;	
        #fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    }
    charset utf-8;
	
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
    location ~ /\.ht {
        deny all;
    }
}
```

```
vi /private/etc/hosts
or
code /private/etc/hosts
```

```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost
127.0.0.1      xxx.com   #this use in nginx!  
127.0.0.1      xxx.com   #this use in nginx!  
```

