---
permalink: /client_advanced_setup/
title: "Advanced Setup"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

This page describes how the client can be customized just from the `./src/client_settings.js` file.  When any changes are made to the client codebase, the client must be re-compiled for the changed to take affect.
```
cd /opt/sealog-client
npm run build
```

#### Enable ReCaptcha bot Protection
Update the `RECAPTCHA_SITE_KEY` variable in the `./src/client_settings.js` file to enable server-side ReCaptcha integration.  This requires configuring the server to use also ReCaptcha. 

#### Customize the page header text
Use the `HEADER_TITLE` variable in the `./src/client_settings.js` to specify the branding text displayed in the top-navigation bar.

#### Customize the login page
Use the `LOGIN_SCREEN_TXT` variable in the `./src/client_settings.js` to specify the text displayed on the login page.

Use the `LOGIN_IMAGE` variable in the `./src/client_settings.js` file to specify an image to show on the login page. The image will be displayed above the `LOGIN_SCREEN_TXT`. This image must end in ".jpg", "jpeg", ".png" and located in the `./assets/images` folder.

#### Customize the cruise review page
Use the `MAIN_SCREEN_HEADER` variable in the `./src/client_settings.js` to specify the title text displayed on the Cruise Review page.

Use the `MAIN_SCREEN_TXT` variable in the `./src/client_settings.js` to specify the text displayed on the Cruise Review page.

#### Read-only client and advanced user permissions
Use the `USE_ACCESS_CONTROL` variable in the `./src/client_settings.js` to enable per-user/pre-cruise/lowering permissions. Enabling this option will allow admins to define exaclt what cruises/lowerings that users can access.  This requres setting the corresponding option on the server.  It's set to `false` by default.

Use the `DISABLE_EVENT_LOGGING` variable in the `./src/client_settings.js` to disable the client's event-logging functionality. This is commonly used when serving a shore-side copy of Sealog for post-cruise viewing. It's set to `false` by default.

#### Set the defaults for creating new cruise and lowering records
The `DEFAULT_VESSEL` variable in `./src/client_settings` will be the default "Vessel" value when adding new cruise records

The `CRUISE_ID_PLACEHOLDER` variable in `./src/client_settings` will be the hint text in the "Cruise ID" field when adding new cruise records.

The `CRUISE_ID_REGEX` variable in `./src/client_settings` will be the validation function for the "Cruise ID" field when adding/updating cruise records. Standard practice is to use the [JavaScript RegEx object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/RegExp). If the text entered for the Cruise ID returns false for the function then a warning message will be displayed under the Cruise ID text field but the form will still submit.

The `CUSTOM_CRUISE_NAME` variable in `./src/client_settings` is what a cruise should be called. Some organizations refer to cruises with different nomiclatures such as "Expedition" or "Voyage".  This variable allows the operator to customize the text used to represent a sealog cruise.  The format of the variable needs to be a list containing the singular and plural forms of the term (i.e. ['voyage', 'voyages']). The default value is ['cruise', 'cruises'].

The `LOWERING_ID_PLACEHOLDER` variable in `./src/client_settings` will be the hint text in the "Lowering ID" field when adding new cruise records.

The `LOWERING_ID_REGEX` variable in `./src/client_settings` will be the validation function for the "Lowering ID" field when adding/updating cruise records. Standard practice is to use the [JavaScript RegEx object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/RegExp). If the text entered for the Lowering ID returns false for the function then a warning message will be displayed under the Lowering ID text field but the form will still submit.

The `CUSTOM_LOWERING_NAME` variable in `./src/client_settings` is what a lowering should be called. Some organizations refer to lowerings with different nomiclatures such as "Expedition" or "Voyage".  This variable allows the operator to customize the text used to represent a sealog lowering.  The format of the variable needs to be a list containing the singular and plural forms of the term (i.e. ['dive', 'dives']). The default value is ['lowering', 'lowerings'].

#### Customized how the client interprets ancillary data

The `POSITION_DATASOURCES` variable in `./src/client_settings` lists the aux_data record data_source names that should be interpreted and postion data. This information is used in the sections of the client with maps showing vessel/vehicle positions. The values in the list should be in the preferred order of use. The default value is: ['vehicleRealtimeNavData'].

The `EXCLUDE_AUX_DATA_SOURCES` variable in `./src/client_settings` lists the aux_data record data_source names that should NOT be displayed in the client. The default value is: []

The `IMAGES_AUX_DATA_SOURCES` variable in `./src/client_settings` lists the aux_data record data_source names that should be interpreted as imagery data. The client will try to show the referenced image file instead of just the image filename. The default value is: ['framegrabber']

The `AUX_DATA_SORT_ORDER` variable in `./src/client_settings` lists the aux_data record data_source names in the order they should be displayed on pages show event details. The default value is: ['vehicleRealtimeNavData']

The `AUX_DATA_DATASOURCE_REPLACE` variable in `./src/client_settings` is used to define how to display the aux_data data_source name on pages showing event details.  The default behavior is to split the native camel-case data_source variable names to individual words (vehicleRealtimeNavData -> Vehicle Realtime Nav Data). This can produce unintented results where capital letters are intentionally placed next to each other (vehicleRealtimeCTDData -> Vehicle Realtime C T D Data'). The `AUX_DATA_DATASOURCE_REPLACE` needs to be a JSON-formated object of key/value pairs where the key is the raw data_source name and the value is the string to display in the client: { 'vehicleRealtimeCTDData': 'Vehicle Realtime CTD Data', ... }.  The default value is: null
