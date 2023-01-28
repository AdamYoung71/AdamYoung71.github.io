---
layout:	post
title:  "Web Cache Deception Attack Simulation"
date: 2022-3-11 21:00
author: "Adam"
header-img: "img/post-bg-web.jpg"
catalog: true
tags:
    - Web
---


# Attack
## Environment
- Platform: Ubuntu Linux 20.04 64bit virtual machine.
- Softwares:
	- Web server: Apache2 + MySQL + PHP
	- Cache server: Varnish 6.6


## Environment setup
### Step 1: Install Apache2 web server, PHP and MySQL
To cache webpages, we first need to have a web server. In this case, we chose the commonly used web server Apache2. The command to install:
`$ sudo apt install apache2`
Once the installation has finished, we can start Apache2 by running:
`$ sudo systemctl start apache2`
Because our website requires user login, we use MySQL as our database and uses PHP to connect the database and the web server **[uncertain about this]**

### Step 2: Install Varnish cache service
To enable webpage caching, a caching server is needed. In this case, we chose Varnish. This time, we built Varnish 6.6 from source.

Then we can start varnish with the following command where `default.vcl` is the configuration we use.
`$ sudo varnishd -f /etc/varnish/default.vcl`

### Step 3: Apache and Varnish configuration
By default, Apache server listens on HTTP port 80 for all incoming connections. Since we intended to place Varnish in the middle of the connection to forward all the requests, we need to configure the Varnish to listen to port 80 and Apache to listen to a different port, in this case, is port 8080.

To configure Apache HTTP listening port, we edited the file:
`$ sudo vim /etc/apache2/ports.conf`
<!--screenshot here-->
Also we need to change the default virtual host of apache:
`$ sudo vim /etc/apache2/sites-emabled/000-default.conf`
<!--screenshot here-->

### Step 4: Configure Varnish to listen to port 80
Now restart Apache and we can access our website via port 8080. The next step is to configure the Varnish to listen to port 80 so that it can forward HTTP requests to the Apache web server and also enable webpage caching. Edit the following file:
`$ sudo vim /etc/default/varnish`
Change the value of `DAEMON_OPTS` to `-a :80`. Then, in file `/etc/varnish/default.vcl`, change the `backend_default` entry to the local IP address and port to 80.
Finally, edit the file `/lib/systemd/system/varnish.service` and modify the port `ExecStart` from the default port 6081 to 80.
Now we need to restart all the services to make all the configuration come to effect.
``` shell
$ sudo systemctl restart apache2
$ sudo systemctl daemon-reload
```
And we can easily test if the services are running properly using the command where `10.0.2.15` is the IP address of this machine.
`$ curl -I 10.0.2.15`
<!screenshot here>

### Step 5: Configure Varnish for webpage cache
In order to enable webpage cache, we need some configuration of the Varnish.
Because we will use `curl` to determine if the request is a "Hit" or a "Miss", we add the following rules to the Varnish configuration file.
<!--code here-->
Also, in real world settings, there must be some rules specifying what kind of pages or elements should be cached or should not be cached. In our website, all the pages are php files and the profile.php contains sensitive information about the user. So we set a rule that all pages with extension type php should not be cached.
``` python
if (req.url ~ "^[^?]\.(php)(\?.)?$") {
return (pass);
}
```
On the other hand, some of the elements should be cached to accelerate the request, so we have another rule specifying some types of files that are commonly cached, such as the css files and jpg files.
``` python
if (req.url ~ "^[^?]\.(css|jpg|js|gif|png|xml|flv|gz|txt|...)(\?.)?$") {
  return (hash);
}
```
Till now, we are basically ready to perform the attack.

## Attack
In this web cache deception attack, we will try to illustrate the whole scenario from the attacker and the victim aspect.

### Step 1: Information gathering
The victim: 
Has an account on the website, the account has sensitive information about the victim, namely: phone number, email, etc.

The attacker:
Would like to steal personal information about the victim. It first registers its own account, by doing this, it knows the possible information it can get as well as the website sub domain it is going to make use of, the /profile.php.

### Step 2: Path forgery and phishing attack
Because of the "Path Confusion", the attacker forges a link requesting for some non-exist files, for example `10.0.2.15/profile.php/nothing.css`, and tries to lure the victim to click this link. By default, `/profile.php` would not be cached by Varnish based on the first rule mentioned, however, by requesting this path, Varnish would reckon it as a `css` file, thus it will cache the page because of the second rule. 
Now the attacker can access the same site `10.0.2.15/profile.php/nothing.css`, and Varnish will directly return the cached page along with all the sensitive information within it to the attacker.
