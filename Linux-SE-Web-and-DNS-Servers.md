# Using DNS Clients 
#Configure resolv.conf
There are two client-side name resolution files of note in CentOS7

	1. /etc/resolv.conf - which contains a listing of global DNS services for the client to query when resolving DNS names. Since this file is read in order, we can establish the order in which we want the system to attempt to resolve the hosts. It can be helpful to put local DNS specifications first if you want the system to check for local sites first.

	2. /etc/nsswitch.conf - which will tell applications which sources to get name service information from. Some of the examples include aliases for mail aliases or hosts for host resolution. For example it is common to add the line “hosts: files dns” to specify that hosts should be resolved through the /etc/hosts file first, then try the DNS options.  

Install DNS Clients

$ sudo yum install -y bind-utils 
Usually installed already.


# Configure a basic Apache web server

The first thing we will want to do is identify our DocumentRoot and DirectoryIndex.
	$ grep ^DocumentRoot /etc/httpd/conf/httpd.conf
	This will likely return “DocumentRoot “/var/www/html”
	$ grep DirectoryIndex /etc/httpd/conf/httpd.conf
	This should return “		DirectoryIndex index.html”
Having identified these two pieces of information, we can make the index.html page in the “DocumentRoot”
	$ cd /var/www/html/
	$ sudo vim index.html
	Add some text, save, and then exit.
Now you should be able to navigate your browser to your Apache server name and see the new index.html page.

# Configure private access using Basic Auth

To help troubleshoot Apache servers, take a look at the httpd error_log
	$ sudo less /var/log/httpd/error_log
Now to configure security so non-root privileged admins can do their job without root access, make a directory
	$ cd /var/www/html
	$ sudo mkdir private
We need to now tell Apache that this directory will house security settings
	$ sudo vim /etc/httpd/conf/httpd.conf
	Add these lines at the bottom of the file
	<Directory “/var/www/html/private”>
		AllowOverride AuthConfig
	</Directory>
Test the Apache config
	$ sudo apachectl configtest
Now make an htaccess file
	$ sudo vim private/.htaccess
Add the lines
	AuthType Basic
	AuthName “Enter your credentials”
	AuthUserFile “/etc/httpd/conf/.userdb”
	Require user user1
	This sets the auth type to basic auth, prompts us with a nice message, specifies the AuthUserFile as .userdb, and requires the user to be user1

We will also need to create an index.html file for /private, add some text to it too.
	$ sudo vim index.html
Next we will change the ownership of the directory /private and its contents.
	$ sudo chown -R apache:apache private
	$ sudo chmod -R 770 private
Now we need to create the user password database file .userdb
	$ cd /etc/httpd/conf
	$ sudo htpasswd -c .userdb user1
	Now enter a password for user1
Change the ownership and permissions for .userdb
	$ sudo chown apache:apache .userdb
	$ sudo chmod 660 .userdb
Restart Apache to put the changes in place
	$ sudo systemctl reload httpd
Make sure to test this in a private tab as it will not remember basic auth info, which will make troubleshooting easier.

# Configure access to group modified content

Move to the Document Root location and create a directory for your group
	$ cd /var/www/html/
	$ sudo mkdir privategroup
Now edit the httpd.conf file
	$ sudo vim /etc/httpd/conf/httpd.conf
Add these lines to the end of the file.
	<Directory “/var/www/html/privategroup”>
		AllowOverride AuthConfig
	</Directory>
Test the configuration
	$ sudo apachectl configtest
Make an htaccess file
	$ sudo vim privategroup/.htaccess
	Add these lines
	AuthType Basic
	AuthName “Enter your group credentials”
	AuthGroupFile “/etc/httpd/conf/.groupdb”
	AuthUserFile “/etc/httpd/conf/.grouppassdb”
	Require group webgroup 
Add an index.html file for the privategroup directory and add some text.
	$ sudo vim privategroup/index.html
Next we will change the ownership of the directory /private and its contents.
	$ sudo chown -R apache:apache privategroup
	$ sudo chmod -R 770 privategroup
Now we need to create the auth group file .groupdb
	$ cd /etc/httpd/conf
	$ sudo vim .groupdb
	Now enter the group users
	webgroup: user1 user2
Next we need to create the password file specified in the httpd.conf
	$ sudo htpasswd -c .grouppassdb user1
	Give user1 a password when prompted 
	Repeat for user2 but drop the -c
	$ sudo htpasswd .grouppassdb user2
Change the ownership and permissions for .groupdb
	$ sudo chown apache:apache .group*
	$ sudo chmod 660 .group*
Restart Apache to put the changes in place
	$ sudo systemctl reload httpd

# Configure basic virtual host

Virtual hosts let us have multiple websites on a single physical machine with separate logs, site name, and access controls.

To create virtual host add a conf file for the host into /etc/httpd/conf.d
	$ sudo vim /etc/httpd/conf.d/vhost1.conf
This is how a basic virtual host conf file will look
	<VirtualHost *:80>
		ServerAdmin webmaster@vhost1.localnet.com
		DocumentRoot “/var/www/html/vhost1.com/“
		ServerName vhost1.localnet.com
		ServerAlias www.vhost1.localnet.com
		ErrorLog “/var/log/httpd/vhost1.com-error-log”
		CustomLog “/var/log/httpd/vhost1.com-access_log” common
	</VirtualHost>
Now we need to create the files we specified for our new virtual host
	$ sudo mkdir /var/www/html/vhost1.com
	$ sudo vim /var/www/html/vhost1.com/index.html
	Add some text for vhost1’s landing page.
Test the configuration
	$ sudo apachectl configtest
We can also “DUMP” our virtual hosts to see their info
	$ sudo httpd -D DUMP_VHOSTS
A simple way to fix the DNS settings to recognize this virtual host, we will edit /etc/hosts
	$ sudo vim /etc/hosts
	Add the following lines
	i.p.addr vhost1.localnet.com vhost1
	i.p.addr www.vhost1.localnet.com
Test name resolution by pinging vhost1, vhost1.localnet.com, and www.vhost1.localnet.com
Reload the apache server
	$ sudo systemctl reload httpd

# Configure virtual host on non-standard port

To create virtual host add a conf file for the host into /etc/httpd/conf.d
	$ sudo vim /etc/httpd/conf.d/vhost2.conf
This is how a basic virtual host conf file will look
	<VirtualHost *:8085>
		ServerAdmin webmaster@vhost2.localnet.com
		DocumentRoot “/var/www/html/vhost2.com/“
		ServerName vhost2.localnet.com
		ServerAlias www.vhost2.localnet.com
		ErrorLog “/var/log/httpd/vhost2.com-error-log”
		CustomLog “/var/log/httpd/vhost2.com-access_log” common
	</VirtualHost>
In this case, we will need to tell Apache to listen to port 8085 as we specified for vhost2.
	$ sudo vim /etc/httpd/conf/httpd.conf
	Add the following line in the Listen area
	Listen 8085
Now we need to create the files we specified for our new virtual host
	$ sudo mkdir /var/www/html/vhost2.com
	$ sudo vim /var/www/html/vhost2.com/index.html
	Add some text for vhost2’s landing page.
Additionally, we need to let SELinux know that we are okay letting in traffic on 8085
	$ sudo semanage port -at http_port_t -p tcp 8085
Check to see if that command worked with
	$ sudo semanage port --list | grep ^http_port_t
After updating SELinux, we need to let traffic for port 8085 through the firewall
	$ sudo firewall-cmd --permanent --add-port 8085/tcp
	$ sudo firewall-cmd --reload
Test the configuration
	$ sudo apachectl configtest
We can also “DUMP” our virtual hosts to see their info
	$ sudo httpd -D DUMP_VHOSTS
A simple way to fix the DNS settings to recognize this virtual host, we will edit /etc/hosts
	$ sudo vim /etc/hosts
	Add the following lines
	i.p.addr vhost2.localnet.com vhost2
	i.p.addr www.vhost2.localnet.com
Test name resolution by pinging vhost1, vhost1.localnet.com, and www.vhost1.localnet.com
Reload the apache server
	$ sudo systemctl reload httpd


# Securing Apache with SSL/TLS

TLS is based on SSL 3.0 and has completely replaced it.

Advantages of HTTPS:
• Prevents intruders from tampering with data
• Prevents intruders from passively listening to data

Disadvantages of HTTPS:
• Longer latency
• CPU overhead of encryption
• More complex management environment

# Generating keypairs and self-signed certificates

Change directories
	$ cd /etc/pki/tls/certs
Generate the RSA private key
	$ sudo openssl genpkey -algorithm rsa -pkeyopt rsa_keygen_bits:2048 -out secure.localnet.com.key
Now that we have the private key, we need to generate the certificate request
	$ sudo openssl req -new -key secure.localnet.com.key -out secure.localnet.com.csr
	enter the requested information’
Next we need to sign the certificate request (this isn’t secure)
	$ sudo openssl x509 -req -days 120 -signkey secure.localnet.com.key -in secure.localnet.com.csr -out secure.localnet.com.crt
We should now make our private key more secure
	$ sudo chmod 600 secure.localnet.com.key
	$ sudo mv secure.localnet.com.key ../private
Restart Apache
	$ sudo systemctl restart httpd
Check to see if the openssl connection works
	$ sud

# Configure a secure virtual host

Start with a new document root
	$ sudo mkdir /var/www/html/secure
Now edit the ssl.conf file
	$ sudo vim /etc/httpd/conf.d/ssl.conf
	Search for VirtualHost, and change the DocumentRoot to our new secure directory
	DocumentRoot “/var/www/html/secure”
	Change the ServerName
	ServerName secure.localnet.com:443
	Next change the SSL cert and key file locations 
	SSLCertificateFile /etc/pki/tls/certs/secure.localnet.com.crt
	SSLCertificateKeyFile /etc/pki/tls/private/secure.localnet.com.key
Check to see if our changes worked
	$ sudo httpd -D DUMP_VHOSTS
Create the index.html file for the new host and add some text
	$ sudo vim /var/www/html/secure/index.html
Restore the correct SELinux security context
	$ sudo restorecon -Rv /var/www/html/secure
Make sure port 443 is open in the firewall and reload firewall
	$ sudo firewall-cod --permanent --add-service https
	$ sudo firewall-cmd --reload
Restart Apache
	$ sudo systemctl restart httpd
Add to the /etc/hosts file for DNS
	$ sudo vim /etc/hosts
	Add the following line
	i.p.addr secure.localnet.com


# Running a basic CGI script in Apache

Create a directory for the CGI scripts
	$ sudo mkdir /var/www/cgi-bin
Place or create your CGI scripts in this directory
	$ sudo vim /var/www/cgi-bin/loggedin.sh
	Here are the contents of this script:
	#!/bin/sh -f
	
	echo “Content-type: text/html”
	echo “”
	echo -n “<h1>Welcome to $(hostname)”
	echo -n “ It’s $(date)”
	
	echo “</h1>”
	
	echo “Here’s Who is Logged In Right Now:”
	echo “<BLOCKQUOTE><PRE>”
	who
	echo “</PRE></BLOCKQUOTE>”
	echo “</BODY></HTML>”
	
	exit 0

Make sure the script is executable
	$ sudo chmod +x /var/www/cgi-bin/loggedin.sh
By default, Apache servers do not allow the execution of CGI scripts because of SELinux. To fix this we need to enable an SELinux Boolean.
	$ sudo setsebool-P http_enable_cgi 1
This requires a restart of Apache
	$ sudo systemctl restart httpd
Test this with Firefox
	$ Firefox rhhost1.localnet.com/cgi-bin/loggedin.sh
The example CGI script will raise and SELinux alert, as it is preventing The who command from being executed.


