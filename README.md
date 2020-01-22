# Linux Server Configuration Project

These are the steps taken to deploy the Music Inventory app which was done as part of the Udacity Full-Stack NanoDegree program.

The website is running on a [Apache2](https://httpd.apache.org/) server with `mod_wsgi`.

The app can be found at www.danisantoscode.com

To add instruments to the database, please login to Google.

The steps taken in order to achieve so were as follows:

## AWS Lightsail Instance

Created a Ubuntu 18.04 LTS instance with the following Public IP:  `35.183.93.70`

Made the IP static to set up the domain name later ahead. (`15.222.142.52`)

Under the Networking panel, added rules to the firewall:

```
Application Protocol Port range
SSH          TCP      22
HTTP         TCP      80
Custom       TCP      123
Custom       TCP      2200
```

## Ubuntu setup

Run the following, to make sure packages, especially security ones, are updated:

- `sudo apt-get update`

- `sudo apt-get upgrade`


## Set the hostname on the host file

- `sudo nano /etc/hosts`

Add the following lines to the file:
```
127.0.0.1 localhost
35.156.207.226 www.danisantoscode.com
```
(I created a domain name on Google. More info below)

## Disable ssh with root

By default, Lighsail won't allow remote access via root

## Create user grader

`sudo adduser grader`

## Add Grader to sudo group

`sudo adduser grader sudo`

## Generate SSH keys for Grader and add to server

- This prevents brute force attacks

- Generate ssh-keys for grader with passphrase
`ssh-keygen`

- Generated two keys (one public, one private)

- Add grader public key to server:
`cd /home/grader`
`sudo mkdir .ssh`
`sudo touch .ssh/authorized_keys`
`sudo nano .ssh/authorized_keys`

- Added public key (`graderAccess.pub`) to authorized keys file on the Server

- You can now log in as `grader` by running:
` ssh grader@35.156.207.226 -p 2200 -i ~/.ssh/graderAccess`
(The key and the passphrase are in the provided "Notes to Reviewer")

Notice: - You might have to run `chmod 400 graderAccess` in order to make it secure, since Amazon Lighsail might raise the following error message:  WARNING: UNPROTECTED PRIVATE KEY FILE!

## Disallow Remote Access via root

Change SSHD Config file to disable remote access with root and enable port 2200:

- `sudo nano /etc/ssh/sshd_config`
- `PermitRootLogin no`

- `PasswordAuthentication no`

- `#Run SSH on a non standard port
Port 2200
`
-  `sudo service sshd restart`

## Ubuntu Firewall

- Make sure UFW is installed. (It is isntalled by default on Lightsail)

- `sudo ufw reset`

- `sudo ufw disable`

- `sudo ufw default deny incoming`

- `sudo ufw default allow outgoing`

- `sudo ufw allow 2200/tcp`

- `sudo ufw allow 80/tcp`

- `sudo ufw allow 123/tcp`


To enable the firewall:

`sudo ufw enable`

`sudo ufw status` will return:

```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/tcp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/tcp (v6)               ALLOW       Anywhere (v6)
```

## WSGI and Apache setup

- Install apache:
`sudo apt-get install apache2`

- Install mod_wsgi:
`sudo apt-get install libapache2-mod-wsgi`

- To enable mod_wsgi:
`sudo a2enmod wsgi`

- Create a `music_inventory.conf` file to Apache by running:
`sudo nano /etc/apache2/sites-available/music_inventory.conf`
(See the conf file on this repo (`music_inventory.conf`)

- Then activate this conf file by running:
`sudo a2ensite music_inventory.conf`

- Make sure the other conf files are disabled by running `sudo a2dissite <NAME>`

- After making those changes, run `sudo service apache2 restart`

## Cloning the webapp project repo on Git

Install git:

- `sudo apt-get install git`

Cd into the following path:

`sudo /var/www`

- Clone the repo `music_inventory` repo under the `wwww` directory

- Create a wsgi file in that repo with app to be run on Mod WSGI's Virtual Host:

`sudo touch music_inventory.wsgi`

*Note*: You can see this file in the current repo!

## Setting up the Environment and installing dependencies for the project to run

1) Install pip:
 `sudo apt-get install python3-pip`

2) `sudo pip3 install virtualenv`

3) Create env `sudo virtualenv venv`

4) Activate the env by running `sudo source venv/bin/activate`

5) To install dependencies, run `pip3 install -r requirements.txt`


## PostgreSQL setup

- `sudo apt-get install postgresql postgresql-contrib`

Upon installation, Postgres creates a Linux user called "postgres" which can be used to access the system. We can change to this user by typing:

`sudo su - postgres`

From here, we can connect to the system by typing:

`psql`

Create database `catalog`:
`CREATE DATABASE catalog;`

The current default when installing PostgreSQL from the Ubuntu repositories is to not allow remote connections to the database, removing a potential attack vector.

(More on the topics can be found at https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

## Creating Roles and Granting Permissions on the Database

Create role `catalog`in database:

- `psql` followed by `\password catalog`

- user catalog has limited permissions

- Allow user to login:

`ALTER ROLE "inventory_user" WITH LOGIN;`

- Connect to the catalog database:
`\c catalog`
- Grant limited powers to the user:
```
GRANT CONNECT ON DATABASE catalog TO catalog;
GRANT SELECT ON instrument TO catalog;
GRANT INSERT ON instrument TO catalog;
GRANT UPDATE ON instrument TO catalog;
GRANT DELETE ON instrument TO catalog;
```

(https://www.postgresql.org/docs/9.1/sql-grant.html)
(https://tableplus.io/blog/2018/04/postgresql-how-to-create-read-only-user.html)

## CHOWN and CHMOD

It's important to give file permissions to write, read and execute the following files:

Go to the /var/www directory and run
`sudo chown :www-data music_inventory`
`sudo chmod 775 music_inventory`

Then, go to /var/www/music_inventory and run:
`sudo chown :www-data catalog`
`sudo chmod 775 catalog`

Give permissions to the database file at `/etc/postgresql/10`:
`sudo chown :www-data main`
`sudo chmod 775 main`


## Setting up the schema

Finally, to set up the schema and prepopulate the database, run the following:

`python3 /var/www/music_inventory/catalog/database_setup.py`

`python3 /var/www/music_inventory/catalog/loadinstruments.py`


## DNS (Google Domains)

This DNS was created for another purpose, but I'm using it in this project. It's a Google domain.

I added the public IP of this AWS instance to the list of resource records.


## OAuth Consent

I added my domain to the list of  Google's OAuth authorized domains.
