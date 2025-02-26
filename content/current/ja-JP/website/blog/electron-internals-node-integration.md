---
title: 'インサイド Electron&#58; メッセージループの統合'
author: zcbenz
date: '2016-07-28'
---

Electron の内部について説明するシリーズの、最初の投稿です。 この投稿では、 Electron が Node のイベントループをどのように Chromium と統合しているかを紹介します。

---

これまで、 Node を GUI プログラミングに使う試みは数多くありました。 GTK+ のバインディングとして [node-gui](https://github.com/zcbenz/node-gui) が、 QT のバインディングとして [node-qt](https://github.com/arturadib/node-qt) が挙げられます。 しかし、 Node がイベントループに libuv を使用しているほか、メインスレッドでは一度に 1 つのイベントループしか実行できないために GUI ツールキットは独自のメッセージループを実装していたので、いずれも本番環境では動作しませんでした。 そこで、 GUI のメッセージループを Node で実行するための一般的なトリックとして、とても短い間隔のタイマーを用いてメッセージループを汲み取ることがありましたが、 GUI インターフェースの応答を遅くしたほか、多くの CPU リソースを消費していました。

Electron の開発中にも同じ問題が発生しましたが、 Node のイベントループを Chromium のメッセージループに統合するという、逆の方法を取りました。

## メインプロセスとレンダラプロセス

メッセージループの統合についての詳細の前に、 Chromium のマルチプロセスアーキテクチャについて説明します。

Electron には、メインプロセスとレンダラプロセス (これはとても単純化されているため、詳しくは [Multi-process Architecture](http://dev.chromium.org/developers/design-documents/multi-process-architecture) を参照してください) があります。 メインプロセスはウィンドウの作成など GUI の作業を行い、レンダラプロセスだけが Web ページの実行とレンダリングを行います。

Electron では JavaScript を用いてメインプロセスとレンダラプロセスの両方を制御できますが、これは Node を両方のプロセスに統合する必要があることを意味します。

## Chromium のメッセージループを libuv に置換

最初の試みは、 Chromium のメッセージループを libuv で再実装することでした。

レンダラプロセスについては、メッセージループはファイルディスクリプタとタイマーのみをリッスンすればよかったため、 libuv でインターフェースを実装するだけの簡単な作業でした。

しかし、メインプロセスではとても困難でした。 各プラットフォームは独自の GUI メッセージループを持ちます。 macOS の Chromium は `NSRunLoop` を使う一方、 Linux では glib を使います。 ネイティブな GUI メッセージループからファイルディスクリプタを展開し、繰り返しのために libuv に与える、いくつもの方法を試しましたが、いずれでも動作しないエッジケースが残りました。

最終的に、短い間隔で GUI メッセージループをポーリングするタイマーを追加しました。 その結果、プロセスは一定の CPU 使用率を持ち、操作によっては長い遅延が発生しました。

## 別のスレッドで Node のイベントループをポーリング

libuv が成熟するにつれて、別のアプローチができるようになりました。

イベントループのためにポーリングするファイルディスクリプタ (またはハンドル) の、バックエンド fd という概念が libuv に導入されました。 バックエンド fd をポーリングすることで、 libuv で新しいイベントが発生したときに通知を受けられるようになりました。

そこで Electron では、バックエンド fd をポーリングするための別スレッドを作りました。これにより、 libuv API ではなくシステムコールでポーリングを行っていたときからスレッドセーフです。 そして、 libuv のイベントループで新しいイベントがあるとき、メッセージが Chromium のメッセージループに送信され、 libuv のイベントはメインスレッドで処理されるようになりました。

この方法によって、 Chromium や Node にパッチを当てることを避け、メインプロセスとレンダラプロセスで同じコードを使うことができました。

## コード

メッセージループの統合の実装は、 [`electron/atom/common/`](https://github.com/electron/electron/tree/master/atom/common) 下の `node_bindings` ファイルで見られます。 Node を統合しようとするプロジェクトへ容易に再利用できます。

