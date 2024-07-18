---
permalink: /server_asnap_setup/
title: "ASNAP Service"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

### Automatic Snapshot (ASNAP)
Automatic snapshots or ASNAP is a service that when enabled will submit an ASNAP event to the sealog-server at a specifed interval.  The ASNAP service is useful for ensuring there is a minimum resolution of events during a cruise or lowering.  Without ASNAP there could be huge periods of a cruise or lowering where there is no information about the vessel or vehicle.

#### sealog_asnap.py
The ASNAP service is contained within the `sealog_asnap.py` script. Before use, the [python service prerequisites]({{ "/server_python_services/" | relative_url }}) must be met and the `sealog_asnap.py` script must be copied from the distributed version.
```
cd /opt/sealog-server
cp ./misc/sealog_asnap.py.dist ./misc/sealog_asnap.py
```

##### Usage Statement
```
usage: sealog_asnap.py [-h] [-v] [-i INTERVAL] [-t TIMEOUT] [--start_ts START_TS] [--stop_ts STOP_TS]

ASNAP event submission service

options:
  -h, --help            show this help message and exit
  -v, --verbosity       Increase output verbosity
  -i INTERVAL, --interval INTERVAL
                        ASNAP interval in seconds.
  -t TIMEOUT, --timeout TIMEOUT
                        set a timeout (in minutes) to stop submitting ASNAP events
  --start_ts START_TS   Add ASNAP events at the specified interval start at this specified start time (UTC)
  --stop_ts STOP_TS     Add ASNAP events until the specified stop time (UTC)
 ```

The script can be called be called in 3 ways:
1. As a service that runs until manually stopped.
2. As a service that runs for a specified amount of time.
3. As a program that submits ASNAP events to the Sealog API with timestamps between a start and stop timestamp at the defined ASNAP interval.

When run as a service the script will only submit the ASNAP events to the Sealog Server IF the `asnapStatus` customVar within the Sealog API returns `On`.
When run as a program the `asnapStatus` customVar status is ignored.

#### Customizations
To change the default interval, edit the `DEFAULT_INTERVAL` variable.  The default value is:
```
DEFAULT_INTERVAL = 10
```

To change the event submitted to the Sealog Server, edit the `ASNAP_EVENT` dict.  The default value is:
```
ASNAP_EVENT = {
    "event_value": "ASNAP",
    "event_options": [],
    "event_free_text": ""
}
```

### Setup the ASNAP service to start at boot:
Edit the supervisor configuration file:
```
sudo pico /etc/supervisor/conf.d/sealog-server.conf
```

Append the following to the supervisor configuration file (assumes the desired user is `sealog`):
```
[program:sealog-asnap]
directory=/opt/sealog-server/misc
command=/opt/sealog-server/venv/bin/python sealog_asnap.py
redirect_stderr=true
stdout_logfile=/var/log/sealog-asnap_STDOUT.log
user=sealog
autostart=true
autorestart=true
stopsignal=QUIT
```

The default ASNAP interval is 10 seconds.  To change that add the `--interval <seconds>` argument to the command statement. i.e.
```
command=/opt/sealog-server/venv/bin/python sealog_asnap.py --interval 300
```