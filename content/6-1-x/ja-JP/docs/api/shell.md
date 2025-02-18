# shell

> デフォルトのアプリケーションを使用してファイルと URL を管理します。

プロセス: [メイン](../glossary.md#main-process), [レンダラー](../glossary.md#renderer-process)

`shell` モジュールは、デスクトップの統合に関する機能を提供します。

以下はユーザの既定のブラウザで URL を開く例です。

```javascript
const { shell } = require('electron')

shell.openExternal('https://github.com')
```

## メソッド

`shell` モジュールには以下のメソッドがあります。

### `shell.showItemInFolder(fullPath)`

* `fullPath` String

Show the given file in a file manager. If possible, select the file.

### `shell.openItem(fullPath)`

* `fullPath` String

戻り値 `Boolean` - アイテムが正常に開かれているかどうか.

指定したファイルをデスクトップの既定の方法で開きます。

### `shell.openExternalSync(url[, options])`

* `url` String - Windows では最大2081文字、そうでなければ関数は false を返します。
* `options` Object (任意)
  * `activate` Boolean (optional) - `true` to bring the opened application to the foreground. The default is `true`. _macOS_
  * `workingDirectory` String (任意) - 作業ディレクトリ。 _Windows_

戻り値 `Boolean` - URL を開くためにアプリケーションが利用可能かどうか。

Open the given external protocol URL in the desktop's default manner. (For example, mailto: URLs in the user's default mail agent).

**非推奨**

### `shell.openExternal(url[, options])`

* `url` String - Windows では最大2081文字です。
* `options` Object (任意)
  * `activate` Boolean (optional) - `true` to bring the opened application to the foreground. The default is `true`. _macOS_
  * `workingDirectory` String (任意) - 作業ディレクトリ。 _Windows_

戻り値 `Promise<void>`

Open the given external protocol URL in the desktop's default manner. (For example, mailto: URLs in the user's default mail agent).

### `shell.moveItemToTrash(fullPath)`

* `fullPath` String

戻り値 `Boolean` - アイテムが正常にゴミ箱に移動されたかどうか.

指定されたファイルをゴミ箱に移動し、操作の状態の Boolean を返します。

### `shell.beep()`

通知音を再生します。

### `shell.writeShortcutLink(shortcutPath[, operation], options)` _Windows_

* `shortcutPath` String
* `operation` String (optional) - Default is `create`, can be one of following:
  * `create` - 新しいショートカットを作成し、必要であれば上書きします。
  * `update` - 既にあるショートカットのみを、指定したプロパティで更新します。
  * `replace` - 既にあるショートカットを上書きし、存在しなければ失敗します。
* `options` [ShortcutDetails](structures/shortcut-details.md)

戻り値 `Boolean` - ショートカットが正常に作成されたかどうか.

`shortcutPath` のショートカットリンクを作成か更新します。

### `shell.readShortcutLink(shortcutPath)` _Windows_

* `shortcutPath` String

戻り値 [`ShortcutDetails`](structures/shortcut-details.md)

`shortcutPath` のショートカットリンクを解決します。

エラーが発生すると、例外がスローされます。
