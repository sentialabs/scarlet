## Using TLS/SSL in your development environment

This project is a working hands-on example of how you can set up your local machine with a fake TLS/SSL certificate and access your locally running instance of the Bloomreach web application via HTTPS instead of HTTP. You can find an article describing how to create your own fake certificates here: [FIXME]

The project itself is a very minimalist Bloomreach project that has virtually no features other than configuration for making HTTPS work (and only for the URLs that I mention here). Its only function is to serve the default homepage and supply the bare minimum CMS, thus demonstrating that content and CMS functions are working correctly over HTTPS.

You can build and run the project in the usual way:
```
mvn clean package
mvn -P cargo.run
```
The project uses version 14.1 of the Bloomreach libraries and works both with JDK 8 and JDK 11. 

When you run the application with ```cargo.run``` you can point your browser as usual to ```http://localhost:8080/cms``` and ```site``` in order to access the application.

---

### Setting up HTTPS
In order to start accessing the same web application over HTTPS you'll need to set up some stuff outside of the project:
* Install and configure Apache2
* Edit your ```hosts``` file
* Import the fake Root CA certificate in your browser
* Point your browser to an URL different from 'localhost'

---

__Instaling Apache__ is a bit outside of the scope of this instruction. For Ubuntu it is easy:
```
apt install apache2
```
and then make sure it runs as a daemon on startup.

For Windows it is less simple. Here is what the Apache Foundation suggests: 

https://httpd.apache.org/docs/2.4/platform/windows.html

---

__Configuring Apache__
The Bloomreach documentation has a good step by step procedure description on what to do:

https://documentation.bloomreach.com/14/library/deployment/configuring/configure-apache-httpd-as-reverse-proxy-for-hippo.html

The title of that page is a bit misleading. Yes, technically you are using Apache as a reverse proxy but you are at the same time using it to forward requests from the browser to your web application.

After freshly installing Apache you'll have to enable several modules that by default are disabled:
```
a2enmod ssl proxy_http headers rewrite
```
Sorry, ```a2enmod``` and ```a2ensite``` only work on Ubuntu. For other distros and for Windows you will probably have to include the modules and the config files in the overall ```httpd.conf``` master configuration file.


Anyway, I have prepared two drop in configuration files for Apache that you can copy from this project to your Apache confguration directory and use as-is. You can find them here in the downloaded project source:
```
apache-and-dns/apache2/sites-available/
```
Instructions: copy all ```*.conf``` files to the same directory in your Apache server configuration and install them with
```
a2ensite acme-cms-ssl.conf acme-site-ssl.conf 
```
The file ```logging.conf``` is my own personal replacement for Apache's default configuration, use it at your own convenience.

---

You will also need the server certificate ```wildcard.acme.eu.crt``` and its private key ```wildcard.acme.eu.key``` which you can find at
```
apache-and-dns/ssl/acme.eu/
```
Copy these two files (the root certificate ```Rokus_CA.crt```is not needed by Apache) to the location where Apache normally keeps server certificates and private keys. Again for Ubuntu this location is already preconfigured in the drop-in config files I provided. For other distros and for Windows, you're a bit on your own. However __do make sure to update__  the location of these two files in the provided drop-in config files while copying things from the project to the destination.

Next, we will __set up DNS__ to point the URL's for ```cms-local.acme.eu``` and ```local-scarlet.acme.eu``` to our own development machine. This sounds more ominous that it is. All that is needed is to add some lines to a file named ```hosts``` . You can find this file here:
```
/etc/hosts          # for all linux distros

or

C:\Windows\System32\drivers\etc\hosts 
```
Note that for Windows you need to have admin privileges to edit this file. (for linux you need to be root but this goes without saying for most of the commands in this explanation).

You need to add these two lines:
```
127.0.0.1       cms-local.acme.eu
127.0.0.1       local-scarlet.acme.eu
```
This bit of configuration overrides any and all information that other sources for DNS (such as your regular network DNS server) may have regarding these URLs.

---

__Importing the fake Root CA certificate__. 
The fake server certificate for the ```*.acme.eu``` domain has been signed by a fake Certificate Auhtority that I created myself. Your standard browser will not recognize my authority out of the box but we can tell it to shut up and stop whining. We do this by importing the fake Root CA into the browsers _trust store_ . 

The root certificate lives in this file:
```
apache-and-dns/ssl/acme.eu/Rokus_CA.crt
```

Where to find the trust store and how to import the Root CA into it varies by operating system and browser. 
Here are some sources:

Windows: https://thomas-leister.de/en/how-to-import-ca-root-certificate/#windows)

Firefox: https://portal.threatpulse.com/docs/am/Solutions/ManagePolicy/SSL/ssl_firefox_cert_ta.htm

Chrome: https://portal.threatpulse.com/docs/sol/Solutions/ManagePolicy/SSL/ssl_chrome_cert_ta.htm


Also, if you have __other web applications__ that need to access your locally running webserver and they happen to run on Java, you must also import the _server certificate_ (not the Root CA) into the trust store (keystore) used by the JVM that runs those webapplications. Here is a source detailing how to do this:

https://stackoverflow.com/questions/4325263/how-to-import-a-cer-certificate-into-a-java-keystore

---

Finally, when all of the above has been installed, configured and set, it is time to play with our new toy. Point your browser to (note: the urls should all end in a slash)

https://cms-local.acme.eu/

https://local-scarlet.acme.eu/

https://cms-local.acme.eu/console/

and behold the fruits of all your toil! 
