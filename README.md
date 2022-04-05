# __The basics of Linux system administration__

## __Summary__

The purpose of this README is to simply administer a Linux system. <br> In this tutorial we will create users and add them to two new groups.<br> Then we will give administrator rights to a group. <br>Then we will see how to generate an RSA key to connect to a machine without a password.<br> Finally we will finish with the installation of WordPress 

<br>

- - - -

## __Initialisation__

One of the first commands to run on a new Linux system is this one:

```
sudo apt update -y && sudo apt upgrade -y
```
It allows to update the system package 

<br>

- - - -

## __Step 1__


This command allows you to create a user with his home folder (his user folder). Once executed you will have to enter the user's password and confirm it. Then you can enter the user's information, otherwise you can prove that you have not entered anything by pressing the enter key. You will only have to confirm with the y key. If any of the information entered is wrong press n and enter the information.

_Command:_
```
sudo adduser bob --home /home/bob/
```

_Output:_

> Adding user '`bob`' ... <br>
    Adding new group '`bob`' (1010) ... <br>
    Adding new user '`bob`' (1010) with group '`bob`' ... <br>
    Creating home directory '`/home/bob/`' ... <br>
    Copying files from '`/etc/skel`' ...  <br>
    Enter new UNIX password: <br>
    Retype new UNIX password: <br>
    passwd: password updated successfully <br>
    Changing the user information for bob <br>
    Enter the new value, or press ENTER for the default <br>
            Full Name []: <br>
            Room Number []:  <br>
            Work Phone []: <br>
            Home Phone []: <br>
            Other []: <br>
    Is the information correct? [Y/n] y <br>

<br>

You will just have to modify __`bob`__ by the name of the other users you have to created 
<br>
To create a group follow the command below, you can modify __`groupname`__ by the name of the group you want and __`12345`__ by the desired gid
```
sudo addgroup groupname --gid 12345
```
Now to add a user to a group you have to use the command below in which you modify __`examplegroup`__ by the name of the group in which you want to add your user and __`exampleusername`__ by the user in question 

```
sudo usermod -a -G examplegroup exampleusername
```
<br>

- - - -

## __Step 2__

The purpose of this step is to assign administrator rights to the group you created earlier. For that we will modify the sudoers file with the command below. 

```
sudo visudo
```
Then you will add this line to the file in the file. You can replace __`examplegroup`__ with the group that will inherit the administrator rights.
```
%examplegroup ALL=(ALL) NOPASSWD:ALL
```
To leave the Document press the keys: ⌃X then ↩ <br>
If the list of keys to use is not displayed at the bottom of your screen you can try using this keystroke sequence: __:q__

if you want to test if the changes have taken place, you can use the following command by replacing __`username`__ with the name of the account on which you want to change and test with the second command

```
su username
sudo apt-get install vim
```

<br>

- - - -
## __Step 3__
In this step we will set up an RSA key. This key will allow us to connect to the machine without using a password.  

Now the future steps will have to be executed on the machine that will want to connect to the server. First of all, we will have to generate a pair of keys, one private and one public. For this we need to execute the following command:
```
ssh-keygen
```
_Output:_

>   Generating public/private rsa key pair. <br>
    Enter file in which to save the key (/Users/charlescoste/.ssh/id_rsa): <br>
    /Users/charlescoste/.ssh/id_rsa already exists. <br>
    Overwrite (y/n)? y <br>
    Enter passphrase (empty for no passphrase):  <br>
    Enter same passphrase again: <br>
    Your identification has been saved in /Users/charlescoste/.ssh/id_rsa <br>
    Your public key has been saved in /Users/charlescoste/.ssh/id_rsa.pub <br>
    The key fingerprint is: <br>
    SHA256: _Something_ local <br>
    The key's randomart image is: <br>
    _Something_

Then to initialize the connection you can write the following command replacing __`bob`__ by the user on which the person must connect and __`172.16.235.12`__ by the server ip
```
cat ~/.ssh/id_rsa.pub | ssh bob@172.16.235.12 "mkdir ~/.ssh -p && cat - >> ~/.ssh/authorized_keys"
```
Then to connect you should use this command replacing __`172.16.235.12`__ by the server ip and __`bob`__ by the server user
```
ssh bob@172.16.235.12
```
Now you are connected with ssh without password to the server
- - - -
## __Step 4__
In this step we will install WordPress. First of all we have to install all the dependencies related to WordPress 

```
sudo apt install apache2 \
                 ghostscript \
                 libapache2-mod-php \
                 mysql-server \
                 php \
                 php-bcmath \
                 php-curl \
                 php-imagick \
                 php-intl \
                 php-json \
                 php-mbstring \
                 php-mysql \
                 php-xml \
                 php-zip
```
Then we will create a folder for the server
```
sudo mkdir -p /srv/www
```
Then we will assign rights to the folder in question
```
sudo chown www-data: /srv/www
```
The following command will download the WordPress installation file and unzip it
```
curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
```
Now the problem is that if we access the website we will see the default page of Apache (a web server creation tool). To display the WordPress configuration page we will have to initialize it. To do this we will open the web page file with the following command 
```
nano /etc/apache2/sites-available/wordpress.conf
```
You will have to copy and paste the following code inside 
```
<VirtualHost *:80>
    DocumentRoot /srv/www/wordpress
    <Directory /srv/www/wordpress>
        Options FollowSymLinks
        AllowOverride Limit Options FileInfo
        DirectoryIndex index.php
        Require all granted
    </Directory>
    <Directory /srv/www/wordpress/wp-content>
        Options FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>
```
Then we will start the server with this command
```
sudo a2ensite wordpress
```
Now we need to configure the WordPress related database. With this command we can access to MySQL
```
sudo mysql -u root
```
Here we create the database
```
CREATE DATABASE wordpress;
```
With this command we are going to create a MySQL user, you can replace __`your-password`__ by a password
```
CREATE USER wordpress@localhost IDENTIFIED BY 'your-password';
```
After creating the user we will assign him the rights necessary for the operation of WordPress 
```
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
    -> ON wordpress.*
    -> TO wordpress@localhost;
```
The following command will reload the grant tables
```
FLUSH PRIVILEGES;
```
Finaly we quit the MySQL terminal  
```
quit
```
and start MySQL
```
sudo service mysql start
```
And then all you have to do is access your server page using your browser. It will be necessary to configure your account then fill in the information of the database created earlier. And congratulations you have administered a linux system and installed WordPress. You can now create the site of your dreams. 
<br><br>

Thank you for following this tutorial
- - - -
