---
layout:	post
title:  "Web Cache Deception Attack Simulaton"
date: 2022-3-16 21:00
author: "Adam"
header-img: "img/post-bg-web.jpg"
catalog: true
tags:
    - Web 
---


# Instruction
### Environment
- Platform: Ubuntu Linux 20.04 64bit virtual machine.
- Softwares:
	- Web server: Apache2 + MySQL + PHP
	- Cache server: Varnish 6.6

### Attack
1. Login the virtual machine with both username and password set to `groupn`.

2. Open a terminal using `Ctrl+Alt+T`

3. Check the IP address of the machine, by default it should be `10.0.2.15`.
	`$ ifconfig`
	
4. To start Apache2 and Varnish services, run the following commands. (`default.vcl` is the main configuration file of Varnish)
	```
	sudo systemctl start apache2
	sudo varnishd -f /etc/varnish/default.vcl
	```
	
5. Check the status of Apache2 and Varnish
	```
	sudo systemctl status apache2
	ps -ef|grep varnishd
	```
	 ![apache-status](/Users/tingyi/Library/CloudStorage/OneDrive-UniversityCollegeLondon/Modules/COMP0055 Compyter Security II/Coursework 1/wcd/apache-status.png)
	![varnish-status](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ay24lbx8j21ph0ai40t.jpg)
	
6. Now you can access our website via Varnish. Open Firefox web browser, type `10.0.2.15` in the address bar and  return.

7. Click on the link on the page and you will be directed to the login page. Then you can login to our website with the preset credentials:
	```
	username: groupn
	password: groupn
	```
	
8. Now you are in the profile page where there are personal information such as phone numbers. And we can check the caching status of this page by entering the following command in the terminal:
	`curl -I 10.0.2.15/profile.php`
	By default, this page should not be cached, so the Cache-tag header would be `Miss`.
	![normal-page](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ay2arsk5j21l80u0abn.jpg)
	![normal-profile](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ay2hxbynj21ot0j8n1f.jpg)

9. Add an arbitrary path that unlikely exists at the end of the current path, for example `/profile.php/nothing.css`, the file type should be one of the following types: `css, jpg, js, gif, png, xml, flv, gz, txt` , then press return.
![attack-nothing](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ay35l9u1j21om0kbaed.jpg)

10. The page will change a little this time, but all the user data is still there. You can run `curl` command again.
	`curl -I 10.0.2.15/profile.php/nothing.css`
	And this time, the Cache-tag is set to `Hit`, meaning that this page is cached by Varnish. Now you can open a new private window on the browser and enter this path `10.0.2.15/profile.php/nothing.css`, and you will see that all the information in `profile.php` is now available. 
	![attack-page](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ay3gpnvfj21lp0u0gn3.jpg)
	
	![private-test](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ay3u1ffuj21lq0u0gnc.jpg)
