---
title: "Disabling Text-to-Speech Keyboard Shortcut"
summary: Speak Selection can be disabled locally by a set of terminal commands or remotely by a script.
tags: [macos]
keywords: [macos]
sidebar: documentation_sidebar
permalink: disabling_text-to-speech_keyboard_shortcut.html
last_updated:
---

Unlike it's mobile OS counterpart, **iOS**, _Speak Selection_ for **macOS** is not identified within any payload of [Apple's MDM Specification](https://developer.apple.com/business/documentation/Configuration-Profile-Reference.pdf).

The _Speak Selection_ GUI element can be found in the _Speech_ tab of the _Accessibility_ preference pane. Toggling the _Speak selected text when the key is pressed_ checkbox on and off manipulates the `SpokenUIUseSpeakingHotKeyFlag` key within the user level property list `com.apple.speech.synthesis.general.prefs.plist`.

{% include image.html file="disable_text-to-speech_keyboard_shortcut-01.png" %}

Assigning the key the boolean value _false_ has nearly the same effect as unselecting the _Speak selected text when the key is pressed_ check box.

## Disable Locally With _Terminal_

Running the following command locally in _Terminal_, while logged into the targeted user account, will disable the _Speak Selection_ keyboard shortcut upon next login.

```sh
/usr/bin/defaults write com.apple.speech.synthesis.general.prefs SpokenUIUseSpeakingHotKeyFlag -bool false
```

{% include note.html content="
Desired changes will not take effect until next user session
" %}

## Disable Remotely With Script

If you have the ability to deploy a script to your clients remotely, you will want to take advantage of the following script. It calls the Python Objective-C bridge, [_PyObjC_](https://pythonhosted.org/pyobjc/), to set the `SpokenUIUseSpeakingHotKeyFlag` key to the boolean value `false`. The script will run the commands as whomever occupies the current user session regardless of whether your management system executes as _Root_. Just as the local method using _Terminal_, this will disable the _Speak Selection_ keyboard shortcut upon next login.

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

CFPreferencesSetAppValue("SpokenUIUseSpeakingHotKeyFlag", False, "com.apple.speech.synthesis.general.prefs")

exit()
```
{% include note.html content="
Desired changes will not take effect until next user session
" %}

## Disable _Accessibility_ Preference Pane from _System Preferences_

To ensure that users do not tamper with the _Speak Selection_ settings you will want to disable the _Accessibility_ preference pane in _System Preferences_ by deploying a configuration profile. In order to disable the _Accessibility_ preference pane from _System Preferences_, exclude `com.apple.preference.universalaccess` from the `EnabledPreferencePanes` array in a configuration profile assigned the `com.apple.systempreferences` payload type.  
It is also within best interest to include the `HiddenPreferencePanes` key as an empty array to prevent a bug that would render your managed preference pane payload useless.
