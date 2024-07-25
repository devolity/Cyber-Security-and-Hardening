# Apache httpd Web Servers

##### Apache Web Server Hardening and Security Guide

​​​![APACHE](https://geekflare.com/wp-content/uploads/2015/02/APACHE-1200x385.jpg)​

A practical guide to secure and harden Apache HTTP Server.

The Web Server is a crucial part of web-based applications. Apache Web Server is often placed at the edge of the network hence it becomes one of the most vulnerable services to attack.

Having default configuration supply much sensitive information which may help hacker to prepare for an attack the applications. The majority of web application attacks are through XSS, Info Leakage, Session Management and SQL Injection attacks which are due to weak programming code and failure to sanitize web application infrastructure.

Interesting research by [Positive Technologies](https://www.ptsecurity.com/ww-en/analytics/web-application-vulnerabilities-2018/) reveals, 52% of the scanned application had high vulnerabilities.

​![vulnerability-report](https://geekflare.com/wp-content/uploads/2015/02/vulnerability-report.png)​

In this article, I will talk about some of the best practices to secure Apache HTTP server on Linux platform.

Following are tested on Apache 2.4.x version.

* This assumes you have installed Apache on UNIX platform. If not, you can go through [the Installation guide](https://geekflare.com/apache-2-4-6-installation-on-unix/).
* I will call the Apache installation directory /opt/apache as $Web_Server throughout this guide.
* You are advised to take a backup of existing configuration file before any modification.

### Audience

This is designed for Middleware Administrator, Application Support, System Analyst, or anyone working or eager to learn Hardening & Security guidelines.

Fair knowledge of Apache Web Server & UNIX command is mandatory.

**Notes**

You require some tool to examine HTTP Headers for some of the implementation verification. There are two ways to do this.

1. Use browser inbuilt developer tools to inspect the HTTP headers. Usually, it’s under Network tab
2. Use online [HTTP response header checker tool](https://tools.geekflare.com/http-headers-test)

### Remove Server Version Banner

I would say this is one of the first things to consider, as you don’t want to expose what web server version you are using. Exposing version means you are helping hacker to speedy the reconnaissance process.

The default configuration will expose Apache Version and OS type as shown below.

​![apache-server-banner](https://geekflare.com/wp-content/uploads/2015/02/apache-server-banner.png)​

* Go to $Web_Server/conf folder
* Modify httpd.conf by using the vi editor
* Add the following directive and save the httpd.conf

```apacheconf
ServerTokens Prod
ServerSignature Off
```

Copy

* Restart apache

​`ServerSignature`​ will remove the version information from the page generated by Apache.

​`ServerTokens`​ will change Header to production only, i.e., Apache

As you can see below, version & OS information is gone.

​![apache-server-banner-masked](https://geekflare.com/wp-content/uploads/2015/02/apache-server-banner-masked.png)​

### Disable directory browser listing

Disable directory listing in a browser, so the visitor doesn’t see what all file and folders you have under root or subdirectory.

Let’s test how does it look like in default settings.

* Go to $Web_Server/htdocs directory
* Create a folder and few files inside that

```markup
# mkdir test
# touch hi
# touch hello
```

Copy

Now, let’s try to access Apache by [http://localhost/test](http://localhost/test)

​![apache-directory-listing](https://geekflare.com/wp-content/uploads/2015/02/apache-directory-listing.png)​

As you could see it reveals what all file/folders you have and I am sure you don’t want to expose that.

* Go to $Web_Server/conf directory
* Open `httpd.conf`​ using vi
* Search for Directory and change Options directive to None or –Indexes

```apacheconf
<Directory /opt/apache/htdocs>
Options -Indexes
</Directory>
```

Copy

(or)

```apacheconf
<Directory /opt/apache/htdocs>
Options None
</Directory>
```

Copy

* Restart Apache

*Note*: if you have multiple Directory directives in your environment, you should consider doing the same for all.

Now, let’s try to access Apache by [http://localhost/test](http://localhost/test)

​![disabled-directory-listing](https://geekflare.com/wp-content/uploads/2015/02/disabled-directory-listing.png)​

As you could see, it displays a forbidden error instead of showing test folder listing.

### Etag

It allows remote attackers to obtain sensitive information like inode number, multipart MIME boundary, and child process through Etag header.

To prevent this vulnerability, let’s implement it as below. This is required to fix for PCI compliance.

* Go to $Web_Server/conf directory
* Add the following directive and save the httpd.conf

```apacheconf
FileETag None
```

Copy

* Restart apache

### Run Apache from a non-privileged account

A default installation runs as nobody or daemon. Using a separate non-privileged user for Apache is good.

The idea here is to protect other services running in case of any security hole.

* Create a user and group called apache

```apacheconf
# groupadd apache
# useradd –G apache apache
```

Copy

* Change apache installation directory ownership to a newly created non-privileged user

```apacheconf
# chown –R apache:apache /opt/apache
```

Copy

* Go to $Web_Server/conf
* Modify httpd.conf using vi
* Search for User & Group Directive and change as non-privileged account apache

```apacheconf
User apache 
Group apache
```

Copy

* Save the httpd.conf
* Restart Apache

grep for running http process and ensure it’s running with apache user

```apacheconf
# ps –ef |grep http
```

Copy

You should see one process is running with root. That’s because Apache is listening on port 80 and it has to be started with root.

### Protect binary and configuration directory permission

By default, permission for binary and configuration is 755 that means any user on a server can view the configuration. You can disallow another user to get into conf and bin folder.

* Go to $Web_Server directory
* Change permission of bin and conf folder

```apacheconf
# chmod –R 750 bin conf
```

Copy

### System Settings Protection

In a default installation, users can override apache configuration using .htaccess. If you want to stop users from changing your apache server settings, you can add `AllowOverride`​ to `None`​ as shown below.

This must be done at the root level.

* Go to $Web_Server/conf directory
* Open httpd.conf using vi
* Search for Directory at a root level

```apacheconf
<Directory /> 
Options -Indexes 
AllowOverride None
</Directory>
```

Copy

* Save the httpd.conf
* Restart Apache

### HTTP Request Methods

HTTP 1.1 protocol support many request methods which may not be required and some of them are having potential risk.

Typically you may need GET, HEAD, POST request methods in a web application, which can be configured in the respective Directory directive.

Default configuration support OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT method in HTTP 1.1 protocol.

* Go to $Web_Server/conf directory
* Open httpd.conf using vi
* Search for Directory and add the following

```apacheconf
<LimitExcept GET POST HEAD>
deny from all
</LimitExcept>
```

Copy

* Restart Apache

### Disable Trace HTTP Request

By default Trace method is enabled in Apache web server.

Having this enabled can allow Cross Site Tracing attack and potentially giving an option to a hacker to steal cookie information. Let’s see how it looks like in default configuration.

* Do a telnet web server IP with listening port
* Make a TRACE request as shown below

```apacheconf
#telnet localhost 80 
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
TRACE / HTTP/1.1 Host: test
HTTP/1.1 200 OK
Date: Sat, 31 Aug 2013 02:13:24 GMT
Server: Apache
Transfer-Encoding: chunked
Content-Type: message/http 20
TRACE / HTTP/1.1
Host: test 
0
Connection closed by foreign host.
#
```

Copy

As you could see in above TRACE request, it has responded my query. Let’s disable it and test it.

* Go to $Web_Server/conf directory
* Add the following directive and save the httpd.conf

```apacheconf
TraceEnable off
```

Copy

* Restart apache

Do a telnet web server IP with listen port and make a `TRACE`​ request as shown below

```apacheconf
#telnet localhost 80
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
TRACE / HTTP/1.1 Host: test
HTTP/1.1 405 Method Not Allowed
Date: Sat, 31 Aug 2013 02:18:27 GMT
Server: Apache Allow:Content-Length: 223Content-Type: text/html; charset=iso-8859-1 <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN"> <html><head> 
<title>405 Method Not Allowed</title> </head><body> 
<h1>Method Not Allowed</h1>
<p>The requested method TRACE is not allowed for the URL /.</p> </body></html>
Connection closed by foreign host.
#
```

Copy

As you could see in above TRACE request, it has blocked my request with HTTP 405 Method Not Allowed.

Now, this web server doesn’t allow TRACE request and help in blocking Cross Site Tracing attack.

### Set cookie with HttpOnly and Secure flag

You can mitigate most of the common Cross Site Scripting attack using HttpOnly and Secure flag in a cookie. Without having HttpOnly and Secure, it is possible to steal or manipulate web application session and cookies, and it’s dangerous.

* Ensure mod_headers.so is enabled in your httpd.conf
* Go to $Web_Server/conf directory
* Add the following directive and save the httpd.conf

```apacheconf
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
```

Copy

* Restart apache

### Clickjacking Attack

Clickjacking is a well-known web application vulnerabilities.

* Ensure mod_headers.so is enabled in your httpd.conf
* Go to $Web_Server/conf directory
* Add the following directive and save the httpd.conf

```apacheconf
Header always append X-Frame-Options SAMEORIGIN
```

Copy

* Restart apache

​![apache-x-frame-options](https://geekflare.com/wp-content/uploads/2015/02/apache-x-frame-options.png)​

X-Frame-Options also support two more options which I explained [here](https://geekflare.com/secure-apache-from-clickjacking-with-x-frame-options/).

### Server Side Include

Server Side Include (SSI) has a risk of increasing the load on the server. If you have shared the environment and heavy traffic web applications you should consider disabling SSI by adding Includes in Options directive.

SSI attack allows the exploitation of a web application by injecting scripts in HTML pages or executing codes remotely.

* Go to $Web_Server/conf directory
* Open httpd.conf using vi
* Search for Directory and add Includes in Options directive

```apacheconf
<Directory /opt/apache/htdocs>
Options –Indexes -Includes
Order allow,denyAllow from all
</Directory>
```

Copy

* Restart Apache

Note: if you have multiple Directory directives in your environment, you should consider doing the same for all.

### X-XSS Protection

Cross Site Scripting (XSS) protection can be bypassed in many browsers. You could apply this protection for a web application if it was disabled by the user. This is used by a majority of giant web companies like Facebook, Twitter, Google, etc.

* Go to $Web_Server/conf directory
* Open httpd.conf using vi and add following Header directive

```apacheconf
Header set X-XSS-Protection "1; mode=block"
```

Copy

* Restart Apache

As you can see, XSS-Protection is the injected in the response header.

​![apache-xss](https://geekflare.com/wp-content/uploads/2015/02/apache-xss.png)​

### Disable HTTP 1.0 Protocol

When we talk about security, we should protect as much we can. So why do we use older HTTP version of the protocol, let’s disable them as well?

HTTP 1.0 has security weakness related to session hijacking. We can disable this by using the mod_rewrite module.

* Ensure to load mod_rewrite module in httpd.conf file
* Enable RewriteEngine directive as following and add Rewrite condition to allow only HTTP 1.1

```apacheconf
RewriteEngine On
RewriteCond %{THE_REQUEST} !HTTP/1.1$
RewriteRule .* - [F]
```

Copy

### Timeout value configuration

By default, Apache time-out value is 300 seconds, which can be a victim of Slow Loris attack and DoS. To mitigate this, you can lower the timeout value to maybe 60 seconds.

* Go to $Web_Server/conf directory
* Open httpd.conf using vi
* Add the following in httpd.conf

```apacheconf
Timeout 60
```

Copy

## SSL

Having SSL is an additional layer of security you are adding into Web Application. However, default SSL configuration leads to certain vulnerabilities, and you should consider tweaking those configurations.

### SSL Key

Breaching SSL key is hard, but not impossible. It’s just a matter of computational power and time.

As you might know, using a 2009-era PC cracking away for around 73 days you can [reverse engineer a 512-bit key](http://www.ticalc.org/archives/news/articles/14/145/145154.html).

So the higher key length you have, the more complicated it becomes to break SSL key. The majority of giant Web Companies use 2048 bit key, as below so why don’t we?

* Outlook.com
* Microsoft.com
* Live.com
* Skype.com
* Apple.com
* Yahoo.com
* Bing.com
* Hotmail.com
* Twitter.com

You can use OpenSSL to generate CSR with 2048 bit as below.

```apacheconf
openssl req -out geekflare.csr -newkey rsa:2048 -nodes -keyout geekflare.key
```

Copy

It will generate a CSR which you will need to send to a [certificate authority](https://namecheap.pxf.io/c/245992/386170/5618?u=https%3A%2F%2Fwww.namecheap.com%2Fsecurity%2Fssl-certificates.aspx) to sign it. Once you receive the signed certificate file, you can add them in httpd-ssl.conf file

```apacheconf
SSLCertificateFile #Certificate signed by authority
SSLCertificateChainFile #Certificate signer given by authority
SSLCertificateKeyFile #Key file which you generated above
```

Copy

* Restart Apache web server and try to access the URL with https

### SSL Cipher

SSL Cipher is an encryption algorithm, which is used as a key between two computers over the Internet. Data encryption is the process of converting plain text into secret ciphered codes.

It’s based on your web server SSL Cipher configuration the data encryption will take place. So it’s important to configure SSL Cipher, which is stronger and not vulnerable.

* Go to $Web_Server/conf/extra folder
* Modify SSLCipherSuite directive in httpd-ssl.conf as below to accept only higher encryption algorithms

```apacheconf
SSLCipherSuite HIGH:!MEDIUM:!aNULL:!MD5:!RC4
```

Copy

* Save the configuration file and restart apache server

Note: if you have many weak ciphers in your SSL auditing report, you can quickly reject them adding ! at the beginning.

### Disable SSL v2 & v3

SSL v2 & v3 has many security flaws, and if you are working towards penetration test or PCI compliance, then you are expected to close security finding to disable SSL v2/v3.

Any SSL v2/v3 communication may be vulnerable to a Man-in-The-Middle attack that could allow data tampering or disclosure.

Let’s implement apache web server to accept only latest TLS and reject SSL v2/v3 connection request.

* Go to $Web_Server/conf/extra folder
* Modify SSLProtocol directive in httpd-ssl.conf as below to accept only TLS 1.2+

```apacheconf
SSLProtocol –ALL +TLSv1.2
```

Copy

Once you are done with SSL configuration, it’s a good idea to test your web application with [online SSL/TLS Certificate tool](https://geekflare.com/ssl-test-certificate/) to find any configuration error.

## Mod Security

Mod Security is an open-source [Web Application Firewall](https://geekflare.com/open-source-web-application-firewall/), which you can use with Apache.

It comes as a module which you have to compile and install. If you can’t afford a commercial [web application firewall](https://geekflare.com/web-application-firewall/), this would be an excellent choice to go for it.

To provide generic web applications protection, the Core Rules use the following techniques:

* HTTP Protection – detecting violations of the HTTP protocol and a locally defined usage policy
* Real-time Blacklist Lookups – utilizes 3rd Party IP Reputation
* Web-based Malware Detection – identifies malicious web content by check against the Google Safe Browsing API.
* HTTP Denial of Service Protections – defense against HTTP Flooding and Slow HTTP DoS Attacks.
* Common Web Attacks Protection – detecting common web application security attack
* Automation Detection – Detecting bots, crawlers, scanners, and another malicious surface activity
* Integration with AV Scanning for File Uploads – identifies malicious files uploaded through the web application.
* Tracking Sensitive Data – Tracks Credit Card usage and blocks leakages.
* Trojan Protection – Detecting access to Trojans horses.
* Identification of Application Defects – alerts on application misconfigurations.
* Error Detection and Hiding – Disguising error messages sent by the server.

### Download & Installation

Following prerequisites must be installed on the server where you wish to use Mod Security with Apache. If any one of these doesn’t exist then Mod Security compilation will fail. You may use yum install on Linux or Centos to install these packages.

* apache 2.x or higher
* libpcre package
* libxml2 package
* liblua package
* libcurl package
* libapr and libapr-util package
* mod_unique_id module bundled with Apache web server

Now, let’s download the latest stable version of Mod Security 2.7.5 from [here](https://www.modsecurity.org/download.html)

* Transfer downloaded file to /opt/apache
* Extract modsecurity-apache_2.7.5.tar.gz

```apacheconf
# gunzip –c modsecurity-apache_2.7.5.tar.gz | tar xvf –
```

Copy

* Go to extracted folder modsecurity-apache_2.7.5

```apacheconf
# cd modsecurity-apache_2.7.5
```

Copy

* Run the configure script including apxs path to existing Apache

```apacheconf
# ./configure –with-apxs=/opt/apache/bin/apxs
```

Copy

* Compile & install with make script

```apacheconf
# make
# make install
```

Copy

* Once the installation is done, you would see mod_security2.so in modules folder under /opt/apache

Now this concludes, you have installed Mod Security module in existing Apache web server.

### Configuration

To use Mod security feature with Apache, we have to load mod security module in httpd.conf. The mod_unique_id module is pre-requisite for Mod Security.

This module provides an environment variable with a unique identifier for each request, which is tracked and used by Mod Security.

* Add following a line to load module for Mod Security in httpd.conf and save the configuration file

```apacheconf
LoadModule unique_id_module modules/mod_unique_id.so
LoadModule security2_module modules/mod_security2.so
```

Copy

* Restart apache web server

Mod Security is now installed!

Next thing you have to do is to install Mod Security core rule to take full advantage of its feature.

Latest Core Rule can be downloaded from following a link, which is free. [https://github.com/SpiderLabs/owasp-modsecurity-crs/zipball/master](https://github.com/SpiderLabs/owasp-modsecurity-crs/zipball/master)

* Copy downloaded core rule zip to /opt/apache/conf folder
* Unzip core rule file
* You may wish to rename the folder to something short and easy to remember. In this example, I will rename to crs.
* Go to crs folder and rename modsecurity_crs10_setup.conf.example to modsecurity_crs10_setup.conf

Now, let’s enable these rules to get it working with Apache web server.

* Add the following in httpd.conf

```apacheconf
<IfModule security2_module>
Include conf/crs/modsecurity_crs_10_setup.confInclude conf/crs/base_rules/*.conf
</IfModule>
```

Copy

In the above configuration, we are loading Mod Security main configuration file modsecurity_crs_10_setup.conf and base rules base_rules/*.conf provided by Mod Security Core Rules to protect web applications.

* Restart apache web server

You have successfully configured Mod Security with Apache!

Well done. Now, Apache Web server is protected by Mod Security web application firewall.

### Getting Started

Let’s get it started with some of the critical configurations in Mod Security to harden & secure web applications.

In this section, we will do all configuration modification in /opt/apache/conf/crs/modsecurity_crs_10_setup.conf.

We will refer /opt/apache/conf/crs/modsecurity_crs_10_setup.conf as setup.conf in this section for example purpose.

It’s important to understand what are the OWASP rules are provided for free. There are two types of rules provided by OWASP.

**Base Rules** – these rules are heavily tested, and probably false alarm ratio is less.

**Experimental Rules** – these rules are for an experimental purpose, and you may have a high false alarm. It’s important to configure, test and implement in UAT before using these in a production environment.

**Optional Rules** – these optional rules may not be suitable for the entire environment. Based on your requirement you may use them.

If you are looking for CSRF, User tracking, Session hijacking, etc. protection, then you may consider using optional rules. We have the base, optional and experimental rules after extracting the downloaded crs zip file from OWASP download page.

These rules configuration file is available in crs/base_rules, crs/optional_rules and crs/experimental_rules folder. Let’s get familiar with some of the base rules.

* modsecurity_crs_20_protocol_violations.conf: This rule is protecting from Protocol vulnerabilities like response splitting, request smuggling, using non-allowed protocol (HTTP 1.0).
* modsecurity_crs_21_protocol_anomalies.conf: This is to protect from a request, which is missing with Host, Accept, User-Agent in the header.
* modsecurity_crs_23_request_limits.conf:This rule has the dependency on application specific like request size, upload size, a length of a parameter, etc.
* modsecurity_crs_30_http_policy.conf:This is to configure and protect allowed or disallowed method like CONNECT, TRACE, PUT, DELETE, etc.
* modsecurity_crs_35_bad_robots.conf:Detect malicious robots
* modsecurity_crs_40_generic_attacks.conf:This is to protect from OS command injection, remote file inclusion, etc.
* modsecurity_crs_41_sql_injection_attacks.conf:This rule to protect SQL and blind SQL inject request.
* modsecurity_crs_41_xss_attacks.conf:Protection from Cross-Site Scripting request.
* modsecurity_crs_42_tight_security.conf:Directory traversal detection and protection.
* modsecurity_crs_45_trojans.conf:This rule to detect generic file management output, uploading of HTTP backdoor page, known signature.
* modsecurity_crs_47_common_exceptions.conf:This is used as an exception mechanism to remove common false positives that may be encountered suck as Apache internal dummy connection, SSL pinger, etc.

### Logging

Logging is one of the first things to configure so you can have logs created for what Mod Security is doing. There are two types of logging available; Debug & Audit log.

Debug Log: this is to duplicate the Apache error, warning and notice messages from the error log.

Audit Log: this is to write the transaction logs that are marked by Mod Security rule Mod Security gives you the flexibility to configure Audit, Debug or both logging.

By default configuration will write both logs. However, you can change based on your requirement. The log is controlled in `SecDefaultAction`​ directive. Let’s look at default logging configuration in setup.conf

```apacheconf
SecDefaultAction “phase:1,deny,log”
```

Copy

To log Debug, Audit log – use “log” To log only audit log – use “nolog,auditlog” To log only debug log – use “log,noauditlog” You can specify the Audit Log location to be stored which is controlled by SecAuditLog directive.

Let’s write audit log into /opt/apache/logs/modsec_audit.log by adding as shown below.

* Add SecAuditLog directive in setup.conf and restart Apache Web Server

```apacheconf
SecAuditLog /opt/apache/logs/modsec_audit.log
```

Copy

* After the restart, you should see modsec_audit.log getting generated

### Enable Rule Engine

By default Engine Rule is Off that means if you don’t enable Rule Engine you are not utilizing all the advantages of Mod Security.

Rule Engine enabling or disabling is controlled by `SecRuleEngine`​ directive.

* Add SecRuleEngine directive in setup.conf and restart Apache Web Server

```apacheconf
SecRuleEngine On
```

Copy

There are three values for SecRuleEngine:

* On – to enable Rule Engine
* Off – to disable Rule Engine
* DetectionOnly – enable Rule Engine but never executes any actions like block, deny, drop, allow, proxy or redirect

Once Rule Engine is on – Mod Security is ready to protect with some of the common attack types.

### Common Attack Type Protection

Now web server is ready to protect with common attack types like XSS, SQL Injection, Protocol Violation, etc. as we have installed Core Rule and turned on Rule Engine. Let’s test a few of them.

**XSS Attack**

* Open Firefox and access your application and put <script> tag at the end or URL
* Monitor the modsec_audit.log in apache/logs folder

You will notice Mod Security blocks request as it contains <script> tag which is the root of XSS attack.

Directory Traversal Attack:- Directory traversal attacks can create a lot of damage by taking advantage of this vulnerabilities and access system related file. Ex – /etc/passwd, .htaccess, etc.

* Open Firefox and access your application with directory traversal
* Monitor the modsec_audit.log in apache/logs folder

```apacheconf
http://localhost/?../.../boot
```

Copy

* You will notice Mod Security blocks request as it contains directory traversal.

### Change Server Banner

Earlier in this guide, you learned how to remove Apache and OS type, version help of ServerTokens directive.

Let’s go one step ahead, how about keeping server name whatever you wish? It’s possible with `SecServerSignature`​ directive in Mod Security. You see it’s interesting.

Note: to use Mod Security to manipulate Server Banner from a header, you must set ServerTokesn to Full in httpd.conf of Apache web server.

* Add SecServerSignature directive with your desired server name in setup.conf and restart Apache Web Server

```apacheconf
SecServerSignature YourServerName
```

Copy

Ex:

```apacheconf
[/opt/apache/conf/crs] #grep SecServer modsecurity_crs_10_setup.conf
SecServerSignature geekflare.com
[/opt/apache/conf/crs] #
```

Copy

## General Configuration

Let’s check out some of the general configurations as best practice.

### Configure Listen

When you have multiple interfaces and IP’s on a single server, it’s recommended to have Listen directive configured with absolute IP and Port number.

When you leave apache configuration to Listen on all IP’s with some port number, it may create the problem in forwarding HTTP request to some other web server. This is quite common in the shared environment.

* Configure Listen directive in httpd.conf with absolute IP and port as a shown example below

```apacheconf
Listen 10.10.10.1:80
```

Copy

### Access Logging

It’s essential to configure access log properly in your web server. Some of the important parameter to capture in the log would be the time taken to serve the request, SESSION ID.

By default, Apache is not configured to capture these data. You got to configure them manually as follows.

* To capture time taken to serve the request and SESSION ID in an access log
* Add %T & %sessionID in httpd.conf under LogFormat directive

```apacheconf
LogFormat "%h %l %u %t "%{sessionID}C" "%r" %>s %b %T" common
```

Copy

You can refer [http://httpd.apache.org/docs/2.2/mod/mod_log_config.html](http://httpd.apache.org/docs/2.2/mod/mod_log_config.html) for a complete list of parameter supported in LogFormat directive in Apache Web Server.

### Disable Loading unwanted modules

If you have compiled and installed with all modules, then there are high chances you will have many modules loaded in Apache, which may not be required.

Best practice is to configure Apache with required modules in your web applications. Following modules have security concerns, and you might be interested in disabling in httpd.conf of Apache Web Server.

WebDAV (Web-based Distributed Authoring and Versioning) This module allows remote clients to manipulate files on the server and subject to various denial-of-service attacks. To disable comment following in httpd.conf

```apacheconf
#LoadModule dav_module modules/mod_dav.so
#LoadModule dav_fs_module modules/mod_dav_fs.so
#Include conf/extra/httpd-dav.conf
```

Copy

Info Module The mod_info module can leak sensitive information using .htaccess once this module is loaded. To disable comment following in httpd.conf

```apacheconf
#LoadModule info_module modules/mod_info.so
```

### apache 2.4.41, intermediate config, OpenSSL 1.1.1k

```
# generated 2024-03-18, Mozilla Guideline v5.7, Apache 2.4.41, OpenSSL 1.1.1k, intermediate configuration
# https://ssl-config.mozilla.org/#server=apache&version=2.4.41&config=intermediate&openssl=1.1.1k&guideline=5.7

# this configuration requires mod_ssl, mod_socache_shmcb, mod_rewrite, and mod_headers
<VirtualHost *:80>
    RewriteEngine On
    RewriteCond %{REQUEST_URI} !^/\.well\-known/acme\-challenge/
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>

<VirtualHost *:443>
    SSLEngine on

    # curl https://ssl-config.mozilla.org/ffdhe2048.txt >> /path/to/signed_cert_and_intermediate_certs_and_dhparams
    SSLCertificateFile      /path/to/signed_cert_and_intermediate_certs_and_dhparams
    SSLCertificateKeyFile   /path/to/private_key

    # enable HTTP/2, if available
    Protocols h2 http/1.1

    # HTTP Strict Transport Security (mod_headers is required) (63072000 seconds)
    Header always set Strict-Transport-Security "max-age=63072000"
</VirtualHost>

# intermediate configuration
SSLProtocol             all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite          ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305
SSLHonorCipherOrder     off
SSLSessionTickets       off

SSLUseStapling On
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"

```
