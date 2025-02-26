# ipcMain

> Ana süreçten işleyici süreçlerine zaman uyumsuz olarak iletişim kurun.

İşlem: [Ana](../glossary.md#main-process)

`ipcMain` modülü [EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter) sınıfının bir örneğini teşkil eder. Ana işlem tarafından kullanıldığında eş zamansız işlemleri gerçekleştirir ve işleme sürecinden (web sayfası) senkronizasyon bilgisi alır. Bir işleyiciden gönderilecek mesajlar bu modüle yayılacaktır.

## Mesaj gönderiliyor

Ana işlemden yan işleme mesaj göndermek mümkündür, daha fazla bilgi için [webContents.send](web-contents.md#contentssendchannel-arg1-arg2-) komutuna bakınız.

* Bir mesaj gönderirken, etkinlik adı `channel`.
* Eşzamanlı bir mesaja cevap vermek için, `event.returnValue`yi ayarlamak gereklidir.
* Göndericiye asenkron bir mesajı geri göndermek için, `even.reply(...)` komutunu kullanabilirsiniz.  Bu yardımcı yöntemi iletileri otomatik olarak ana pencere olmayan (örn: iframes) mesajı yakalar halbuki `event.sender.send(...)` her zaman ana çerçeveye gönderir.

İşleyici ve ana işlemler arasında mesaj gönderme ve işleme ilişkin bir örneği:

```javascript
// Ana süreç içinde.
const { ipcMain } = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.reply('asynchronous-reply', 'pong')
})

ipcMain.on('synchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.returnValue = 'pong'
})
```

```javascript
// Oluşturucu işleminde (web sayfası).
const { ipcRenderer } = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // prints "pong"
})
ipcRenderer.send('asynchronous-message', 'ping')
```

## Metodlar

`ipcMain` modülü olayları dinlemek için aşağıdaki yöntemi içerir:

### `ipcMain.on(kanal, dinleyici)`

* `channel` Dizesi
* `listener` fonksiyon
  * `event` IpcMainEvent
  * `...args` herhangi[]

`listener` ile yeni bir mesaj geldiğinde `listener(event, args...)` ile çağırabilir. `channel`' ı dinler.

### `ipcMain.once(kanal, dinleyici)`

* `channel` Dizesi
* `listener` fonksiyon
  * `event` IpcMainEvent
  * `...args` herhangi[]

Adds a one time `listener` function for the event. This `listener` is invoked only the next time a message is sent to `channel`, after which it is removed.

### `ipcMain.removeListener(kanal, dinleyici)`

* `channel` Dizesi
* `listener` fonksiyon

Belirtilen `listener` öğesini `channel` öğesi için kaldırır.

### `ipcMain.removeAllListeners([kanal])`

* `channel` Dizesi

Belirtilen `kanalın` dinleyicilerini kaldırır.

## Etkinlik objesi

The documentation for the `event` object passed to the `callback` can be found in the [`ipc-main-event`](structures/ipc-main-event.md) structure docs.