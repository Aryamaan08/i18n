# 코드 서명(Code Signing)

코드 서명(code signing)은 이 앱을 당신이 만들었다고 인증하는데 사용하는 보안 기술입니다.

On macOS the system can detect any change to the app, whether the change is introduced accidentally or by malicious code.

On Windows, the system assigns a trust level to your code signing certificate which if you don't have, or if your trust level is low, will cause security dialogs to appear when users start using your application.  Trust level builds over time so it's better to start code signing as early as possible.

서명 되지 않은 애플리케이션을 배포할 수 있지만 권장 되지 않습니다. Both Windows and macOS will, by default, prevent either the download or the execution of unsigned applications. Starting with macOS Catalina (version 10.15), users have to go through multiple manual steps to open unsigned applications.

![macOS Catalina Gatekeeper warning: The app cannot be opened because the
developer cannot be verified](../images/gatekeeper.png)

As you can see, users get two options: Move the app straight to the trash or cancel running it. You don't want your users to see that dialog.

If you are building an Electron app that you intend to package and distribute, it should be code-signed.

# Signing & notarizing macOS builds

Properly preparing macOS applications for release requires two steps: First, the app needs to be code-signed. Then, the app needs to be uploaded to Apple for a process called "notarization", where automated systems will further verify that your app isn't doing anything to endanger its users.

To start the process, ensure that you fulfill the requirements for signing and notarizing your app:

1. (연회비 필요) [애플 개발자 프로그램](https://developer.apple.com/programs/)에 등록
2. Download and install [Xcode](https://developer.apple.com/xcode) - this requires a computer running macOS
3. [signing certificates](https://github.com/electron/electron-osx-sign/wiki/1.-Getting-Started#certificates) 생성, 다운로드, 및 설치

Electron's ecosystem favors configuration and freedom, so there are multiple ways to get your application signed and notarized.

## `electron-forge`

If you're using Electron's favorite build tool, getting your application signed and notarized requires a few additions to your configuration. [Forge](https://electronforge.io) is a collection of the official Electron tools, using [`electron-packager`], [`electron-osx-sign`], and [`electron-notarize`] under the hood.

Let's take a look at an example configuration with all required fields. Not all of them are required: the tools will be clever enough to automatically find a suitable `identity`, for instance, but we recommend that you are explicit.

```json
{
  "name": "my-app",
  "version": "0.0.1",
  "config": {
    "forge": {
      "packagerConfig": {
        "osxSign": {
          "identity": "Developer ID Application: Felix Rieseberg (LT94ZKYDCJ)",
          "hardened-runtime": true,
          "entitlements": "entitlements.plist",
          "entitlements-inherit": "entitlements.plist",
          "signature-flags": "library"
        },
        "osxNotarize": {
          "appleId": "felix@felix.fun",
          "appleIdPassword": "my-apple-id-password",
        }
      }
    }
  }
}
```

The `plist` file referenced here needs the following macOS-specific entitlements to assure the Apple security mechanisms that your app is doing these things without meaning any harm:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
    <key>com.apple.security.cs.debugger</key>
    <true/>
  </dict>
</plist>
```

To see all of this in action, check out Electron Fiddle's source code, [especially its `electron-forge` configuration file](https://github.com/electron/fiddle/blob/master/forge.config.js).


## `electron-builder`

Electron Builder comes with a custom solution for signing your application. You can find [its documentation here](https://www.electron.build/code-signing).

## `electron-packager`

If you're not using an integrated build pipeline like Forge or Builder, you are likely using [`electron-packager`], which includes [`electron-osx-sign`] and [`electron-notarize`].

If you're using Packager's API, you can pass [in configuration that both signs and notarizes your application](https://electron.github.io/electron-packager/master/interfaces/electronpackager.options.html).

```js
const packager = require('electron-packager')

packager({
  dir: '/path/to/my/app',
  osxSign: {
    identity: 'Developer ID Application: Felix Rieseberg (LT94ZKYDCJ)',
    'hardened-runtime': true,
    entitlements: 'entitlements.plist',
    'entitlements-inherit': 'entitlements.plist',
    'signature-flags': 'library'
  },
  osxNotarize: {
    appleId: 'felix@felix.fun',
    appleIdPassword: 'my-apple-id-password'
  }
})
```

The `plist` file referenced here needs the following macOS-specific entitlements to assure the Apple security mechanisms that your app is doing these things without meaning any harm:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
    <key>com.apple.security.cs.debugger</key>
    <true/>
  </dict>
</plist>
```

## Mac App Store

See the [Mac App Store Guide](mac-app-store-submission-guide.md).

# Signing Windows builds

Windows 에서 빌드를 서명 하기 전에 다음을 수행 해야 합니다.

1. Get a Windows Authenticode code signing certificate (requires an annual fee)
2. Install Visual Studio to get the signing utility (the free [Community Edition](https://visualstudio.microsoft.com/vs/community/) is enough)

You can get a code signing certificate from a lot of resellers. Prices vary, so it may be worth your time to shop around. Popular resellers include:

* [digicert](https://www.digicert.com/code-signing/microsoft-authenticode.htm)
* [Comodo](https://www.comodo.com/landing/ssl-certificate/authenticode-signature/)
* [GoDaddy](https://au.godaddy.com/web-security/code-signing-certificate)
* Amongst others, please shop around to find one that suits your needs, Google is your friend 😄

패키지 된 응용 프로그램 서명을 위한 도구가 몇개 있습니다:

- [`electron-winstaller`] will generate an installer for windows and sign it for you
- [`electron-forge`] can sign installers it generates through the Squirrel.Windows or MSI targets.
- [`electron-builder`] 는 Windows 타겟 중 일부에 서명 할 수 있습니다.

## Windows 스토어

[Windows Store Guide](windows-store-guide.md)를 확인하세요.
