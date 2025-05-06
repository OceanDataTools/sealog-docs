---
permalink: /server_install/
title: "Server Installation"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

Sealog Server is tested on Ubuntu versions 20.04 and 22.04 but will likely work on Ubuntu 24.04, RHEL 9 and Rocky 9 with some adaptation. It is also possible to deploy the server as a container.

## Prerequisites
 - [MongoDB](https://www.mongodb.com) >=v6.x
 - [nodeJS](https://nodejs.org) >=20.x
 - [npm](https://www.npmjs.com) >=6.13.x
 - [git](https://git-scm.com)
 
 
### Installing MongoDB 6.0 on Ubuntu 22.04 LTS
Install the dependecies:
```
sudo apt install gnupg wget apt-transport-https ca-certificates software-properties-common
```

Download/Install the GPG Keys for MongoDB:
```
curl -fsSL https://pgp.mongodb.com/server-6.0.asc |  sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor
```

Setup the MongoDB Apt repository:
```
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```

Use `apt` to install MongoDB:
```
sudo apt-get update
sudo apt-get install mongodb-org
```

Start MongoDB, verify it started successfully and set service to start at boot:
```
sudo systemctl start mongod
sudo systemctl status mongod
sudo systemctl enable mongod
```

### Installing NodeJS/npm on Ubuntu 22.04 LTS
Download/run the nvm install script.  You will want to do this as the system user that will run the Sealog-Server services:
```
cd ~
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```
Install the LTS version of NodeJS (v20.x.x) using `nvm`:
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
nvm install --lts
```

## Install Sealog Server from GitHub

### Clone the Sealog Server repository
You will want to do this as the system user that will run the Sealog-Server services.
```
cd ~
git clone https://github.com/OceanDataTools/sealog-server.git
```

This will clone the repo to a `sealog-server` sub-directory.

### Create the configurations files
The settings in the default configuration files are appropriate for most single instance deployments.

Create the working configuration files from the distibution versions:
```
cd ./sealog-server
cp ./config/db_constants.js.dist ./config/db_constants.js
cp ./config/email_settings.js.dist ./config/email_settings.js
cp ./config/manifest.js.dist ./config/manifest.js
cp ./config/server_settings.js.dist ./config/server_settings.js
```

Run the following commands to create a secret JWT encryption key and save it to the `./config/secret.js` file:
```
SECRET=$(node -e "console.log(require('crypto').randomBytes(256).toString('base64'));")
ESC_SECRET=$(sed 's/[\/&]/\\&/g' <<< "$SECRET")
sed "s/''/'$ESC_SECRET'/g" ./config/secret.js.dist > ./config/secret.js
```

### Common modifications to the configuration files

#### Setup custom database name
To set a custom database name, change the `sealogDB` and `sealogDB_devel` variables in the `./config/db_constants.js` file.

#### Setup email integration
By default email integration is disabled. To setup email integration:
1. Set the `senderAddress` and `notificationEmailAddresses` variables in the `./config/email_constants.js` file.
2. Uncomment and configure the type of email integration used.  The `email_constants.js` file includes code blocks for [GMail](https://nodemailer.com/usage/using-gmail/), [Mailgun](https://www.mailgun.com/) and [Mailjet](https://www.mailjet.com/) integrations.

#### Setup a custom listening port
By default the Sealog-Server API is hosted on port 8000.  Modify the `port` variable in the `./config/manifest.js` file.

#### Setup a custom locations for event imagery and uploaded files.
By default imagery and uploaded files will be stored at `/opt/sealog-server/sealog-files`.  Change this by updating the `FILEPATH_ROOT` variable in the `./config/server_settings.js`.  If this path does not exist the server will try to create it.  If it cannot the server will throw an error and exit.  If it throws an error make sure the sealog user has permission to create directories within `FILEPATH_ROOT`

#### Enable ReCaptcha bot Protection.
Update the `reCaptchaSecret` variable in the `./config/server_settings.js` file to enable server-side ReCaptcha integration.  This requires configuring the client to use also ReCaptcha. 

### Move installation to production location
It is recommended for most Linux distributions to install Sealog Server to the `/opt` directory:
```
sudo mv sealog-server /opt/
```

### Install the nodeJS modules
Run the following commands to install the required `npm` libraries
```
cd /opt/sealog-server`
npm install
```

## Running Sealog Server
### Running in development mode
Run the following command to run the Sealog Server in development mode. This is a good was to confirm the installation was successful.
```
cd /opt/sealog-server
npm run start-devel
```

**This will start the server in development mode.**  In development mode the server process is uber-verbose and uses a development-mode database that is reset with each start (i.e. any data added from a previous run is blown away).

Running in development mode will create an admin account (user: admin. passwd: demo). 

### Running in production mode
Run the following command to run the Sealog Server in production mode:
```
cd /opt/sealog-server
npm start
```

This will start the server in **production** mode. In production mode, sealog-server will connect to the `sealogDB` MongoDB database as defined in `./config/db_constants.js`.  If the database does not exist, sealog-server will attempt to create it.  Running in production mode for the first time will create an admin account (admin:demo) and 1 regular user account (guest).  There is no password set for the regular account.

### Setting Sealog Server to start at boot
The recommended way to setup Sealog to start at boot is via Supervisor.  To install Supervisor on Ubuntu:
```
sudo apt-get install supervisor
```

Create a supervisor configuration file:
```
sudo pico /etc/supervisor/conf.d/sealog-server.conf
```

Copy/Paste the following into the file and that the desired user is `sealog`):
```
[program:sealog-server]
directory=/opt/sealog-server
command=node server.js
environment=NODE_ENV="production"
redirect_stderr=true
stdout_logfile=/var/log/sealog-server_STDOUT.log
user=sealog
autostart=true
autorestart=true
```

Start the sealog-server supervisor task:
```
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start sealog-server:
```

## Need to make everything available over port 80?
Sometimes on vessel networks it's only possible to access web-services using the standard network ports (i.e. 80, 443).  To use sealog server on these types of networks the API and websocket services will need to be tunnelled through port 80... luckily Apache makes this relatively easy.

### Prerequisites
- [apache](https://httpd.apache.org)
- [mod_proxy](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html)
- [mod_proxy_wstunnel](https://httpd.apache.org/docs/2.4/mod/mod_proxy_wstunnel.html)
 
Make sure these modules have been enabled within Apache and that Apache has been restarted since the modules were enabled.

### Update the Apache site configuration
Add the following code block to the apache site configuration (on Ubuntu this is located at: `/etc/apache2/sites-available/000-default.conf`)
 
```
ProxyPreserveHost On
ProxyRequests Off
ServerName <serverIP>
ProxyPass /sealog-server/ http://<serverIP>:8000/sealog-server/
ProxyPassReverse /sealog-server/ http://<serverIP>:8000/sealog-server/
ProxyPass /ws ws://<serverIP>:8000/
ProxyPassReverse /ws ws://<serverIP>:8000/
```

You will need to reload Apache for the changes to take affect.
```
service apache2 restart
```

If everything went correctly you should NOT be able to access the sealog-server API at `http://<serverIP>:8000/sealog-server/` and the sealog websocket service at `ws://<serverIP>:8000/ws`

## Need to make everything available over port https?
By default sealog-server runs over http.  To run the server over https set `<privKey.pem>` and `<fullchain.pem>` values within the `./config/manifest.js` file with the full paths to the cert files.

```
tlsOptions = {
  key: Fs.readFileSync(process.env.SEALOG_SERVER_TLS_PRIVKEY || '<privKey.pem>'),
  cert: Fs.readFileSync(process.env.SEALOG_SERVER_TLS_FULLCHAIN || '<fullchain.pem>')
};
```

**Note:** Make sure the user running the sealog-server process has read access to the certificate files.
## Additional Services and Advanced Setup

Please refer to these pages for information on more advanced setups and additional services:
<!-- - [Advanced Setup]({{ "/server_advanced_setup/" | relative_url }}) -->
- [Enabling Python Services]({{ "/server_python_services/" | relative_url }})
- [ASNAP Service]({{ "/server_asnap_setup/" | relative_url }})
- [Auto-Actions Service]({{ "/server_auto_action_setup/" | relative_url }})
- [InfluxDB Integration]({{ "/server_influxdb_setup/" | relative_url }})
- [Data Export]({{ "/server_data_export/" | relative_url }})
- [Advanced User Permissions ]({{ "/server_advanced_permissions/" | relative_url }})


