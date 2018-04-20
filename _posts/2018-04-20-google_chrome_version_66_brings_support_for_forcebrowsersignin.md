---
title:  "Google Chrome version 66 brings support for ForceBrowserSignin"
categories: [macos, chrome]
summary:
permalink: google_chrome_version_66_brings_support_for_forcebrowsersignin.html
tags: [macos, chrome]
sidebar: blog_sidebar
folder: blog
---

Google Chrome version 66 has officially been released and comes with out-of-the-box support for enforcing macOS users to sign-in to a managed G-Suite domain. By enabling the ```ForceBrowserSignin``` key paired with the ```RestrictSigninToPattern``` key end users will be required to sign into a G-Suite account limited to your specification.   

In the example below; all users would be forced to sign into their pretendco.com G-Suite account before they have access to use Google Chrome.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>ForceBrowserSignin</key>
    <true/>
    <key>RestrictSigninToPattern</key>
    <array>
      <string>.*@pretendco.com</string>
    </array>
  </dict>
</plist>
```
