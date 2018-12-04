---
title: "Disabling Look-Up Gesture"
summary: Definition look up can be disabled with a configuration profile that can in-turn be deployed by MDM, bundled in a remotely distributable package or installed locally.
tags: [macos]
keywords: [macos]
sidebar: documentation_sidebar
permalink: disabling_look-up_gesture.html
folder: documentation
last_updated:
---

Definition lookup for macOS is identified within the _Restrictions_ payload of Apple's MDM specification. Disabling definition lookup can be accomplished by assigning the `AllowDefinitionLookup` key the boolean value `false` in a configuration profile assigned the `com.apple.applicationaccess`, or _Restrictions_, payload type.

{% include note.html content="
A configuration profile is an XML file that allows you to distribute configuration information.
" %}


## Deploy Configuration Profile With MDM

### Native Restrictions Payload

Definition Lookup should be an available option in most, if not all, MDM products under the _Restrictions_ payload of their configuration profile offerings. The benefits of using your MDM providers native _Restrictions_ payload are ease of use; simply check the box and click deploy.

{% include warning.html content="
In most cases when deploying a MDM vendor's native _Restrictions_ payload all additional keys managed in that payload are also deployed, regardless of whether you need them or not. Exercise caution as this could have adverse effects on your deployment.
" %}

### Custom Settings Payload

Should you want a less cumbersome option than your MDM's native _Restrictions_ payload you may alternatively deploy a _Custom Settings_ configuration profile. The benefit of a _Custom Settings_ configuration profile is narrower scope of impact. You define exactly which payload types and which corresponding keys you would like to manage; no more, no less.

In the case of definition lookup you would create a XML file formatted as a plist and include a single key, `allowDefinitionLookup`, assigned the boolean value `false`. You would then name this file the name of the targeted payload type, `com.apple.applicatonaccess` with the `.plist` file extension and upload it to the _Custom Settings_ payload in your MDM's _configuration profile_ menu.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>allowDefinitionLookup</key><!-- If set to false, disables definition lookup -->
    <false/>
  </dict>
</plist>
```

{% include note.html content="
Not all MDM vendors implement the same way and the above steps could vary. For example: Instead of uploading the plist file you may only need to past the contents or you may need to manually enter the payload type should your MDM not auto-populate it based upon file name.
" %}

## Deploy Configuration Profile Without MDM

The first few step to deploying a _Custom Settings_ configuration profile without MDM match that of [deploying a _Custom Settings_ configuration profile with MDM](disabling_look-up_gesture.html#custom-settings-payload). The difference being: instead of uploading the `com.apple.applicationaccess.plist` file to your MDM server, you will feed it to an open-source utility, developed by Tim Sutton, called [**mcxToProfile**](https://github.com/timsutton/mcxToProfile).

### Build Configuration Profile With mcxToProfile

Download **mcxToProfile** from GitHub as a .zip archive and extract to a convenient location. In _Terminal_ navigate to the extracted archive directory and execute **mcxToProfile** with the following parameters:

```
./mcxToProfile.py --plist ~/Path/To/com.apple.applicationaccess.plist --identifier DisableLookUp
```
This command will generate a configuration profile from `com.apple.applicationaccess.plist` and output a file called **DisableLookUp.mobileconfig** in the current working directory.

### Install Configuration Profile Locally

Opening the **DisableLookUp.mobileconfig** file will open _System Preferences_ and prompt for local installation.

### Deploy Configuration Profile Remotely

Configuration profiles can be deployed remotely without MDM by first bundling the _.mobileconfig_ file in a distributable package along with a _postinstall_ script triggering install. You could use any packaging tool or utility you prefer to build the distributable package but Tim Sutton has released an open-source utility to do this as well.

[**Make Profile Pkg**](https://github.com/timsutton/make-profile-pkg) can be downloaded as a .zip archive from GitHub. Once Downloaded extract the archive to a convenient location. Open _Terminal_, navigate to the extracted archive directory and execute **Make Profile Pkg** with the following parameters:

```
./make_profile_pkg.py /Path/To/DisableLookUp.mobileconfig --version 1.0
```

This command will generate a distributable package containing the the **DisableLookUp.mobileconfig** and a _postinstall_ script to trigger installation. The package will be located in the current working directory and will be named **DisableLookUp-1.0.pkg**. You now have a distributable package that can be deployed remotely using the tool of your choice.

## Disable Profiles Preference Pane from System Preferences

To ensure that users do not tamper with configuration profiles installed on the system you will want to disable the _Profiles_ preference pane in _System Preferences_ by deploying a configuration profile. In order to disable the _Profiles_ preference pane from _System Preferences_, exclude `com.apple.preferences.configurationprofiles` from the `EnabledPreferencePanes` array in a configuration profile assigned the `com.apple.systempreferences` payload type.  
It is also within best interest to include the `HiddenPreferencePanes` key as an empty array to prevent a bug that would render your managed preference pane payload useless.
