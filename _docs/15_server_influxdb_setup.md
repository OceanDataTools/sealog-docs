---
permalink: /server_influxdb_setup/
title: "InfluxDB Integration"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

### Aux Data Inserter for InfluxDB

Aux-Data-Inserter-Influx is a service that associates ancillary data queried from InfluxDB with events. Examples of ancilary data include vessel/vehicle position and sensor data.

An internal yaml-formated string or optional externally supplied yaml-formatted file defines what data to query from InfluxDB and how to format the resulting aux_data records submitted to the Sealog Server API.

#### sealog_aux_data_inserter_influx.py
The service is contained within the `sealog_aux_data_inserter_influx.py` script. Before use, the [Python service prerequisites]({{ "/server_python_services/" | relative_url }}) and [InfluxDB service prerequisites](#setup-influxdb-integration) must be met and the `sealog_aux_data_inserter_influx.py` script must be copied from the distributed version.
```
cd /opt/sealog-server
cp ./misc/sealog_aux_data_inserter_influx.py.dist ./misc/sealog_aux_data_inserter_influx.py
```

#### Usage
```
usage: sealog_aux_data_inserter_influx.py [-h] [-v] [-f CONFIG_FILE] [-n] [-e EVENTS] [-c CRUISE_ID] [-l LOWERING_ID]

Aux Data Inserter Service - InfluxDB

options:
  -h, --help            show this help message and exit
  -v, --verbosity       Increase output verbosity
  -f CONFIG_FILE, --config_file CONFIG_FILE
                        use the specifed configuration file
  -n, --dry_run         compile the aux_data records but do not submit to server API
  -e EVENTS, --events EVENTS
                        list of event_ids to apply the influx data
  -c CRUISE_ID, --cruise_id CRUISE_ID
                        cruise_id to fix aux_data for
  -l LOWERING_ID, --lowering_id LOWERING_ID
                        lowering_id to fix aux_data for
```

The script can be called be called in 3 ways:
1. As a service that runs until manually stopped.
2. As a program that upserts (updates or inserts) aux_data records for the specified list of event IDs
3. As a program that upserts aux_data records for the specified cruise/lowering.

If a `CONFIG_FILE` is NOT specified via the `-f` argument the script will use the internally defined YAML object.

#### Config YAML Format
Let's look at the default YAML object included in the script:
```
- data_source: realtimeVesselPosition
  query_measurements:
    - seapath1
  aux_record_lookup:
    S1HeadingTrue:
      name: heading
      uom: deg
      round: 2
    S1Latitude:
      name: latitude
      uom: ddeg
      round: 6
      modify:
        - test:
            - field: S1NorS
              eq: S
          operation:
            - multiply: -1
    S1Longitude:
      name: longitude
      uom: ddeg
      round: 6
      modify:
        - test:
            - field: S1EorW
              eq: W
          operation:
            - multiply: -1
    S1NorS:
      no_output: true
    S1EorW:
      no_output: true
```

This object is setup to build the following aux_data record:
```
    {
        "event_id": "<event_id>",
        "data_source": "vesselRealtimePosition",
        "data_array":
        [
            {
                "data_name": "heading",
                "data_value": "181.15",
                "data_uom": "deg"
            },
            {
                "data_name": "latitude",
                "data_value": "18.533928",
                "data_uom": "ddeg"
            },
            {
                "data_name": "longitude",
                "data_value": "-66.128473",
                "data_uom": "ddeg"
            }
        ]
    }
```

The `heading` field requires no manipulation and is directly applied to the output.  In this scenario the latitude and longitude values are stored independent of hemisphere. The desired output is to change the sign of the value based on hemisphere.  To accomplish this the yaml object includes a modify sub-object that looks at value of the hemisphere data and performs a mathmatical operation to flip the sign.  Although the hemisphere data is required to perform the modification, it should not be included in the output.  This is defined by setting the `no_output` key/value pair for the influx field to `true`.

The `round` key/value numerically rounds the data point to the desired precision.

Additional definitions can be to this YAML string to have the service insert multiple aux_data records for a given event.  This only requirement is that value of the `data_source` key must be unique.

If a data point needs to be modified consistently (no test required) then the `test` portion of the `modify` sub-object can be omitted. An example of this would be to flip the sign of a GGA altitude from a USBL source so that the depth can be shown as a positive number.
```
  aux_record_lookup:
    GGAAltitude:
      name: depth
      uom: m
      round: 2
      modify:
        - operation:
            - multiply: -1
```

The available modify tests are:
- eq: value is equal
- gt: value is greater than
- gte: value is greater than or equal
- lt: value is less than
- lte: value is less than or equal
- ne: value is not equal

The available modify operations are:
- add
- subtract
- multiple
- divide

### Setup the Aux-Data-Inserter-Influx service to start at boot:
Edit the supervisor configuration file:
```
sudo pico /etc/supervisor/conf.d/sealog-server.conf
```

Append the following to the supervisor configuration file (assumes the desired user is `sealog`):
```
[program:sealog-aux-data-inserter-influx]
directory=/opt/sealog-server/misc
command=/opt/sealog-server/venv/bin/python sealog_aux_data_inserter_influx.py -f ./embed_config.yaml
redirect_stderr=true
stdout_logfile=/var/log/sealog-aux-data-inserter-influx_STDOUT.log
user=sealog
autostart=true
autorestart=true
stopsignal=QUIT
```

### Setup InfluxDB Integration
Sealog Server comes with a InfluxDB wrapper for communicating with InfluxDB. The wrapper requires a valid server address, organization, bucket name and token.  This information must be saved in the `./misc/influx_sealog/settings.py` file.  If this file does not already exist, create it from the distributed version
```
cd /opt/sealog-server
cp ./misc/influx_sealog/settings.py.dist ./misc/influx_sealog/settings.py
```

Refer to InfluxDB documentation on creating and retrieving this information.

If the vessel/vehicle uses OpenRVDAS to push data to InfluxDB then this information can be copy/pasted from `/<install_root>/openrvdas/database/influxdb/settings.py`
