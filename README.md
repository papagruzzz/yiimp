<h1 align="center"> Yiimp - Yaamp Fork </h1>

<h3 align="left"> Requirements </h3>

* Linux
* Mysql
* Php 7.3
* Memcached
* Nginx (Recommended)
    - You can also use lighttpd or apache

<h2 align="left"> Configuration</h2>
<h3 align="center">Nginx </h3>

Use this Config on nginx:


	location / {
		try_files $uri @rewrite;
	}

	location @rewrite {
		rewrite ^/(.*)$ /index.php?r=$1;
	}

	location ~ \.php$ {
		fastcgi_pass unix:/var/run/php5-fpm.sock;
		fastcgi_index index.php;
		include fastcgi_params;
	}


<h3 align="center"> Apache </h3>

It should be something like that (already set in web/.htaccess):

	RewriteEngine on

	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^(.*) index.php?r=$1 [QSA]

<h3 align="center"> Lighttpd </h3>

use the following config:

	$HTTP["host"] =~ "yiimp.ccminer.org" {
	        server.document-root = "/var/yaamp/web"
	        url.rewrite-if-not-file = (
			"^(.*)/([0-9]+)$" => "index.php?r=$1&id=$2",
			"^(.*)\?(.*)" => "index.php?r=$1&$2",
	                "^(.*)" => "index.php?r=$1",
	                "." => "index.php"
	        )

		url.access-deny = ( "~", ".dat", ".log" )
	}

<h2 align="left"> Setup Instructions </h2>

For the database, import the initial dump present in the sql/ folder

Then, apply the migration scripts to be in sync with the current git, they are sorted by date of change.

Your database need at least 2 users, one for the web site (php) and one for the stratum connections (password set in config/algo.conf).



The recommended install folder for the stratum engine is /var/stratum. Copy all the .conf files, run.sh, the stratum binary and the blocknotify binary to this folder. 

Some scripts are expecting the web folder to be /var/web. You can use directory symlinks...


Add your exchange API public and secret keys in these two separated files:

	/etc/yiimp/keys.php - fixed path in code
	web/serverconfig.php - use sample as base...

You can find sample config files in web/serverconfig.sample.php and web/keys.sample.php

This web application includes some command line tools, add bin/ folder to your path and type "yiimp" to list them, "yiimp checkup" can help to test your initial setup.
Future scripts and maybe the "cron" jobs will then use this yiic console interface.

You need at least three backend shells (in screen) running these scripts:

	web/main.sh
	web/loop2.sh
	web/block.sh

Start one stratum per algo using the run.sh script with the algo as parameter. For example, for x11:

	run.sh x11

Edit each .conf file with proper values.

Look at rc.local, it starts all three backend shells and all stratum processes. Copy it to the /etc folder so that all screen shells are started at boot up.

All your coin's config files need to blocknotify their corresponding stratum using something like:

	blocknotify=blocknotify yaamp.com:port coinid %s

On the website, go to http://server.com/site/adminRights to login as admin. You have to change it to something different in the code (web/yaamp/modules/site/SiteController.php). A real admin login may be added later, but you can setup a password authentification with your web server, sample for lighttpd:

	htpasswd -c /etc/yiimp/admin.htpasswd <adminuser>

and in the lighttpd config file:

	# Admin access
	$HTTP["url"] =~ "^/site/adminRights" {
	        auth.backend = "htpasswd"
	        auth.backend.htpasswd.userfile = "/etc/yiimp/admin.htpasswd"
	        auth.require = (
	                "/" => (
	                        "method" => "basic",
	                        "realm" => "Yiimp Administration",
	                        "require" => "valid-user"
	                )
	        )
	}

And finally remove the IP filter check in SiteController.php

Using MySQLTuner:

sudo apt-get install mysqltuner
wait at least 24h
in cli: mysqltuner

There are logs generated in the /var/stratum folder and /var/log/stratum/debug.log for the php log.

More instructions coming as needed.


There a lot of unused code in the php branch. Lot come from other projects I worked on and I've been lazy to clean it up before to integrate it to yaamp. It's mostly based on the Yii framework which implements a lightweight MVC.

	http://www.yiiframework.com/


Credits:

Thanks to globalzon to have released the initial Yaamp source code.

--

You can support this project donating 

tpruvot :

	BTC : 1Auhps1mHZQpoX4mCcVL8odU81VakZQ6dR

xiaolin1579 :

	BTC : 1Hfa7BBHejzGj4CNRpV2Lh4xYGizX8c1A5
	
	BTC [BEP20] : 0xf6e4e1ce8b3801a612cfff40f29116fbf595b13e
	
	DOGE : D6oP3WPygJ4NR26XxfFydUsCiNS4oX9rqb
	
	USDT [BEP20] : 0xf6e4e1ce8b3801a612cfff40f29116fbf595b13e
	
	USDT [TRC20] : TF6vSbcFFGtwSRgvH1JqkQuVb9J4cszpGc
	
	RXD : 1N8WZpwSPaFvbaSMzDrPbLqbfM8tLVE87e

