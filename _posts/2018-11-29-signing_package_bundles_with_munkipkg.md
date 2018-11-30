---
title:  "Signing Package Bundle Installers With munkipkg"
keywords: [packaging, pkg, certificates]
summary: Munkipkg has the ability to sign packages with a registered Apple Distribution Certificate. Here is how.
permalink: signing_package_bundle_installers_with_munkipkg.html
tags: [macos]
sidebar: blog_sidebar
folder: blog
---
## Prerequisites

In order to sign a package in a _munkipkg_ project you will need:
* membership of the paid Apple Developer Program
* _Xcode_ installed
* _munkipkg_ installed, and a working knowledge of the tool

Once all 3 criteria are met you are ready to begin signing your _munkipkg_ projects.

## Generate a Distribution Certificate

Open _Xcode_ and sign into your Apple Developer Program registered Apple ID under the Accounts tab. Once you are signed select _Manage Certificates_ in the bottom right of the _Apple ID_ window. In the popup window click the + button in the bottom left corner and select _Developer ID Installer_. The requested certificate will now display in the current window and a signing certificate will be added to your local keychain.

## Apply a Distribution Certificate to a munkipkg project

Now that you have a valid signing certificate, open a current _munkipkg_ project and a `signing-info` dictionary to the `build-info.plist`. In the `signing-info` dictionary add a key titled `identity` with a string value containing the _Common Name_ of your registered developer organization. Then add another key to the `signing-info` dictionary titled `timestamp` with the boolean value true.

**Your singing_info dictionary should look like this:**
```xml
<key>signing_info</key>
<dict>
  <key>identity</key>
  <string>Pretend Co.</string>
  <key>timestamp</key>
  <true/>
</dict>
```

Save your amendments to the `build-info.plist`.

## Build a signed munkipkg project

Back out to the parent directory of your project and run _munkipkg_. After execution you should have a shiny, new signed package installer waiting for you in the _build_ directory.
