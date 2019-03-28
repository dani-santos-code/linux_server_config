# Linux Server Configuration Project

These are the steps taken to deploy the Music Inventory app which was done as part of the Udacity Full-Stack NanoDegree program.

The website is running on a [Apache2](https://httpd.apache.org/) server with `mod wsgi`.

The app can be found at www.danisantoscode.com

To add instruments to the database, please login to Google.

The steps taken in order to achieve so, were as follows:

## AWS Lightsail Instance

Created a Ubuntu 18.04 LTS instance with the following Public IP:  `35.156.207.226`

Made the IP static to set up the domain name later ahead.

Under the Networking panel, added rules to the firewall:

`Application	Protocol	Port range
SSH	          TCP	      22
HTTP	        TCP	       80
Custom	      TCP	     123
Custom	      TCP	     2200`


## Ubuntu setup

Run the following, to make sure packages, especially security ones, are updated:

- `sudo apt-get update`

- `sudo apt-get upgrade`


## Set the hostname on the host file

- `sudo nano /etc/hosts`

Add the following lines to the file:
`127.0.0.1 localhost
35.156.207.226 www.danisantoscode.com`

## Disable ssh with root
By default, Lighsail won't allow remote access via root

## Create user grader
`sudo adduser grader`

## Add Grader to sudo group

`sudo adduser grader sudo`

## Generate SSH keys for Grader and add to server
- This prevents brute force attacks

- Generate ssh-keys for grader, with passphrase of `grader1453`
`ssh-keygen`

- Generated two keys (one public, one private)

- Add grader public key to server:
`cd /home/grader`
`sudo mkdir .ssh`
`sudo touch .ssh/authorized_keys`
`sudo nano .ssh/authorized_keys`

- Added public key (`graderAccess.pub`) to authorized keys file on the Server
- You can now log in as `grader` by running:
` ssh grader@18.184.67.11 -p 2200 -i ~/.ssh/graderAccess`
And entering passphrase provided in the Notes: `grader1453`

Notice: - You might have to run `chmod 400 graderAccess` in order to make it secure, since Amazon Lighsail might raise the following error message:  WARNING: UNPROTECTED PRIVATE KEY FILE!

## Disallow Remote Access via root

Change SSHD Config file to disable remote access with root and enable port 2200:

- `PermitRootLogin no`

- `PasswordAuthentication no`

- `sudo nano /etc/ssh/sshd_config`

- `#Run SSH on a non standard port
Port 2200
`
- - `sudo service sshd restart`

## Ubuntu Firewall

- Make sure UFW is installed. (It is by default on Lightsail)

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
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/tcp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/tcp (v6)               ALLOW       Anywhere (v6)
5000 (v6)                  ALLOW       Anywhere (v6)


## Managing the Environment
1) Install pip:
 `sudo apt-get install python3-pip`

2) `sudo pip3 install virtualenv`

3) Create env `sudo virtualenv flask_env`

4) Activate the env by running sudo `source flask_env/bin/activate`

5) To install dependencies, run `pip3 install -r requirements.txt`

## WSGI and Apache setup

- Install apache:
`sudo apt-get install apache2`

- Install mod_wsgi:
`sudo apt-get install libapache2-mod-wsgi`

- To enable mod_wsgi:
`sudo a2enmod wsgi`

- Create a `music_inventory.conf file to apache by running:
`sudo nano /etc/apache2/sites-available/music_inventory.conf`
(See the conf file on this repo (`music_inventory.conf`)

- Then activate this conf file by running:
`sudo a2ensite music_inventory.conf`

- Make sure the other conf files are disabled by running `sudo a2dissite <NAME>`

## Cloning the webapp project repo on Git

Install git:

- `sudo apt-get install git`

## PostgreSQL setup

- `sudo apt-get install postgresql postgresql-contrib`

Upon installation, Postgres creates a Linux user called "postgres" which can be used to access the system. We can change to this user by typing:

`sudo su - postgres`

From here, we can connect to the system by typing:

`psql`

(https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
One simple way to remove a potential attack vector is to not allow remote connections to the database. This is the current default when installing PostgreSQL from the Ubuntu repositories.

We can double check that no remote connections are allowed by looking in the host based authentication file:

`sudo nano /etc/postgresql/10/main/pg_hba.conf`

Log into PostgreSQL by running:

`sudo su - postgres`
`psql`

# Creating Roles and Granting Permissions on the Database

Create role `catalog`in database:

- `psql` followed by `\password catalog`

- user catalog has limited permissions

- Allow user to login:

`ALTER ROLE "inventory_user" WITH LOGIN;`

- Connect to the catalog database:
`\c catalog`
- Grant limited powers to the user:
`GRANT CONNECT ON DATABASE catalog TO catalog;`

`GRANT SELECT ON instrument TO catalog;`
`GRANT INSERT ON instrument TO catalog;`
`GRANT UPDATE ON instrument TO catalog;`
`GRANT DELETE ON instrument TO catalog;`

(https://www.postgresql.org/docs/9.1/sql-grant.html)
(https://tableplus.io/blog/2018/04/postgresql-how-to-create-read-only-user.html)
