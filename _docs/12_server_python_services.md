---
permalink: /server_python_services/
title: "Enabling Python Services"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

## Enabling Additional Python Functionality
Although Sealog-Server is written in NodeJS, many of the additional functionality is written in Python3 (v3.10+).  These additional features are completely optional and do not have to be configured to install and run Sealog but greatly add to the overall usefulness of the platform.

### Prerequisites
It is recommended to setup a python virtual environment within the sealog-server root directory from which to run the python scripts.

Install the required packages:
```
sudo apt-get install python3 python3-dev python3-pip python3-venv
```

Goto the sealog-server installation directory and create the python virtual environment.
```
cd /opt/sealog-server
python3 -m venv ./venv
```

Activate the python virtual environment and install the requried python libraries
```
source ./venv/bin/activate
pip install -r requirements.txt
deactivate
```

### Setup API Authentication
Sealog Server comes with a Python wrapper library for interacting with the Sealog API. The wrapper requires a valid JWT to authenicate with the API.  The following commands will retrieve and set the JWT for the wrapper library.  Adjust the username, password and SERVER values to match the target installation. 
```
cd /opt/sealog-server

DATA='{"username":"admin","password":"demo"}'
SERVER=http://localhost:8000

TOKEN=`curl -H "Content-Type: application/json" -X POST -d "$DATA" "$SERVER/sealog-server/api/v1/auth/login" |
    python3 -c "import sys, json; print(json.load(sys.stdin)['token'])"`

sed "s/''/'$TOKEN'/g" ./misc/python_sealog/settings.py.dist > ./misc/python_sealog/settings.py
```
