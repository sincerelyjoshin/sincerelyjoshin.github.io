---
title: "Disabling Display of Notification Center"
summary:
tags: [macos]
keywords: [macos]
sidebar: documentation_sidebar
permalink: disabling_display_of_notification_center.html
folder: documentation
last_updated:
---

The 'swipe left from the right edge with two fingers' Notification Center trackpad gesture can be tracked down to a single key on two seperate plist's. One for the integrated multitouch trackpad, com.apple.AppleMultitouchTrackpad.plist and another for the external bluetooth trackpad, com.apple.driver.AppleBluetoothMultitouch.trackpad.plist.

> Obliously not relevant for Mac desktop products, such as iMac or Mac Mini, unless they interact with a Magic Trackpad

## Targeted Plists and Associated Key(s)

### com.apple.AppleMultitouchTrackpad.plist

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<!-- com.apple.AppleMultitouchTrackpad.plist -->
  <dict>
    <key>TrackpadTwoFingerFromRightEdgeSwipeGesture</key><!-- If set to 0, disables notification center gesture -->
    <integer>0</integer>
  </dict>
</plist>
```

### com.apple.driver.AppleBluetoothMultitouch.trackpad.plist


```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<!-- com.apple.driver.AppleBluetoothMultitouch.trackpad.plist -->
  <dict>
    <key>TrackpadTwoFingerFromRightEdgeSwipeGesture</key><!-- If set to 0, disables notification center gesture -->
    <integer>0</integer>
  </dict>
</plist>
```

## Disabling with Terminal

`/usr/bin/defaults write com.apple.AppleMultitouchTrackpad TrackpadTwoFingerFromRightEdgeSwipeGesture -int 0`

`/usr/bin/defaults write com.apple.driver.AppleBluetoothMultitouch.trackpad TrackpadTwoFingerFromRightEdgeSwipeGesture -int 0`

> Note: Desired changes require user to log out and back in again

## Disabling with Python Script

```
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

> Note: Desired changes require user to log out and back in again

## Ensure Trackpad Gesture Remains Disabled

### Disable Trackpad Preference Pane from System Preferences

To exclude the Trackpad System Preference pane from System Preferences, exclude `com.apple.preference.trackpad` from the EnabledPreferencePanes array in a `com.apple.systempreferences` configuration profile.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<!-- com.apple.systempreferences.plist -->
  <dict>
    <key>EnabledPreferencePanes</key><!-- -->
    <array>
      <string>com.apple.Localization</string>
      <string>com.apple.preferences.Bluetooth</string>
      <string>com.apple.preference.desktopscreeneffect</string>
      <string>com.apple.preference.displays</string>
      <string>com.apple.preference.digihub.discs</string>
      <string>com.apple.preference.dock</string>
      <string>com.apple.preference.expose</string>
      <string>com.apple.preference.general</string>
      <string>com.apple.preference.mouse</string>
      <string>com.apple.preference.printfax</string>
      <string>com.apple.preference.sound</string>
      <!-- <string>com.apple.preference.trackpad</string> comment out to disable Trackpad preference pane -->
    </array>
    <key>HiddenPreferencePanes</key><!-- leave an empty array -->
    <array/>
  </dict>
</plist>
```
