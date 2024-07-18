---
permalink: /server_advanced_permissions/
title: "Advanced User Permissions"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

Sealog provide several options for managing user access.

### Disable self-registered users pending admin approval
Any account created via the register API call is disabled.  A Sealog admin must enable the user account prior to use. To enable this feature, set the `disableRegisteringUsers` variable in `config/server_setttings.js` to `true`. Default: `false`

### Set the default role(s) of self-registered users
Defines the role(s) given to self-registered accounts. To change this setting, update the `registeringUserRoles` list variable in `config/server_settings.js`. Refer to the `scope` variable in `routes/api/v1/auth.js` for available roles.  Default: `['event_watcher', 'event_logger']`

### Activate per-cruise/per-lowering access permissions
It's possible to add per cruise and per lowering access for users. This feature must be enabled on the server AND clients. To enable this feature, set the `useAccessControl` variable in `config/server_setttings.js` to `true`. Default: `false`