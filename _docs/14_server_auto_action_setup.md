---
permalink: /server_auto_action_setup/
title: "Auto-Actions Service"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

### Auto-Actions
Auto-Actions is a service that triggered additional actions based on submitted events.  This can include setting lowering milestones in real-time via events, turning on/off the ASNAP service or really anything that can be initiated using python.  The sealog-server repository includes a boilerplate auto-actions script that is useful to users of the sealog-client-vehicle project.  In it's default behavior this script will enable/disable the ASNAP server when it detects a "Descending"/"On Surface" events.  It will also update the lowering start/stop times and lowering milestones based on the corresponding events.

#### sealog_auto_actions.py
The ASNAP service is contained within the `sealog_auto_actions.py` script. Before use, the [python service prerequisites]({{ "/server_python_services/" | relative_url }}) must be met and the `sealog_auto_actions.py` script must be copied from the distributed version.
```
cd /opt/sealog-server
cp ./misc/sealog_auto_actions.py.dist ./misc/sealog_auto_actions.py
```

#### Usage
```
usage: sealog_auto_actions.py.dist [-h] [-v]

Auto-Actions Service

options:
  -h, --help       show this help message and exit
  -v, --verbosity  Increase output verbosity

```

#### Customizations
The distributed version of the file comes configured for use with an underwater vehicle but can easily be modified for vessel-focused installations. It assumes the related event templates match what is used when the server is run in "demo-vehicle" and "demo-vessel" modes.

To use this file on a new production install it is recommended to import the event templates from the `demo` folder of the sealog-server repository.  The file to import for vehicle-focused installations is `FKt230303_S0492_eventTemplates.json`. The files to import for vessel-focued installations is `FKt230303_eventTemplates.json`.

Refer to the [UI Quick Start]({{ "/ui_quickstart/" | relative_url }}) guide for information on how to import event templates from file.

In the vehicle-focused configuration the auto-actions service will listen for events with an event_value of `VEHICLE`.  Depending on the value of a `milestone` event_option key, the service will set the value of the `asnapStatus` customVar, and/or update the timestamp of a lowering milestone and/or update the start_ts/stop_ts timestamp of the lowering record.  The mapping of the event option to action is handled by the `ASNAP_LOOKUP`, `MILESTONE_LOOKUP` and `START_STOP_LOOKUP` variables respectively.

To reconfigure for use with a vessel, comment out the `INCLUDE_SET`, `ASNAP_LOOKUP`, `START_STOP_LOOKUP` and `MILESTONE_LOOKUP` variables under the `For vehicle-focused sealog instances` section and uncomment the `INCLUDE_SET` and `ASNAP_LOOKUP` variables under the `For vessel-focused sealog instances` section.  The vessel-focus configuration does NOT have a corresponding `START_STOP_LOOKUP` or `MILESTONE_LOOKUP` because cruise records are not configured to track milestones.

This script can be easily expanded to listen for additional events and perform additional actions as needed.

### Setup the Auto-Actions service to start at boot:
Edit the supervisor configuration file:
```
sudo pico /etc/supervisor/conf.d/sealog-server.conf
```

Append the following to the supervisor configuration file (assumes the desired user is `sealog`):
```
[program:sealog-auto-actions]
directory=/opt/sealog-server/misc
command=/opt/sealog-server/venv/bin/python sealog_auto_actions.py
redirect_stderr=true
stdout_logfile=/var/log/sealog-auto-actions_STDOUT.log
user=sealog
autostart=true
autorestart=true
stopsignal=QUIT
```
