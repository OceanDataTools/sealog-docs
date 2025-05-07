---
permalink: /client_multiple_instances/
title: "Multiple Instances"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

It's possible to run more than one instance of Sealog on a single server or VM but doing so requires a few extra steps. 

### Server Side

When setting up a second instance of sealog-server, clone the repo to a directory other than the directory used by the first instance, which is typically `/opt/sealog-server`

Do this by specifying a target directory as part of the `git clone` operation.
```
git clone https://github.com/OceanDataTools/sealog-server.git ~/sealog-server-rov
```

Each Sealog Server instance requires it's own Mongo database with a unique name.  To set the name of the database, edit the `<INSTALL_ROOT>/config/db_constants.js` file. Set the `sealogDB` and `sealogDB_devel` variables to a something other than the default values.

For example... a vessel uses Sealog to log vessel events and wants to add a second instance to log the vehicle events.

The vessel instance can use the default database name.
For the vehicle instance use:
```
const sealogDB = 'sealogDB_rov';
const sealogDB_devel = 'sealogDB_rov_devel';
```

Each Sealog server instance requires a unique port number for server requests.  The default port is 8000.  This is defined in the `<INSTALL_ROOT>/config/manifest.js` file.  Set the `SERVER_PORT` variable to something other than the default value and a port that is not already in use.  It is common practice to set the port for additional Sealog instances to 8100, 8200, 8300, etc.

Each Sealog server instance requires a place to save files.  The default file location is `/opt/sealog-server/sealog-files`  Since this location is used by the first Sealog Server instance the second instance must be configured to look at a different location i.e. `/opt/sealog-server-rov/sealog-files`.  This path is specifed in the `<INSTALL_ROOT>/config/server_settings.js` file under the `FILEPATH_ROOT` variable.


### Supervisor Processes
To auto-start the server process and ancilary services (asnap, etc) via Supervisor, please refer to: [Server Installation]({{ "/server_install/" | relative_url }}) and [ASNAP Service]({{ "/server_asnap_setup/" | relative_url }})


### Client Side

When setting up a second instance of sealog-client, clone the repo to a directory other than the directory used by the first instance, which is typically `/opt/sealog-client`

Do this by specifying a target directory as part of the `git clone` operation.
```
git clone https://github.com/OceanDataTools/sealog-client-vehicle.git ~/sealog-client-rov
```

Here are the client-side changes required to running multiple Sealog instances on the same server.

The following changes need to be made to the `<INSTALL_ROOT>/sealog-client/src/client_settings.js` file.
1. Set `SERVER_PORT` to match the SERVER_PORT value of the server the client is supposed to communicate with.

2. Set `ROOT_PATH` to a unique URL path.  The default is '/', meaning the client should be served from the root of the web-server.  i.e. http://192.168.0.42.  If mutltiple sealog clients are being served from the same server then the URL paths need to be unique. i.e. http://192.168.0.42/ship/, http://192.168.0.42/rov/

Remember to build the clients after any settings are changed via `npm run build`


### nginx

The Sealog client is hosted from a web-server, typically nignx. Here's how to setup the `/etc/nginx/sites-available/sealog.conf`
```
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/html;
    index index.html;

    location /sealog-ship {
        index index.html;
        try_files $uri $uri/ /sealog-ship/index.html;
    }

    location /sealog-rov {
        index index.html;
        try_files $uri $uri/ /sealog-rov/index.html;
    }
}
```

Restart nignx for the changes to take affect.
```
sudo systemctl restart nginx
```

Create symlinks from the `dist` directories to the web root 
```
ln -s /opt/sealog-client/dist /var/www/html/sealog-ship
ln -s /opt/sealog-client-rov/dist /var/www/html/sealog-rov
```

### Synchronizing instances
It's possible to define on instance of Sealog to automatically replicate cruise records across multiple servers.  This way the operator creating the Sealog cruise record only has to do it once.

Setup `sealog_cruise_sync.py`
The sync functionality is performed by the `sealog_cruise_sync.py` service located in the `misc` directory of the Sealog server.

Make a copy of the sealog_cruise_sync file.
`cp /<INSTALL_ROOT>/misc/sealog_cruise_sync.py.dist /<INSTALL_ROOT>/misc/sealog_cruise_sync.py` 

The service must be run on the Sealog server instance that will server as the master. Before running the service the `SEALOG_SERVER_INSTANCES` variable must be configured.

Here's what it would look like for the example described in this guide.
```
SEALOG_SERVER_INSTANCES = [
    {
        'apiServerURL': 'http://localhost:8100/sealog-server',
        'token':'<TOKEN>'
    }
]
```

The `apiServerURL` is the root of the sealog-server-rov API.  Notice URL includes the SERVER_PORT `8100` as previously described.

The `<TOKEN>` string must be replaced with a Java Web Token for the sealog-rov instance. Please refer to: [Using the API]({{ "/server_using_api/" | relative_url }}) for how to retrieve the token.

Finally the cruise_sync service needs to be set to automatically start at boot.

Add the following to the supervisor configuration file for the master Sealog server instance:
```
[program:sealog-cruise-sync]
directory=/opt/sealog-server/misc
command=/opt/sealog-server/venv/bin/python sealog_cruise_sync.py
redirect_stderr=true
stdout_logfile=/var/log/sealog-cruise-sync_STDOUT.log
user=sealog
autostart=true
autorestart=true
stopsignal=QUIT
```

Reload the supervisor configuration via:
```
sudo supervisorctl reread
sudo supervisorctl update
```
