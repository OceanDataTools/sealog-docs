---
permalink: /client_custom/
title: "Developing a Custom Client"
layout: single
toc: false
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

Sealog Server and the Sealog clients are completely separated code bases. This was by design as it opens the possibility for institutions to develop their own custom clients that can still leverage all underlying Sealog-Server technology including the RESTful API, web-socket pubs/subs, ASNAP service, Auto-Action service, data export scripts and InfluxDB integration.

There are 3 main ways to proceed when developing a custom client:
1. Go your own way, start from scratch, build the hardware/software Sealog interface of your dreams!
2. Build a physical interface using COTS hardware such as a [Elgato StreamDeck](https://www.elgato.com/us/en/p/stream-deck-mk2-black) (yes, this has actually been done) [Here's how.]({{ "/client_streamdeck/" | relative_url }}))
3. Fork an existing client repository and modify it to meet a vessel/vehicle's unique needs (Pretty easy and often done).

For groups that opt for the thrid option, just be aware that the forked client must adhere to the stipulations of the [MIT Open Source License](https://opensource.org/license/mit).

For groups that would like assistance building a custom client, please contact the OceanDataTools.org team. We may be available to help with that.
