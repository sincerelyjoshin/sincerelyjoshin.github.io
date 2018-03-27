---
title: "Enforce G-Suite user login for macOS"
summary: Workflow requiring macOS clients to sign in to Google Chrome with a managed account within a whitelisted domain to enforce policies set at Google Admin Console.
tags: [macos, google, chrome]
keywords: [google, chrome, macos]
sidebar: documentation_sidebar
permalink: enforce_g-suite_user_login_for_macos.html
folder: documentation
last_updated:
---
As of Google Chrome version 64 Windows has had access to the [```ForceBrowserSignin```](https://www.chromium.org/administrators/policy-list-3#ForceBrowserSignin) management key requiring users to sign in to Google Chrome with their profile before using the browser. When paired with the [```RestrictSigninToPattern```](https://www.chromium.org/administrators/policy-list-3#RestrictSigninToPattern) key users are not only forced to sign in to their browser but are also restricted to whitelisted domains. Support for ```ForceBrowserSignin```will supposedly be available for macOS in version 66, but until then we all still have a job to do.   

##  OBJECTIVE

Force users to sign into Google Chrome by blocking all ```http://``` and ```https://``` traffic besides the [Google accounts sign in page](https://accounts.google.com). Once a user is signed in to Google Chrome with a managed account within a whitelisted domain traffic will be restored and users will be subject to all additional policies configured in the Google Admin Console.  

**THIS IS ACCOMPLISHED VIA 3 COMPONENTS:**
  1. **Launch Agent** - blacklists all ```http://``` and ```https://``` traffic (Policy Level - Recommended)

  2. **Configuration Profile** - whitelists accounts.google.com and whitelists approved G_Suite domains (Policy Level - Mandatory)

  3. **Chrome Management from Google Admin Console** - blacklists foo://bar and adds all```http://``` and ```https://``` traffic to the blacklist exception (Policy Level - Mandatory)

{% include tip.html content="

Basic knowledge of Google Chrome policy is recommended to understand how the 3 components work with each other. [Chrome Policy FAQs](https://support.google.com/chrome/a/answer/187202).

" %}

## COMPONENTS

### LAUNCH AGENT

The launch agent will run the following command on user login:
```sh
defaults write com.google.Chrome URLBlacklist -array "http://*" "https://*"
```
This will block all ```http://``` and ```https://``` traffic when using Google Chrome at the Recommended policy level. A policy that is at Recommended level can be easily overwritten

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.yourcompany.studentgoogleconfig</string>
	<key>LowPriorityIO</key>
	<false/>
	<key>Nice</key>
	<integer>0</integer>
	<key>ProgramArguments</key>
	<array>
		<string>/bin/sh</string>
		<string>-c</string>
		<string>defaults write com.google.Chrome URLBlacklist -array &apos;&quot;http://*&quot;&apos; &apos;&quot;https://*&quot;&apos;</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
</dict>
</plist>
```

### CONFIGURATION PROFILE

The Configuration Profile for preference domain com.google.Chrome will whitelist the URL accounts.google.com so that users can access the Google login page. The ``RestrictSigninToPattern`` key will also be managed here to limit which domains can login.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>RestrictSigninToPattern</key>
	<string>*@yourcompany.com</string>
	<key>URLWhitelist</key>
	<array>
		<string>accounts.google.com</string>
	</array>
</dict>
</plist>
```

### CHROME MANAGEMENT - GOOGLE ADMIN CONSOLE

{% include warning.html content="

Maintain isolation from any production Google Suite environment. Create test users, create test groups, don't lose your job.

" %}  

From <a href="https://admin.google.com/">Google Admin Console</a>, Chrome management needs to be configured with a URL Blacklist and a URL Blacklist Exception. I recommend setting the blacklist to a bogus URL with a bogus protocol: ``foo://bar``. The Blacklist Exception needs to wildcard all ``http://*`` and ``https://*`` traffic.

{% include image.html file="GoogleAdminConsole-ChromeManagement.png" %}
