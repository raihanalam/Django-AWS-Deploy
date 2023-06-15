# Django-AWS-Deploy

# AWS Django Deploy (EC2, RDS, S3)
This is an example/tutorial for `Django deployment` on `Heroku` and `AWS`.

# Initial Server Setup with Ubuntu 20.04
When you first create a new Ubuntu 20.04 server, you should perform some important configuration steps as part of the basic setup. These steps will increase the security and usability of your server, and will give you a solid foundation for subsequent actions.
Tutorial [Link](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)

## Step 1 — Logging in as root
To log into your server, you will need to know your `server’s public IP address`. You will also need the password or — if you installed an SSH key for authentication — the private key for the root user’s account. If you have not already logged into your server, you may want to follow our guide on [how to connect to Droplets with SSH](https://www.digitalocean.com/docs/droplets/how-to/connect-with-ssh/), which covers this process in detail.

If you are not already connected to your server, log in now as the root user using the following command (substitute the highlighted portion of the command with your server’s public IP address):
```
ssh root@your_server_ip
```
Accept the warning about host authenticity if it appears. If you are using password authentication, provide your root password to log in. If you are using an SSH key that is passphrase protected, you may be prompted to enter the passphrase the first time you use the key each session. If this is your first time logging into the server with a password, you may also be prompted to change the root password.

### About root
The root user is the administrative user in a Linux environment that has very broad privileges. Because of the heightened privileges of the root account, you are discouraged from using it on a regular basis. This is because part of the power inherent with the root account is the ability to make very destructive changes, even by accident.

The next step is setting up a new user account with reduced privileges for day-to-day use. Later, we’ll teach you how to gain increased privileges during only the times when you need them.

## Step 2 — Creating a New User
Once you are logged in as root, we’re prepared to add the new user account. In the future, we’ll log in with this new account instead of root.

This example creates a new user called `djuser`, but you should replace that with a username that you like:
```
sudo adduser djuser
```
You will be asked a few questions, starting with the account password.

Enter a strong password and, optionally, fill in any of the additional information if you would like. This is not required and you can just hit `ENTER` in any field you wish to skip.

## Step 3 — Granting Administrative Privileges
Now, we have a new user account with regular account privileges. However, we may sometimes need to do administrative tasks.

To avoid having to log out of our normal user and log back in as the root account, we can set up what is known as superuser or root privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word sudo before each command.


To add these privileges to our new user, we need to add the user to the sudo group. By default, on Ubuntu 20.04, users who are members of the sudo group are allowed to use the sudo command.

As root, run this command to add your new user to the sudo group (substitute the highlighted username with your new user):
```
sudo usermod -aG sudo djuser
```
Now, when logged in as your regular user, you can type sudo before commands to perform actions with superuser privileges.

## Step 4 — Setting Up a Basic Firewall
Ubuntu 20.04 servers can use the UFW firewall to make sure only connections to certain services are allowed. We can set up a basic firewall very easily using this application.

**Note:** If your servers are running on DigitalOcean, you can optionally use [DigitalOcean Cloud Firewalls](https://www.digitalocean.com/docs/networking/firewalls/) instead of the UFW firewall. We recommend using only one firewall at a time to avoid conflicting rules that may be difficult to debug.

Applications can register their profiles with UFW upon installation. These profiles allow UFW to manage these applications by name. OpenSSH, the service allowing us to connect to our server now, has a profile registered with UFW.

You can see this by typing:
```
sudo ufw app list
```
```
Output
Available applications:
  OpenSSH
```
We need to make sure that the firewall allows SSH connections so that we can log back in next time. We can allow these connections by typing:
```
sudo ufw allow OpenSSH
```
Afterwards, we can enable the firewall by typing:
```
sudo ufw enable
```
Type `y` and press `ENTER` to proceed. You can see that SSH connections are still allowed by typing:
```
sudo ufw status
```
```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```
As the *firewall is currently blocking all connections except for SSH*, if you install and configure additional services, you will need to adjust the firewall settings to allow traffic in. You can learn some common UFW operations in our [UFW Essentials guide](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands).


## Step 5 — Enabling External Access for Your Regular User
Now that we have a regular user for daily use, we need to make sure we can SSH into the account directly.

**Note:** Until verifying that you can log in and use sudo with your new user, we recommend staying logged in as root. This way, if you have problems, you can troubleshoot and make any necessary changes as root. If you are using a DigitalOcean Droplet and experience problems with your root SSH connection, you can [log into the Droplet using the DigitalOcean Console](https://www.digitalocean.com/docs/droplets/resources/console/).

The process for configuring SSH access for your new user depends on whether your server’s root account uses a password or SSH keys for authentication.

### If the root Account Uses Password Authentication
If you logged in to your root account using a password, then password authentication is enabled for SSH. You can SSH to your new user account by opening up a new terminal session and using SSH with your new username:
```
ssh djuser@your_server_ip
```
After entering your regular user’s password, you will be logged in. Remember, if you need to run a command with administrative privileges, type `sudo` before it like this:
```
sudo command_to_run
```
You will be prompted for your regular user password when using sudo for the first time each session (and periodically afterwards).

To enhance your server’s security, *we strongly recommend setting up SSH keys instead of using password authentication*. Follow our guide on [setting up SSH keys on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04) to learn how to configure key-based authentication.

### If the Root Account Uses SSH Key Authentication
**Note:** This step did not worked at `AWS EC2`

If you logged in to your root account using SSH keys, then password authentication is disabled for SSH. You will need to add a copy of your local public key to the new user’s `~/.ssh/authorized_keys` file to log in successfully.

Since your public key is already in the root account’s `~/.ssh/authorized_keys` file on the server, we can copy that file and directory structure to our new user account in our existing session.

The simplest way to copy the files with the correct ownership and permissions is with the `rsync` command. This will copy the root user’s `.ssh` directory, preserve the permissions, and modify the file owners, all in a single command. Make sure to change the highlighted portions of the command below to match your regular user’s name:

**Note:** The `rsync` command treats sources and destinations that end with a trailing slash differently than those without a trailing slash. When using `rsync` below, be sure that the source directory `(~/.ssh)` does not include a trailing slash (check to make sure you are not using `~/.ssh/`).
If you accidentally add a trailing slash to the command, `rsync` will copy the contents of the root account’s `~/.ssh` directory to the `sudo` user’s home directory instead of copying the entire `~/.ssh` directory structure. The files will be in the wrong location and SSH will not be able to find and use them.

```
rsync --archive --chown=djuser:djuser ~/.ssh /home/djuser
```

Now, open up a new terminal session on you local machine, and use SSH with your new username:
```
ssh djuser@your_server_ip
```
You should be logged in to the new user account without using a password. Remember, if you need to run a command with administrative privileges, type `sudo` before it like this:
```
sudo command_to_run
```
You will be prompted for your regular user password when using `sudo` for the first time each session (and periodically afterwards).


# How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 20.04
Django is a powerful web framework that can help you get your Python application or website off the ground. Django includes a simplified development server for testing your code locally, but for anything even slightly production related, a more secure and powerful web server is required.

In this guide, we will demonstrate how to install and configure some components on Ubuntu 20.04 to support and serve Django applications. We will be setting up a PostgreSQL database instead of using the default SQLite database. We will configure the Gunicorn application server to interface with our applications. We will then set up Nginx to reverse proxy to Gunicorn, giving us access to its security and performance features to serve our apps.

## Installing the Packages from the Ubuntu Repositories
To begin the process, we’ll download and install all of the items we need from the Ubuntu repositories. We will use the Python package manager pip to install additional components a bit later.

We need to update the local apt package index and then download and install the packages. The packages we install depend on which version of Python your project will use.

If you are using Django with Python 3, type:
```bash
sudo apt update
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl
```
This will install pip, the Python development files needed to build Gunicorn later, the Postgres database system and the libraries needed to interact with it, and the Nginx web server.


## (Local Machine) Creating a Python Virtual Environment for your Project
**Note:** This step only for local machine.
```
python3 -m venv venv
venv\Scripts\activate

pip install django gunicorn psycopg2-binary
```

For Herokuapp also need `whitenoise` (*Only for heroku*):
```
pip install whitenoise
```
## Creating a Python Virtual Environment for your Project
Now that we have our database, we can begin getting the rest of our project requirements ready. We will be installing our Python requirements within a virtual environment for easier management.

To do this, we first need access to the `virtualenv` command. We can install this with `pip`.

If you are using **Python 3**, upgrade pip and install the package by typing:
```bash
sudo -H pip3 install --upgrade pip
sudo -H pip3 install virtualenv
# for ubuntu 22.04
sudo apt-get install python-is-python3
# gs install
sudo apt -y install ghostscript

```

With `virtualenv` installed, we can start forming our project. Create and move into a directory where we can keep our project files:
```
cd ~
mkdir djprojectdir
cd djprojectdir
```

Within the project directory, create a Python virtual environment by typing:
```bash
virtualenv venv
```
This will create a directory called `venv` within your `djprojectdir` directory. Inside, it will install a local version of Python and a local version of `pip`. We can use this to install and configure an isolated Python environment for our project.

Before we install our project’s Python requirements, we need to activate the virtual environment. You can do that by typing:
```
source venv/bin/activate
```
With your virtual environment active, install Django, Gunicorn, and the psycopg2 PostgreSQL adaptor with the local instance of `pip`:
```
pip install django gunicorn psycopg2-binary
```
```
sudo apt install gunicorn
```
You should now have all of the software needed to start a Django project.


## Creating and Configuring a New Django Project
Now we can clone an existing django project or create new django project here.

Option-1: Create new django project
```
django-admin startproject core .
```

Option-2: Clone an existing git repository:
```
git config --global user.name "MdNazmul9"

git config --global user.email "nazmul.cse.pust@gmail.com"
```
```
git clone https://github.com/belal-bh/aws-django.git .
```

### Adjusting the Project Settings
```
DEBUG = True
# DEBUG = False

# ALLOWED_HOSTS = []
ALLOWED_HOSTS = ['aws-django-bh.herokuapp.com', '127.0.0.1']

MIDDLEWARE = [
    #...
    'whitenoise.middleware.WhiteNoiseMiddleware',
    #...
]

# Please use config file to store these database configuration
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'dbt47e9ubk01jk',
        'USER': 'lqullhrsibwgvq',
        'PASSWORD': '47cb9cd13df1850c1f9dfa88e83d4d80ffd64596a17874e14f0377072a9d6180',
        'HOST': 'ec2-54-163-47-62.compute-1.amazonaws.com',
        'PORT': '5432',
    }
}

STATIC_ROOT = BASE_DIR.joinpath("staticfiles")
```
Then have to log in heroku:
-   makemigrations
-   migrate
-   createsuperuser

### Completing Initial Project Setup
```
python3 manage.py collectstatic
```

If you followed the initial server setup guide, you should have a UFW firewall protecting your server. In order to test the development server, we’ll have to allow access to the port we’ll be using.

Create an exception for port 8000 by typing:
```
sudo ufw allow 8000
```
Finally run test:
```
python3 manage.py runserver 0.0.0.0:8000
```

## Testing Gunicorn’s Ability to Serve the Project
The last thing we want to do before leaving our virtual environment is test Gunicorn to make sure that it can serve the application. We can do this by entering our project directory and using `gunicorn` to load the project’s WSGI module:
```
gunicorn --bind 0.0.0.0:8000 core.wsgi
```
This will start Gunicorn on the same interface that the Django development server was running on. You can go back and test the app again.

We’re now finished configuring our Django application. We can back out of our virtual environment by typing:
```
deactivate
```

## Creating systemd Socket and Service Files for Gunicorn
We have tested that Gunicorn can interact with our Django application, but we should implement a more robust way of starting and stopping the application server. To accomplish this, we’ll make systemd service and socket files.

The Gunicorn socket will be created at boot and will listen for connections. When a connection occurs, systemd will automatically start the Gunicorn process to handle the connection.

Start by creating and opening a systemd socket file for Gunicorn with `sudo` privileges:
```
sudo nano /etc/systemd/system/gunicorn.socket
```
Inside, we will create a [Unit] section to describe the socket, a [Socket] section to define the socket location, and an [Install] section to make sure the socket is created at the right time:
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

Save and close the file when you are finished.

Next, create and open a systemd service file for Gunicorn with sudo privileges in your text editor. The service filename should match the socket filename with the exception of the extension:
```
sudo nano /etc/systemd/system/gunicorn.service
```
Start with the [Unit] section, which is used to specify metadata and dependencies. We’ll put a description of our service here and tell the init system to only start this after the networking target has been reached. Because our service relies on the socket from the socket file, we need to include a `Requires` directive to indicate that relationship:
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target
```
Next, we’ll open up the [Service] section. We’ll specify the user and group that we want to process to run under. We will give our regular user account ownership of the process since it owns all of the relevant files. We’ll give group ownership to the www-data group so that Nginx can communicate easily with Gunicorn.

We’ll then map out the working directory and specify the command to use to start the service. In this case, we’ll have to specify the full path to the Gunicorn executable, which is installed within our virtual environment. We will bind the process to the Unix socket we created within the /run directory so that the process can communicate with Nginx. We log all data to standard output so that the journald process can collect the Gunicorn logs. We can also specify any optional Gunicorn tweaks here. For example, we specified 3 worker processes in this case:
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=djuser
Group=www-data
WorkingDirectory=/home/djuser/djprojectdir/djproject
ExecStart=/home/djuser/djprojectdir/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          core.wsgi:application
```
Finally, we’ll add an [Install] section. This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running:
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=djuser
Group=www-data
WorkingDirectory=/home/djuser/djprojectdir/djproject
ExecStart=/home/djuser/djprojectdir/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          core.wsgi:application

[Install]
WantedBy=multi-user.target
```
With that, our systemd service file is complete. Save and close it now.

We can now start and enable the Gunicorn socket. This will create the socket file at `/run/gunicorn.sock` now and at boot. When a connection is made to that socket, systemd will automatically start the `gunicorn.service` to handle it:
```
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```
We can confirm that the operation was successful by checking for the socket file.

### Checking for the Gunicorn Socket File
Check the status of the process to find out whether it was able to start:
```
sudo systemctl status gunicorn.socket
```
You should receive an output like this:
```
Output
● gunicorn.socket - gunicorn socket
     Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor prese>
     Active: active (listening) since Fri 2020-06-26 17:53:10 UTC; 14s ago
   Triggers: ● gunicorn.service
     Listen: /run/gunicorn.sock (Stream)
      Tasks: 0 (limit: 1137)
     Memory: 0B
     CGroup: /system.slice/gunicorn.socket
```
Next, check for the existence of the `gunicorn.sock` file within the `/run` directory:
```
file /run/gunicorn.sock
```
```
Output
/run/gunicorn.sock: socket
```

If the `systemctl status` command indicated that an error occurred or if you do not find the `gunicorn.sock` file in the directory, it’s an indication that the Gunicorn socket was not able to be created correctly. Check the Gunicorn socket’s logs by typing:
```
sudo journalctl -u gunicorn.socket
```
Take another look at your `/etc/systemd/system/gunicorn.socket` file to fix any problems before continuing.

### Testing Socket Activation
Currently, if you’ve only started the `gunicorn.socket` unit, the `gunicorn.service` will not be active yet since the socket has not yet received any connections. You can check this by typing:
```
sudo systemctl status gunicorn
```
```
Output
● gunicorn.service - gunicorn daemon
   Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
```
To test the socket activation mechanism, we can send a connection to the socket through `curl` by typing:
```
curl --unix-socket /run/gunicorn.sock localhost
```
You should receive the HTML output from your application in the terminal. This indicates that Gunicorn was started and was able to serve your Django application. You can verify that the Gunicorn service is running by typing:
```
sudo systemctl status gunicorn
```
If the output from curl or the output of systemctl status indicates that a problem occurred, check the logs for additional details:
```
sudo journalctl -u gunicorn
```

Check your `/etc/systemd/system/gunicorn.service` file for problems. If you make changes to the `/etc/systemd/system/gunicorn.service` file, reload the daemon to reread the service definition and restart the Gunicorn process by typing:
```
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

Make sure you troubleshoot the above issues before continuing.

## Configure Nginx to Proxy Pass to Gunicorn
Now that Gunicorn is set up, we need to configure Nginx to pass traffic to the process.

Start by creating and opening a new server block in Nginx’s `sites-available` directory:
```
sudo nano /etc/nginx/sites-available/djproject
```
Inside, open up a new server block. We will start by specifying that this block should listen on the normal port 80 and that it should respond to our server’s domain name or IP address:
```
server {
    listen 80;
    server_name server_domain_or_IP;
}
```

Next, we will tell Nginx to ignore any problems with finding a favicon. We will also tell it where to find the static assets that we collected in our `~/djprojectdir/static` directory. All of these files have a standard URI prefix of “/static”, so we can create a location block to match those requests:
```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/djuser/djprojectdir/djproject;
    }
}
```

Finally, we’ll create a `location / {}` block to match all other requests. Inside of this location, we’ll include the standard `proxy_params` file included with the Nginx installation and then we will pass the traffic directly to the Gunicorn socket:
```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/djuser/djprojectdir/glascutr_website;
    }

    location /media/ {
        root /home/djuser/djprojectdir/glascutr_website;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```
Save and close the file when you are finished. Now, we can enable the file by linking it to the `sites-enabled` directory:
```
sudo ln -s /etc/nginx/sites-available/djproject /etc/nginx/sites-enabled
```

Test your Nginx configuration for syntax errors by typing:
```
sudo nginx -t
```
If no errors are reported, go ahead and restart Nginx by typing:
```
sudo systemctl restart nginx
```
Finally, we need to open up our firewall to normal traffic on port 80. Since we no longer need access to the development server, we can remove the rule to open port 8000 as well:
```
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'
```
You should now be able to go to your server’s domain or IP address to view your application.

**Note:** After configuring Nginx, the next step should be securing traffic to the server using SSL/TLS. This is important because without it, all information, including passwords are sent over the network in plain text.

If you have a domain name, the easiest way to get an SSL certificate to secure your traffic is using Let’s Encrypt. Follow [this guide](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04) to set up Let’s Encrypt with Nginx on Ubuntu 20.04. Follow the procedure using the Nginx server block we created in this guide.

Also this set-up collected from [this guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04?fbclid=IwAR2o3v9Rx9k7k7dtvpQv7u5USq37mjBwqPhreZGYHQRwr_bYsdrzyboGEUY).

## Everytime Need to run when something changed
```bash
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
sudo systemctl restart nginx
```

Test Nginx
```bash
sudo nginx -t
```



# Create aws deployed site  Secure Site for free:
```
sudo apt install snapd
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /user/bin/certbot
sudo certbot --nginx
2
```
