# Linux-Configuration-Item-Catalog
This is P5 in the Full Stack Nanodegree program of Udacity : Linux Server Configuration

## Description
In this project I have deployed the web application that I developed earlier for Project 3: Item Catalog. Udacity and Amazon have provided a virtual server in Amazon’s Elastic Compute Cloud (EC2) for me to use for this project. The project `Item-Catalog` is live at : [http://ec2-54-68-44-235.us-west-2.compute.amazonaws.com](http://ec2-54-68-44-235.us-west-2.compute.amazonaws.com/catalog/)

#### Public IP Address

`54.68.44.235`

#### on SSH port
`2200`

## Screenshots
![API Screenshot](https://github.com/harshitcodes/Linux-Configuration-Item-Catalog/blob/master/screenshots/server_run.png)


## Configuration Steps
Following are the steps I executed while configuring my web server(apache2) and my database server. I have briefly explained what each step does:

### 1. Creation of Development Environment:

Source : [Udacity Development Environment](https://www.udacity.com/account#!/development_environment)

1. Create a new development environment.
2. Download private key from the Udacity's development environment provided to you and write down your public IP address.
3. Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
`
mv ~/Downloads/udacity_key.rsa ~/.ssh/
`
4.Set file rights (only owner can write and read.):
```
chmod 600 ~/.ssh/udacity_key.rsa
```

5. SSH login into the instance:
```
ssh -i ~/.ssh/udacity_key.rsa root@IP-ADDRESS
```

### 2. User Management: Creation of a new user and give user sudo permissions

1. Creating new user named grader
`
adduser grader
`
2. Give the grader the permission to sudo
  1. Create a file with user's name in following location /etc/sudoers.d
  `
  sudo nano /etc/sudoers.d/grader
  `
  2. Add the following lines to this file:
  `
  grader    ALL=(ALL) NOPASSWD:ALL
  `
  3. To resolve problem `unable to resolve host ip-10-20-7-186` add the folowing line in `/etc/hosts` file:
  `
  127.0.1.1 ip-10-20-18-47
  `


### 3. Give grader access via ssh and remove access from root user

1. Implementing key based authentication:
  Source: [Udacity Linux Web Server Course](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089477)
  * Open another terminal on your local machine
  * Generate key pairs on local machine and run:
  `ssh-keygen`
  * Provide the filename in the same `.ssh` folder and location to store the generated keys along with passphrase.
  * Switch to terminal connected to Linux VM in Step 1 through root user

2. Setting up for connecting to the grader user.

```
mkdir /home/grader/.ssh
cp ~/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
chgrp grader /home/grader/.ssh/authorized_keys
chown grader /home/grader/.ssh/authorized_keys
rm ~/.ssh/authorized_keys
rmdir .ssh
```

Now to remove root login edit the `/etc/ssh/sshd_config` file by changing the line `PermitRootLogin without-password` to `PermitRootLogin no`.

```
nano /etc/ssh/sshd_config
```

Now we can login the server through grader user via ssh with command:

```
ssh -i ~/.ssh/udacity_key.rsa grader@52.34.190.50
```

### 4. Updating the packages:

```
sudo apt-get update
sudo apt-get upgrade
```
### 5. Changing the port from 22 to 2200

* Edit the file /etc/ssh/sshd_config line which states 'Port 22'
* switch over to the new port by restarting SSH
* verify SSH is listening on the new port by connecting to it with option -p 2200
`
sudo nano /etc/ssh/sshd_config
sudo restart ssh
ssh -i ~/.ssh/udacity_key.rsa grader@52.34.190.50 -p 2200
`

### 6. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

  * First check the status of firewall if it is already active
  `sudo ufw status`
  * Status for now should be `inactive`

  * Configuring the firewall
  * Blocking all incoming traffic by default
  `
  sudo ufw default deny incoming
  `

  * Set to allow outgoing traffic
  `
  sudo ufw default allow outgoing
  `
  * Now start allowing incoming traffic only on needed ports i.e. SSH, HTTP, NTP

  ```
  sudo ufw allow ssh
  sudo ufw allow www
  sudo ufw allow ntp
  ```

  Adding port 2200 to the firewall as well to avoid being locked out server.
  ```
  sudo ufw allow 2200/tcp
  ```

  * Enable Firewall

  ```
  sudo ufw enable
  ```

### 7. Configure the local timezone to UTC

```
sudo cat /etc/timezone  //check if it is UTC
cd /etc
sudo rm localtime       // if not
sudo ln -s /usr/share/zoneinfo/UTC localtime
```

### 8. Install and configure Apache to serve a Python mod_wsgi application

  * Installing apache and dependencies:
  ```
  sudo apt-get install apache2
  sudo apt-get install libapache2-mod-wsgi
  ```

  * In the file `000-default.conf` append the line `WSGIScriptAlias / /var/www/html/myapp.wsgi` just
    before the closing tag `</VirtualHost>`

```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```

  * To resolve problem `AH00558: apache2: Could not reliably determine the server's fully qualified domain   name, using 127.0.1.1` add the line `ServerName localhost` in:

```
sudo nano /etc/apache2/apache2.conf
```

restart the server:

```
sudo apache2ctl restart
```

### 9. Install and configure PostgreSQL

  * Installing postgresql and server dependencies

  ```
  sudo apt-get install postgresql
  sudo apt-get install postgresql-server-dev-9.3
  ```

  * Make sure PostgreSQL does not allow remote connections. PostgreSQL is set to disable remote connections by default and it can be verified as below.

  Open the file:

  ```
  sudo nano /etc/postgresql/9.3/main/pg_hba.conf
  ```

  and the entries should look like:

  ```
  local   all             postgres                                peer

  # TYPE  DATABASE        USER            ADDRESS                 METHOD
  "local" is for Unix domain socket connections only
  local   all             all                                     peer
  # IPv4 local connections:
  host    all             all             127.0.0.1/32            md5
  # IPv6 local connections:
  host    all             all             ::1/128                 md5
  ```

  * Restart postgres

  ```
  sudo service postgresql restart
  ```

  * Start postgres

  ```
  sudo -u postgres -i
  ```

  Run `psql`

  * After we enter in psql we should execute the following commands:

  ```
  => CREATE ROLE catalog PASSWORD 'strongpassword';
  => ALTER ROLE "catalog" WITH LOGIN;
  => CREATE DATABASE catalogdb OWNER catalog;
  ```

### 10. Install git

```
sudo apt-get install git
```

### 11. Install all required python modules

```
sudo apt-get install python-dev
sudo apt-get install python-pip
sudo pip install flask==0.9
sudo pip install Werkzeug==0.8.3
sudo pip install sqlalchemy
sudo pip install oauth2client
sudo pip install psycopg2
sudo pip install Flask-SeaSurf
```

### 12. Clone the Item-Catalog-API

  * Go to /var/www/ and create a folder itemcatalog

  ```
  cd /var/www/
  mkdir itemcatalog
  cd itemcatalog
  sudo git clone https://github.com/harshitcodes/Item-Catalog-API.git ItemCatalog
  ```

This will copy the itemcatalog source to /var/www/itemcatalog/ItemCatalog

### 13. Make git web inaccessible

Create a file named `.htaccess` in `/var/www/itemcatalog` append this in `.htaccess`

```
RedirectMatch 404 /\.git
```

### 14. Enabling mod_wsgi and Apache Virtual Host Configuration

* Run the command:

```
sudo a2enmod wsgi
```

* Create a file in /etc/apache2/sites-available/ with the name of itemcatalog.conf This will hold the configuration for Virtual Host pointing towards item catalog application.

```
sudo nano /etc/apache2/sites-enabled/itemcatalog.conf
```

* Insert the following lines of code in this file itemcatalog.conf

```
<VirtualHost *:80>
      ServerName 54.68.44.235
      ServerAdmin admin@54.68.44.235
      WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi
      <Directory /var/www/itemcatalog/ItemCatalog/>
          Order allow,deny
          Allow from all
      </Directory>
      Alias /static /var/www/itemcatalog/ItemCatalog/static
      <Directory /var/www/itemcatalog/ItemCatalog/static/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Save and exit

* Create .wsgi file
  * cd into `/var/www/itemcatalog`
  * Create a file named `itemcatalog.wsgi`

  Add following python code in this file

    ```python
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/itemcatalog/")
    from ItemCatalog import app as application
    #from itemcatalog import app as application
    application.secret_key = 'super_secret_key'
    ```

* Save and exit
* Go to `/var/www/itemcatalog/ItemCatalog`

```
cd /var/www/itemcatalog/ItemCatalog
```

* Rename `/var/www/itemcatalog/ItemCatalog/application.py` to `/var/www/itemcatalog/ItemCatalog/__init__.py`

```
mv app.py __init__.py
```

### 14 Item Catalog source modifications for production

1. Removing debugging settings from `__init__.py` file

  * Open `__init__.py`

  ```
  sudo nano __init__.py
  ```

  * Remove the application's debugging settings in the end and replace with app.run() Replace this section

  ```python
  if __name__ == '__main__':
    app.secret_key = 'my_secret_key'
    app.debug = True
    app.run(host='0.0.0.0', port=8000)
  ```

  with

  ```python
  if __name__ == '__main__':
    app.run()
  ```

2. Update the Database engine creation line from `sqlite` to `postgresql`.

replace line:

```python
engine = create_engine('sqlite:///db/categorylist.db')
```

with

```
engine = create_engine('postgresql://catalog:strongpassword@localhost/catalogdb')
```

* Permissions on uploads folder
Give permissions to all for writing and reading files from static directory.

```
sudo chmod -R 777 /var/www/itemcatalog/itemcatalog/static/
```

* Use os.chdir() before reading client_secrets.json in __init__.py

Add following line

```
os.chdir(r'/var/www/itemcatalog/ItemCatalog')
```

before `CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']`. This will resolve the error of not being able to read client_secrets.json on server.

### 15. Google SignIn Configuration

* Login to console.developers.google.com and select the previously created project for item catalog
* Go to API Manager > credentials
* Pick the entry for previously configured OAuth 2.0 client ID from table
* Add `http://ec2-54-68-44-235.us-west-2.compute.amazonaws.com` and `54.68.44.235` to Authorized JavaScript Origins
* Add
`http://ec2-54-68-44-235.us-west-2.compute.amazonaws.com/oauth2callback`

`54.68.44.235/oauth2callback` to Authorized redirect URIs.

* Save Changes

### Final Steps

* Enable the site using a2ensite

```
Enable the site
```

* Restart `apache2`

```
sudo service apache2 restart
```

### And the site is live and running on [54.68.44.235](http://54.68.44.235)

#### Monitoring packages:

Install `glances` for monitoring web servers

`
sudo apt-get install Glances
`

Launch Glances to monitor system

`
glances
`

### Monitoring Screenshot

![monitoring glances log](https://github.com/harshitcodes/Linux-Configuration-Item-Catalog/blob/master/screenshots/monitor.png)


### Resources Used in configuring the application

* Server setup

  * [ Configuring Linux Web Servers course](https://www.udacity.com/course/viewer#!/c-ud299-nd/)
  * [Setting up a Firewall on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04)
  * [TimeZone in Ubuntu](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)
  * [Linux server configuration](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)

* Flask

  * [Deploying a Flask application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

* Udacity Forum

  * [How I got through it](https://discussions.udacity.com/t/p5-how-i-got-through-it/15342/)
  * [Project 5 Resources](https://discussions.udacity.com/t/project-5-resources/28343)




