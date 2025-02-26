# Build Instructions (macOS)

Follow the guidelines below for building Electron on macOS.

## Prerequisites

* macOS >= 10.11.6
* [Xcode](https://developer.apple.com/technologies/tools/) >= 9.0.0
* [node.js](https://nodejs.org) (external)
* Python 2.7 with support for TLS 1.2

## Python

Please also ensure that your system and Python version support at least TLS 1.2. This depends on both your version of macOS and Python. For a quick test, run:

```sh
$ npm run check-tls
```

If the script returns that your configuration is using an outdated security protocol, you can either update macOS to High Sierra or install a new version of Python 2.7.x. To upgrade Python, use [Homebrew](https://brew.sh/):

```sh
$ brew install python@2 && brew link python@2 --force
```

If you are using Python as provided by Homebrew, you also need to install the following Python modules:

* [pyobjc](https://pypi.org/project/pyobjc/#description)

You can use `pip` to install it:

```sh
$ pip install pyobjc
```

## macOS SDK

If you're developing Electron and don't plan to redistribute your custom Electron build, you may skip this section.

המהדורות הרשמיות של Electron נבנות באמצעות [Xcode 9.4.1](http://adcdownload.apple.com/Developer_Tools/Xcode_9.4.1/Xcode_9.4.1.xip) ו־SDK 10.13 על גבי MacOS.  אפשר לבנות גם עם SDK עדכני יותר אך המהדורה הנוכחית משתמשת ב־SDK בגרסה 10.13.

## Building Electron

See [Build Instructions: GN](build-instructions-gn.md).
