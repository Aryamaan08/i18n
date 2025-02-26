# `window.open` Fonksiyon

> Yeni bir pencere aç ve URL yükle.

When `window.open` is called to create a new window in a web page, a new instance of [`BrowserWindow`](browser-window.md) will be created for the `url` and a proxy will be returned to `window.open` to let the page have limited control over it.

The proxy has limited standard functionality implemented to be compatible with traditional web pages. For full control of the new window you should create a `BrowserWindow` directly.

The newly created `BrowserWindow` will inherit the parent window's options by default. To override inherited options you can set them in the `features` string.

### `window.open(url[, frameName][, features])`

* `url` String
* `frameName` String (opsiyonel)
* `features` String (opsiyonel)

[`BrowserWindowProxy`](browser-window-proxy.md) Döndürür - Yeni bir pencere oluşturur ve `BrowserWindowProxy` sınıfının bir örneğini döndürür.

`features` karakter dizisi standart tarayıcının biçimi izler, fakat her bir özellik `BrowserWindow` seçeneğinin bir alanı olmalıdır. These are the features you can set via `features` string: `zoomFactor`, `nodeIntegration`, `preload`, `javascript`, `contextIsolation`, `webviewTag`.

Örneğin:
```js
window.open('https://github.com', '_blank', 'nodeIntegration=no')
```

**Notlar:**

* Ana pencerede devre dışı bırakılmış ise, açılan `window`'de Node entegrasyonu devre dışı bırakılacaktır.
* Eğer Ortam izolasyonu, ana pencerede etkinleştirilmiş ise, açılan `window`'da daima etkinleştirilir.
* Eğer JavaScript, ana pencerede devre dışı bırakılmış ise, açılan `window`'da daima devre dışı bırakılır.
* `features`'ta verilen standart olmayan özellikler (Chromium veya Electron tarafından ele alınmaz) `additionalFeatures` argümanı içindeki kayıtlı `webContent`'in `new-windows` olay işleyicisine geçirilir.

### `window.opener.postMessage(message, targetOrigin)`

* `message` String
* `targetOrigin` String

Belirtilen kaynak ile ana pencereye bir ileti gönderir veya `*` ile hiçbir kaynağa göndermez.

### Chrome' un `window.open()` uygulaması kullanılarak

Chrome'un dahili `window.open()` uygulamasını kullanmak istiyorsanız, `webPreferences` seçenekler nesnesinde `nativeWindowOpen` öğesini `true` olarak ayarlayın.

Yerel `window.open()`, açılan pencerelere eşzamanlı erişime izin verir; Bir iletişim kutusu veya tercihler penceresi açmanız gerekiyorsa uygun bir seçim.

Bu seçenek aynı zamanda `<webview>` etiketlerinde de ayarlanabilir:

```html
<webview webpreferences="nativeWindowOpen=yes"></webview>
```

`BrowserWindow`'un oluşturulması, `WebContents`'in `new-window` etkinliği aracılığıyla özelleştirilebilir.

```javascript
// ana süreç
const mainWindow = new BrowserWindow({
  width: 800,
  height: 600,
  webPreferences: {
    nativeWindowOpen: true
  }
})
mainWindow.webContents.on('new-window', (event, url, frameName, disposition, options, additionalFeatures) => {
  if (frameName === 'modal') {
    // pencereyi modal olarak aç
    event.preventDefault()
    Object.assign(options, {
      modal: true,
      parent: mainWindow,
      width: 100,
      height: 100
    })
    event.newGuest = new BrowserWindow(options)
  }
})
```

```javascript
// işleyici süreç (mainWindow)
let modal = window.open('', 'modal')
modal.document.write('<h1>Hello</h1>')
```
