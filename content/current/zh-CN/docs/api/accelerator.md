# 快捷键

> 定义键盘快捷键。

快捷键可以包含多个功能键和一个键码的字符串，由符号`+`结合，用来定义你应用中的键盘快捷键

示例

* `CommandOrControl+A`
* `CommandOrControl+Shift+Z`

快捷方式使用 [` register `](global-shortcut.md#globalshortcutregisteraccelerator-callback) 方法在 [` globalShortcut `](global-shortcut.md) 模块中注册, 即:

```javascript
const { app, globalShortcut } = require('electron')

app.on('ready', () => {
  // Register a 'CommandOrControl+Y' shortcut listener.
  globalShortcut.register('CommandOrControl+Y', () => {
    // Do stuff when Y and either Command/Control is pressed.
  })
})
```

## 跨平台提醒

在 Linux 和 Windows 上, ` Command ` 键没有任何效果, 所以使用 ` CommandOrControl `表述, macOS 是 ` Command ` ，在 Linux 和 Windows 上是` Control `。

Use `Alt` instead of `Option`. The `Option` key only exists on macOS, whereas the `Alt` key is available on all platforms.

`Super`键是指 Windows 和 Linux 系统上的 `Windows` 键，但在 macOS 里为 `Cmd` 键.

## 可用的功能键

* `Command` (缩写为`Cmd`)
* `Control` (缩写为`Ctrl`)
* `CommandOrControl` (缩写为 `CmdOrCtrl`)
* `Alt`
* `Option`
* `AltGr`
* `Shift`
* `Super`

## 可用的普通按键

* `0` 到 `9`
* `A` 到 `Z`
* `F1` 到 `F24`
* 类似`~`, `!`, `@`, `#`, `$`的标点符号
* `Plus`
* `Space`
* `Tab`
* `大写锁定（Capslock）`
* `数字锁定（Numlock）`
* `滚动锁定`
* `Backspace`
* `Delete`
* `Insert`
* `Return` (等同于 `Enter`)
* `Up`, `Down`, `Left` and `Right`
* `Home` 和 `End`
* `PageUp` 和 `PageDown`
* `Escape` (缩写为 `Esc`)
* `VolumeUp`, `VolumeDown` 和 `VolumeMute`
* ` MediaNextTrack `、` MediaPreviousTrack `、` MediaStop ` 和 ` MediaPlayPause `
* `PrintScreen`
* 小键盘按键
  * `num1`-`num9` -数字1-数字9
  * `numdec` - 小数点
  * `numadd` - 加号
  * `numsub` - 减号
  * `nummult` - 乘号
  * `numdiv` - 除号
