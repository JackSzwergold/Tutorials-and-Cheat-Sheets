# Cheat Sheet - Apache - Load Balancer Related Items

By Jack Szwergold, October 18, 2015

The purpose of this cheat sheet is to explain the basic concepts behind using an Apache-based load balancer. 

In this setup we are using the same physical server with balancer members running on ports `8001` and `8002` just for an example’s sake. This setup is strictly for proof-of-concept and testing purposes. In a real world implementation, the balancer—and each of the members of the load balancer—would be it’s own separate server thus balancing the load across different servers.

***

## Apache load balancer prerequisites and related items.

Activate proxy related modules in Apache:

	sudo a2enmod proxy proxy_http proxy_balancer

Get a summary of the `VirtualHost` configurations for Apache:
 
    sudo apachectl -S

Check the list of sites that are available on the server:

    ls -la /etc/apache2/sites-available/

Check the list of sites that are enabled on the server:

    ls -la /etc/apache2/sites-enabled/

## Create the main load balancer web server.

##### Create the document root for the virtual host.

Create the document root directory like this:

    sudo mkdir -p /var/www/sandbox.local/site

And add a simple `index.php` page to it like this:

    sudo nano /var/www/sandbox.local/site/index.php

Then paste this content to the `index.php` file:

	<?php
	echo "Balancer - " . $_SERVER['SERVER_NAME'] . ":" . $_SERVER['SERVER_PORT'];
	?>

Note that if you can actually read that file while the balancer is active, something went wrong. That file should never be accessible unless something is seriously misconfigured with the balancer. The only content coming through the balancer should be the content piped in from the nodes.

##### Create the virtual host configuration.

Create the virtual host configuration for the `sandbox.local` load balancer that will run on port `80`:

    sudo nano /etc/apache2/sites-available/sandbox.local.conf

And add these contents to that file:

	<VirtualHost *:80>
	  DocumentRoot /var/www/sandbox.local/site
	  ServerName sandbox.local
	  ServerAlias sandbox.local

      ErrorLog /var/log/apache2/sandbox.local.error.log
      CustomLog /var/log/apache2/sandbox.local.access.log combined

	  ProxyRequests Off
	  ProxyPreserveHost On

	  <Proxy *>
	    Order deny,allow
	    Allow from all
	  </Proxy>

	  ProxyPass /balancer-manager !
	  ProxyPass / balancer://simple_cluster/ stickysession=BALANCEID nofailover=On

	  # ProxyPassReverse / http://sandbox.local:8001/
	  # ProxyPassReverse / http://sandbox.local:8002/

	  <Proxy balancer://simple_cluster>
	    BalancerMember http://sandbox.local:8001/
	    BalancerMember http://sandbox.local:8002/
	    ProxySet lbmethod=byrequests
	    # ProxySet lbmethod=bytraffic
	    # ProxySet lbmethod=bybusyness
	  </Proxy>

	  <Location /balancer-manager>
	    SetHandler balancer-manager

	    Order Deny,Allow
	    Deny from all
	    Allow from 127.0.0.1 ::1
	    Allow from localhost
	    Allow from 192.168
	    Allow from 10
	    Satisfy Any

	  </Location>

	</VirtualHost>

##### Enable the virtual host configuration.

Then enable this virtual host configuration by running this `a2ensite` command:

	sudo a2ensite sandbox.local.conf
	
And restart Apache for the config to take effect:

    sudo service apache2 restart

If you need to disable that virtual host configuration, run this `a2dissite` command:

    sudo a2dissite sandbox.local.conf

## Creating the first (port 8001) node web server.

##### Create the document root for the virtual host.

Create the document root directory like this:

    sudo mkdir -p /var/www/sandbox_port8001.local/site

And add a simple `index.php` page to it like this:

    sudo nano /var/www/sandbox_port8001.local/site/index.php

Then paste this content to the `index.php` file:

	<?php
	echo "Node 1 - " . $_SERVER['SERVER_NAME'] . ":" . $_SERVER['SERVER_PORT'];
	?>

##### Create the virtual host configuration.

Create the virtual host configuration for the second node of `sandbox.local` that will run on port `8001`:

    sudo nano /etc/apache2/sites-available/sandbox_port8001.local.conf

And add these contents to that file:

    NameVirtualHost *:8001
    Listen 8001
	<VirtualHost *:8001>
	  DocumentRoot /var/www/sandbox_port8001.local/site
	  ServerName sandbox.local
	  ServerAlias sandbox.local
	
	  ErrorLog /var/log/apache2/sandbox_port8001.local.error.log
	  CustomLog /var/log/apache2/sandbox_port8001.local.access.log combined
	
	  <Directory "/var/www/sandbox_port8001.local/site">
	    Options FollowSymLinks
	
	    Order Allow,Deny
	    Allow from all
	    Satisfy Any
	
	  </Directory>
	
	  # 2014-01-11: Compression JSON output.
	  <IfModule mod_deflate.c>
	    AddOutputFilterByType DEFLATE application/json
	  </IfModule>
	
	</VirtualHost>

##### Enable the virtual host configuration.

Then enable this virtual host configuration by running this `a2ensite` command:

    sudo a2ensite sandbox_port8001.local.conf

And restart Apache for the config to take effect:

    sudo service apache2 restart

If you need to disable that virtual host configuration, run this `a2dissite` command:

    sudo a2dissite sandbox_port8001.local.conf

## Creating the second (port 8002) node web server.

##### Create the document root for the virtual host.

Create the document root directory like this:

    sudo mkdir -p /var/www/sandbox_port8002.local/site

And add a simple `index.php` page to it like this:

    sudo nano /var/www/sandbox_port8002.local/site/index.php

Then paste this content to the `index.php` file:

	<?php
	echo "Node 2 - " . $_SERVER['SERVER_NAME'] . ":" . $_SERVER['SERVER_PORT'];
	?>

##### Create the virtual host configuration.

Create the virtual host configuration for the second node of `sandbox.local` that will run on port `8002`:

    sudo nano /etc/apache2/sites-available/sandbox_port8002.local.conf

And add these contents to that file:

    NameVirtualHost *:8002
    Listen 8002
	<VirtualHost *:8002>
	  DocumentRoot /var/www/sandbox_port8002.local/site
	  ServerName sandbox.local
	  ServerAlias sandbox.local
	
	  ErrorLog /var/log/apache2/sandbox_port8002.local.error.log
	  CustomLog /var/log/apache2/sandbox_port8002.local.access.log combined
	
	  <Directory "/var/www/sandbox_port8002.local/site">
	    Options FollowSymLinks
	
	    Order Allow,Deny
	    Allow from all
	    Satisfy Any
	
	  </Directory>
	
	  # 2014-01-11: Compression JSON output.
	  <IfModule mod_deflate.c>
	    AddOutputFilterByType DEFLATE application/json
	  </IfModule>
	
	</VirtualHost>

##### Enable the virtual host configuration.

Then enable this virtual host configuration by running this `a2ensite` command:

    sudo a2ensite sandbox_port8002.local.conf

And restart Apache for the config to take effect:

    sudo service apache2 restart

If you need to disable that virtual host configuration, run this `a2dissite` command:

    sudo a2dissite sandbox_port8002.local.conf

***

*Cheat Sheet - Apache - Load Balancer Related Items (c) by Jack Szwergold*

*This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC-BY-NC-SA-4.0).*