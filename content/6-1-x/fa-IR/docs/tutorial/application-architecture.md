# Electron Application Architecture

Before we can dive into Electron's APIs, we need to discuss the two process types available in Electron. They are fundamentally different and important to understand.

## فرآیندهای اصلی و رندرینگ

In Electron, the process that runs `package.json`'s `main` script is called __the main process__. The script that runs in the main process can display a GUI by creating web pages. یک برنامه ی الکترون همواره از یک فرآیند اصلی تشکیل میشود نه بیشتر.

از آنجا که الکترون برای نمایش صفحات وب از Chromium استفاده می‌کند، از معماری چند پردازشی هم بهرمند می‌شود. Each web page in Electron runs in its own process, which is called __the renderer process__.

در مرورگرهای معمولی، صفحات وب اغلب در محیط sandboxed اجرا می‌شوند و اجازه دسترسی به منابع اصلی ندارند. کاربران الکترون، به هر حال، قادرند با استفاده از API های Node.js در صفحات وب دسترسی به لایه های زیرین اجرایی سیستم عامل داشته باشند.

### تفاوت پردازش اصلی و پردازش رندرینگ

پروسه اصلی به وسیله خلق نمونه های BrowserWindow صفحات وب را می‌سازد. هر نمونه از BrowserWindow پروسه رندرینگ خود را اجرا می کند. وقتی که یک نمونه از BrowserWindow نابود می‌شود، پروسه رندرینگ متناظر با آن هم خاتمه می‌یابد.

The main process manages all web pages and their corresponding renderer processes. Each renderer process is isolated and only cares about the web page running in it.

در صفحات وب، دسترسی به API های رابط کاربری گرافیکی نیتیو مجاز نیست به خاطر اینکه مدیریت منابع نیتیو رابط کاربری گرافیکی در صفحات وب خیلی خطرناک است و به راحتی باعث نشتی منابع می‌شود. اگر هم بخواهید عملیات های رابط کاربری گرافیکی در صفحه وب اجرا کنید، پروسه رندر درون صفحه وب باید با ارتباط با پروسه اصلی و درخواست از آن عملیات را انجام دهد.

> #### Aside: Communication Between Processes
> 
> In Electron, we have several ways to communicate between the main process and renderer processes, such as [`ipcRenderer`](../api/ipc-renderer.md) and [`ipcMain`](../api/ipc-main.md) modules for sending messages, and the [remote](../api/remote.md) module for RPC style communication. There is also an FAQ entry on [how to share data between web pages](../faq.md#how-to-share-data-between-web-pages).

## Using Electron APIs

Electron offers a number of APIs that support the development of a desktop application in both the main process and the renderer process. In both processes, you'd access Electron's APIs by requiring its included module:

```javascript
const electron = require('electron')
```

All Electron APIs are assigned a process type. Many of them can only be used from the main process, some of them only from a renderer process, some from both. The documentation for each individual API will state which process it can be used from.

A window in Electron is for instance created using the `BrowserWindow` class. It is only available in the main process.

```javascript
// This will work in the main process, but be `undefined` in a
// renderer process:
const { BrowserWindow } = require('electron')

const win = new BrowserWindow()
```

Since communication between the processes is possible, a renderer process can call upon the main process to perform tasks. Electron comes with a module called `remote` that exposes APIs usually only available on the main process. In order to create a `BrowserWindow` from a renderer process, we'd use the remote as a middle-man:

```javascript
// This will work in a renderer process, but be `undefined` in the
// main process:
const { remote } = require('electron')
const { BrowserWindow } = remote

const win = new BrowserWindow()
```

## استفاده از API های Node.js

Electron exposes full access to Node.js both in the main and the renderer process. This has two important implications:

1) All APIs available in Node.js are available in Electron. Calling the following code from an Electron app works:

```javascript
const fs = require('fs')

const root = fs.readdirSync('/')

// This will print all files at the root-level of the disk,
// either '/' or 'C:\'.
console.log(root)
```

As you might already be able to guess, this has important security implications if you ever attempt to load remote content. You can find more information and guidance on loading remote content in our [security documentation](./security.md).

2) You can use Node.js modules in your application. Pick your favorite npm module. npm offers currently the world's biggest repository of open-source code – the ability to use well-maintained and tested code that used to be reserved for server applications is one of the key features of Electron.

As an example, to use the official AWS SDK in your application, you'd first install it as a dependency:

```sh
npm install --save aws-sdk
```

Then, in your Electron app, require and use the module as if you were building a Node.js application:

```javascript
// A ready-to-use S3 Client
const S3 = require('aws-sdk/clients/s3')
```

There is one important caveat: Native Node.js modules (that is, modules that require compilation of native code before they can be used) will need to be compiled to be used with Electron.

The vast majority of Node.js modules are _not_ native. Only 400 out of the ~650.000 modules are native. However, if you do need native modules, please consult [this guide on how to recompile them for Electron](./using-native-node-modules.md).
