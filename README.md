# **LEMP STACK IMPLEMENTATION**
* In order to complete this project you will need an AWS account and a Vitual Server with Ubuntu Server OS.
* To learn how to create an AWS free tier account and an EC2 Instance of t2.micro family with Ubuntu Server 20.04 LTS(HVM), follow this link: [Create an EC2 Instance in AWS](https://github.com/Shadrach247/LAMP-STACK-IMPLEMENTATION-IN-AWS)
* This time, instead of connecting via PuTTY, I will connect to my EC2 Instance with an already downloaded and installed Git Bash Command Line Interface (CLI).
* To connect, Launch Git Bash and run the following command:

![SSH Client](SSHclient.png)
```Bash
ssh -i shadrach-ec2.pem ubuntu@ec2-35-178-12-124.eu-west-2.compute.amazonaws.com
```
* _It will look like this:_
![GIT BASH LOGIN](GITBASH-A.png)

# **STEP 1 - Installing the Nginx Web Server**
* In order to display web pages to our site visitors we are going to employ Nginx, a high performance Web Server. We'll use the **apt** Package Manager to install this package.
* Since this is our first time using **apt** for this session, start of by updating your server's package index. Following that, you can use **apt** install to get Nginx installed:
```Bash
$ sudo apt update

$ sudo apt install nginx
```
* When prompted, enter **Y** to confirm that you want to install Nginx. Once the installation is finished, the Nginx Web Server will be active and running on your Ubuntu 20.04 Server.
* To verify that nginx was successfully installed and running as a service in Ubuntu, run:
```Bash
$ sudo systemctl status nginx
```
* If it is green and running, then you did everything correctly - you have just launch your first web Server in the Clouds!

* We need to configure our firewall settings to allow HTTP traffic. UFW has different application profiles that we can leverage for accomplishing that.
* To list all currently available UFW application profile, **run :**
```Bash
sudo ufw app list
```
![App List](AppList.png)

_Available applications :_

* Nginx Full : This profile opens both port 80(normal, unencrypted web traffic) and port 443(TLS/SSL encrypted web traffic)
* Nginx HTTP : This profile opens only port 80(normal, unencrypted web traffic)
* Nginx HTTPS : This profile opens only port 443(TLS/SSL encrypted traffic)

It is recommended that you enable the most restrictive profile that will still allow the traffic you've configured. Since we haven't configured SSL for our Server yet, we will ony need to allow traffic on port 80.
* You can enable this by typing:
```Bash
$ sudo ufw allow 80

$ sudo ufw allow ssh

$ sudo ufw enable
```
_You may get a prompt like this:_
> command may disrupt existing ssh connection. proceed with operation (y/n)?

_Click on **y** on your keyboard and ENTER to proceed._

* You can verify the change by typing:
```Bash
$ sudo ufw status
```
_You should see HTTP traffic allowed in the displayed output :_

![UFW STATUS](UfwStatus.png)

First, lets try to access it locally in our Ubuntu shell, **run:**

```Bash
$ curl http://localhost:80
or
$ curl http://127.0.0.1:80
```
![UBUNTU SHELL](Curl.png)
As an output, you can see some strangely formatted test, do not worry, we just made sure that our Nginx web service responds to _'curl'_ command with some payload.

Now it is time for to test how our Nginx Server can respond to requests from the internet.
* Another way to retrieve your Public IP address, other than to check it in AWS Web Console, is to use the following command.
```Bash
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

![PUBLIC IP](PublicIP.png)

* Open a web browser of your choice and try to access the following URL:
```Bash
http://<Public-IP-Address>:80
```
* See the Public IP I used:
```Bash
35.178.12.124:80
```
To make it work and accessible from the internet, I had to do a manual adjustment of the Security Group settings to allow inbound request from TCP/80/AnyWhere.

![INBOUND RULES](Inbound.png)

* Voila! Connection was successful after saving my new Inbound rules and refreshing the page. 
 * _see my output :_
![Welcome Nginx](WelcomeNginx.png)

Infact, it is the same content that we previously got by _'curl'_ command, but represented in a nice HTML formatting by the web browser.

# **STEP 2 - Installing MySQL**
Now that you have a Web Server up and running you need to install a Database Management System(DBMS) to be able to store and manage data for your site in a relational database. MySQL is a popular relational database management system used within PHP environments, so we will use it in our project.
Again use **_apt_** to acquire and install this software :
```Bash
$ sudo apt install mysql-server
```
When prompted, confirm installation by typing **Y**, and then ENTER.

When the installation is finished, it's recommended that you run a security script that comes pre-installed with MySQL. This Script will remove some insecure default settings and lock down access to your database system.
Start the interactive script by running:
```Bash
$ sudo mysql_secure_installation
```
This will ask if you want to configure the VALIDATE PASSWORD PLUGIN. Answer **Y** for "yes" or anything else to continue without enabling.
If you answer "yes", you'll be asked to select a level of password validation. Keep in mind that if you enter **2** for the strongest level you will receive errors when attempting to set any password which does not contain numbers, upper and lowercase letters, and special charachers, or which is based on common dictionary words.

Regardless of whether you chose to set up the VALIDATE PASSWORD PLUGIN, your Server will next ask you to select and confirm a password for MySQL **root** user. This is not to be confused with the **system root**. The **database root** user is an administrative user with full privileges over the database system. Even though the default authentication method for the MySQL root user dispenses the use of a password, **even when one is set**, you should define a strong password here as an additional safety measure.

If you enabled password validation, you'll be shown the password strength for the root password you just entered and your Server will ask if you want to continue with that password. If you are happy with your current password, enter **Y** for "yes" at the prompt.

For the rest of the questions, press **Y** and hit ENTER key at the prompt. This will remove some anonymous users and the test database, disable remote root logins, and load these new rules so that MySQL immediately respect the changes you have made.

When you are finished, test if you're able to log into the MySQL console by typing:

```Bash
$ sudo mysql
```
This will connect to the MySQL Server as the administrative database user **root**, which is infered by the use of **_sudo_** when running this command.

You should see an output like this :

To exit the MySQL console, type :

Notice that you didn't need to provide a password to connect as the **root** user, even though you have defined one when running the mysql_secure_installation script. That is because the default authentication method for the administrative MySQL user is **unix_socket** instead of **password**. Even though this might look like a security concern at first, it makes the database server more secure because the only users allowed to log is as the root MySQL user are the system users with _**sudo**_ privileges connecting from the console or through an application running with the same privileges. In practical terms, that means you won't be able to use the administrative database root user to connect from your PHP application. Setting a password for the root MySQL accounts works as a safeguard, in case the default authentication method is changed from **unix_socket** to **password**.

For increased security, it's best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server

Your MySQL Server is now installed and secured. Next, we will install PHP, the final component in the **LEMP Stack**

# **STEP 3 - Installing PHP**
You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the Web Server.

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the Web Server. This allows for a better performance in most PHP-based websites, but it requires additional configuration. You'll need to install **php-fpm**, which stands for "PHP fastCGI Process Manager", and tell Nginx to pass PHP requests to this software for processing. Additionally you'll need **php-mysql**, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependenes.

To install these 2 pacjages at once, **run :**
```Bash
$ sudo apt install php-fpm php-mysql
```
When prompted, type **Y** and press ENTER to confirm installation. You now have your PHP components installed. Next, you will configure Nginx to use them.