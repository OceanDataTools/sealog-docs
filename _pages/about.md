---
permalink: /about/
title: "About"
layout: single
toc: false
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

Sealog is a general purpose event logging framework built to support research vessels and deployed underwater vehicles.
It provides vessel/vehicle operators with an event-logging solution that can be customized to support the operator’s unique needs and provide a science party with a tool that allows them to design and enforce standardized documenting procedures and vocabularies. 

## Architecture

Sealog uses a traditional server/client architecture with the server's functionality remaining small and concise.  The sealog-server just an API that clients, scripts and application leverage to submit and retrieve event data.  Data can be submitted/retrieved via RESTful API calls or accessed asynchronously via websockets pub/sub channels.

## Short-list of features

 - 100% of functionality accessable via RESTful API, completely indenpendent of any graphical/CLI front-end.
 - Ad-hoc association of ancilary data with events such as sensor data, navigation, etc. 
 - Ability to filter events based on user, value, keywords and time spans
 - Ability to subscribe to the live eventlog feed (using websockets).
 - Simple exporting of all or a filtered list of events merged with ancilary data is JSON or CSV format
 - Defining event templates for quick event submission
 - role-based authentication using Java Web Tokens (JWT)

## Software Requirements

The Sealog server and clients have been tested against Ubuntu 20.04/22.04/24.04 and RHEL 9.  The clients are based on the React NodeJS framework and are compatible with most modern browsers.

The project code repositories are at:
 - [Sealog Server](https://github.com/oceandatatools/sealog-server)
 - [Sealog client (for vessels)](https://github.com/oceandatatools/sealog-client-vessel)
 - [Sealog client (for vehicles)](https://github.com/oceandatatools/sealog-client-vehicle)

The server and clients are made available under the [MIT Open Source License](https://opensource.org/license/mit).

Sealog is a part of the [Ocean Data Tools project](http://oceandata.tools).