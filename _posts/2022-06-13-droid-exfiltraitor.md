---
title: "Test Droid-Exfiltraitor: How you can compromise Intranet Windows Machine using an Android Phone"
date: 2022-06-13
categories:
  - POCs
tags:
  - Proof of Concept
  - Security Research
  - Follina
---

|![droid](/assets/images/poc/droid-1.png)|
|:--:|
|A Malicious RTF file|

I wrote a Portable data exfiltration server app in Python-flask to demonstrate how the Follina exploit can be leveraged to exfiltrate sensitive data from the victim machine using an Android device.

|![droid](/assets/images/poc/droid-2.jpg)|
|:--:|
|Flask server running on Android|

A Malicious Person can lure Windows users to open the infected MS Word document, which takes advantage of the MS Office 0-Day vulnerability to send victim's file to the Malicious Person's Android Phone which is running [**Droid-Exfiltraitor**](https://github.com/szyth/droid-exfiltraitor).

Threat: An Internal Network with no internet connection, hence no means of MS Office Security Auto-Patch, can compromise all windows machine with just this flask-server and a malicious doc.

|![droid](/assets/images/poc/droid-3.jpg)|
|:--:|
|Successful File exfiltration to the Android Device|

This post is to educate and make people aware of the consequences of this 0 day, hence they should keep their systems up-to-date regardless if they are on an Intranet.
{: .notice--danger}

-----
