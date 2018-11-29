---
title: "Disabling Display of Notification Center"
summary: The following details how to disable the trackpad gesture used to access Notification Center.
tags: [macos]
keywords: [macos]
sidebar: documentation_sidebar
permalink: disabling_display_of_notification_center.html
folder: documentation
last_updated:
---

The *swipe left from the right edge with two fingers* trackpad gesture used to access Notification Center can be tracked down to a single key on two separate property lists; one for the integrated multi-touch trackpad and another for an external bluetooth multi-touch trackpad.

{% include image.html file="TrackpadTwoFingerFromRightEdgeSwipeGesture-01.png" %}

{% include note.html content="
The following will not pertain to Mac desktop products, such as iMac or Mac Mini, unless they interact with Magic Trackpad
" %}

## Integrated Multi-Touch Trackpad

The `TrackpadTwoFingerFromRightEdgeSwipeGesture` key in the `com.apple.AppleMultitouchTrackpad` preference domain controls the setting for the Notification Center gesture of the internal, built-in trackpad. Setting the key to the integer value of 0 has nearly the same effect as unselecting the check box in Trackpad preference pane in System Preferences.

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

The `TrackpadTwoFingerFromRightEdgeSwipeGesture` key in the `com.apple.driver.AppleBluetoothMultitouch.trackpad` preference domain controls the setting for the Notification Center gesture of an external bluetooth trackpad. Setting the key to the integer value of 0 has nearly the same effect as unselecting the check box in Trackpad preference pane in System Preferences.

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

## Disabling Locally with Terminal

Running the following commands locally in Terminal, while logged into the targeted user account will disable the Notification Center trackpad gesture upon next login.

```sh
/usr/bin/defaults write com.apple.AppleMultitouchTrackpad TrackpadTwoFingerFromRightEdgeSwipeGesture -int 0
```

```sh
/usr/bin/defaults write com.apple.driver.AppleBluetoothMultitouch.trackpad TrackpadTwoFingerFromRightEdgeSwipeGesture -int 0
```

{% include note.html content="
Desired changes will not take effect until next user session
" %}

## Disabling Remotely with Script

If you have the ability to deploy a script to your clients remotely, you will want to take advantage of the following script. It calls the Python Objective-C bridge, [PyObjC](https://pythonhosted.org/pyobjc/), to set the ```TrackpadTwoFingerFromRightEdgeSwipeGesture``` key to the integer value 0 in both preference domains. The script will run the commands as whomever occupies the current user session regardless of whether your management system executes as root. Just as the local method using terminal, this will disable the Notification Center trackpad gesture upon next login.
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

## Disable Trackpad Preference Pane from System Preferences

To ensure that users do not tamper with the Trackpad Gesture settings you will want to disable the Trackpad preference pane in System Preferences by deploying a configuration profile. In order to disable the Trackpad System Preference pane from System Preferences, exclude `com.apple.preference.trackpad` from the `EnabledPreferencePanes` array in a configuration profile targeting the `com.apple.systempreferences` preference domain.  
It is also within best interest to include the key `HiddenPreferencePanes` as an empty array to prevent a bug that would render your managed preference pane payload useless.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>EnabledPreferencePanes</key><!-- -->
    <array>
      <string>com.apple.Localization</string>
      <strint>com.apple.preference.datetime</string>
      <string>com.apple.preference.desktopscreeneffect</string>
      <string>com.apple.preference.digihub.discs</string>
      <string>com.apple.preference.displays</string>
      <string>com.apple.preference.dock</string>
      <string>com.apple.preference.energysaver</string>
      <string>com.apple.preference.expose</string>
      <string>com.apple.preference.general</string>
      <string>com.apple.preference.ink</string>
      <string>com.apple.preference.keyboard</string>
      <string>com.apple.preference.mouse</string>
      <string>com.apple.preference.network</string>
      <string>com.apple.preference.notifications</string>
      <string>com.apple.preference.printfax</string>
      <string>com.apple.preference.security</string>
      <string>com.apple.preference.sound</string>
      <string>com.apple.preference.speech</string>
      <string>com.apple.preference.spotlight</string>
      <string>com.apple.preference.startupdisk</string>
      <!-- <string>com.apple.preference.trackpad</string> comment out to disable Trackpad preference pane -->
      <string>com.apple.preference.universalaccess</string>
      <string>com.apple.preferences.appstore</string>
      <string>com.apple.preferences.Bluetooth</string>
      <string>com.apple.preferences.configurationprofiles</string>
      <string>com.apple.preferences.extensions</string>
      <string>com.apple.preferences.icloud</string>
      <string>com.apple.preferences.internetaccounts</string>
      <string>com.apple.preferences.parentalcontrols</string>
      <string>com.apple.preferences.password</string>
      <string>com.apple.preferences.sharing</string>
      <string>com.apple.preferences.users</string>
      <string>com.apple.preferences.wallet</string>
      <string>com.apple.prefpanel.fibrechannel</string>
      <string>com.apple.prefs.backup</string>
    </array>
    <key>HiddenPreferencePanes</key><!-- leave an empty array -->
    <array/>
  </dict>
</plist>
```
