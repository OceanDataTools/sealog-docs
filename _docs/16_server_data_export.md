---
permalink: /server_data_export/
title: "Data Export"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

One of the first things operators will want after installing Sealog is a process for delivering data to the science party.  The sealog-server repository includes 2 files to help with the data export process.  The `sealog_vehicle_data_export.py` file is an example script for exporting sealog data from vehicle-focused installations.  The `sealog_vessel_data_export.py` file is an example script for exporting sealog data from vessel-focused installations.

These scripts without any arguments will export the most recent lowering/cruise depending on the version.  Command-line arguments can be added to specify a specific lowering/cruise to export.  The vehicle version includes to additional options: export ALL lowering for the most recent cruise, export ALL lowering for a specific cruise.

The export will include:
- event-only data in json and csv formats
- event with ancillary data in json and csv formats
- ancillary data in json format
- event templates in json format
- the cruise record in json format
- the lowering record in json format (vehicle-version only)

### Configuring the data export script:
Before using either export script the [python service prerequisites]({{ "/server_python_services/" | relative_url }}) must be met.

Next the appropriate example script needs to be enabled.
```
cd /opt/sealog-server
cp ./misc/sealog_vessel_data_export.py.dist ./misc/sealog_data_export.py
```
or
```
cd /opt/sealog-server
cp ./misc/sealog_vehicle_data_export.py.dist ./misc/sealog_data_export.py
```

### Usage Statement
```
usage: sealog_data_export.py [-h] [-v] [-c] [-C CRUISE_ID] [-L LOWERING_ID]

Sealog Deep_Discoverer Data export

options:
  -h, --help            show this help message and exit
  -v, --verbosity       Increase output verbosity
  -c, --current_cruise  export the data for the most recent cruise
  -C CRUISE_ID, --cruise_id CRUISE_ID
                        export all cruise and lowering data for the specified cruise (i.e. FK200126)
  -L LOWERING_ID, --lowering_id LOWERING_ID
                        export data for the specified lowering (i.e. S0314)
``` 

### Customization
The `sealog_data_export.py` file needs to be customized to meet the needs of the operator.

#### Sealog for Vehicles
Set the export directory and vehicle name.
```
EXPORT_ROOT_DIR = '/data/sealog-export'
VEHICLE_NAME = 'Deep_Discoverer'
```

Modify the `_cruise_file_prefix`, `_lowering_file_prefix` and `_build_lowering_name` functions to customized the output directory structure and filenames

#### Sealog for Vessels
Set the export directory and vessel name.
```
EXPORT_ROOT_DIR = '/data/sealog-export'
VESSEL_NAME = 'Discoverer'
```

Modify the `_cruise_file_prefix` function to customized the output filenames

### Extending the Export
Operators can add easily add to what products are included in the export by expanding on the `export_cruise` and `export_lowering` functions.  Examples of additional functionality include:
- Automated PDF report generation
- CSV, KML, GeoJSON formatted cruise/lowering tracklines
- OpenVDM integration 
