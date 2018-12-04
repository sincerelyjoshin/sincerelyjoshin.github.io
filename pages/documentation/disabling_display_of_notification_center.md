---
title: "Disabling Display of Notification Center"
summary: The trackpad gesture used to access Notification Center can be disabled locally by a set of terminal commands or remotely by a script.
tags: [macos]
keywords: [macos]
sidebar: documentation_sidebar
permalink: disabling_display_of_notification_center.html
folder: documentation
last_updated:
---

The _swipe left from the right edge with two fingers_ trackpad gesture used to access Notification Center can be tracked down to a single key on two separate property lists; one for the integrated multi-touch trackpad and another for an external bluetooth multi-touch trackpad.

{% include image.html file="TrackpadTwoFingerFromRightEdgeSwipeGesture-01.png" %}

{% include note.html content="
The following will not pertain to Mac desktop products, such as iMac or Mac Mini, unless they interact with Magic Trackpad
" %}

## Integrated Multi-Touch Trackpad

The `TrackpadTwoFingerFromRightEdgeSwipeGesture` key in the `com.apple.AppleMultitouchTrackpad` preference domain controls the setting for the _Notification Center_ gesture of the internal, built-in trackpad. Setting the key to the integer value `0` has nearly the same effect as unselecting the check box in Trackpad preference pane in _System Preferences_.

**com.apple.AppleMultitouchTrackpad.plist**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>TrackpadTwoFingerFromRightEdgeSwipeGesture</key>
    <integer>0</integer>
  </dict>
</plist>
```

## External Multi-Touch Trackpad

The `TrackpadTwoFingerFromRightEdgeSwipeGesture` key in the `com.apple.driver.AppleBluetoothMultitouch.trackpad` preference domain controls the setting for the _Notification Center_ gesture of an external bluetooth trackpad. Setting the key to the integer value `0` has nearly the same effect as unselecting the check box in Trackpad preference pane in _System Preferences_.

**com.apple.driver.AppleBluetoothMultitouch.trackpad.plist**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>TrackpadTwoFingerFromRightEdgeSwipeGesture</key>
    <integer>0</integer>
  </dict>
</plist>
```

## Disabling Locally With Terminal

Running the following commands locally in _Terminal_, while logged into the targeted user account will disable the _Notification Center_ trackpad gesture upon next login.

```sh
/usr/bin/defaults write com.apple.AppleMultitouchTrackpad TrackpadTwoFingerFromRightEdgeSwipeGesture -int 0
```

```sh
/usr/bin/defaults write com.apple.driver.AppleBluetoothMultitouch.trackpad TrackpadTwoFingerFromRightEdgeSwipeGesture -int 0
```

{% include note.html content="
Desired changes will not take effect until next user session
" %}

## Disabling Remotely With Script

If you have the ability to deploy a script to your clients remotely, you will want to take advantage of the following script. It calls the Python Objective-C bridge, [_PyObjC_](https://pythonhosted.org/pyobjc/), to set the ```TrackpadTwoFingerFromRightEdgeSwipeGesture``` key to the integer value `0` in both preference domains. The script will run the commands as whomever occupies the current user session regardless of whether your management system executes as _Root_. Just as the local method using _Terminal_, this will disable the _Notification Center_ trackpad gesture upon next login.
```py
#!/usr/bin/python

import os
import pwd

from CoreFoundation import CFPreferencesSetAppValue
from SystemConfiguration import SCDynamicStoreCopyConsoleUser

def get_console_user():
    '''returns current logged in username'''
    cfuser = SCDynamicStoreCopyConsoleUser(None, None, None)
    return cfuser[0]

CONSOLE_USER = get_console_user()
USER_ID = pwd.getpwnam(CONSOLE_USER).pw_uid

os.seteuid(USER_ID)

CFPreferencesSetAppValue("TrackpadTwoFingerFromRightEdgeSwipeGesture", 0, "com.apple.AppleMultitouchTrackpad")

CFPreferencesSetAppValue("TrackpadTwoFingerFromRightEdgeSwipeGesture", 0, "com.apple.driver.AppleBluetoothMultitouch.trackpad")

exit()
```
{% include note.html content="
Desired changes will not take effect until next user session
" %}

## Disable Trackpad Preference Pane From System Preferences

To ensure that users do not tamper with the trackpad gesture settings you will want to disable the _Trackpad_ preference pane in _System Preferences_ by deploying a configuration profile. In order to disable the _Trackpad_ preference pane from _System Preferences_, exclude `com.apple.preference.trackpad` from the `EnabledPreferencePanes` array in a configuration profile assigned the `com.apple.systempreferences` payload type.  
It is also within best interest to include the `HiddenPreferencePanes` key as an empty array to prevent a bug that would render your managed preference pane payload useless.
