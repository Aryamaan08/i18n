# Obyek SumberPenangkapDesktop

* `id` String - The identifier of a window or screen that can be used as a `chromeMediaSourceId` constraint when calling [`navigator.webkitGetUserMedia`]. Format pengenal akan menjadi `window:XX` atau `layar:XX`, dimana `XX` adalah sebuah nomor yang dihasilkan secara acak.
* `nama` String - Sebuah sumber layar akan dinamai, setiap`Seluruh Layar` atau `Layar <index>`, sementara itu nama sumber window akan cocok dengan judul window.
* `thumbnail` [NativeImage](../native-image.md) - Sebuah gambar thumbnail. **Note:** There is no guarantee that the size of the thumbnail is the same as the `thumbnailSize` specified in the `options` passed to `desktopCapturer.getSources`. Ukuran sesungguhnya tergantung pada ukuran dari layar atau window.
* `display_id` String - A unique identifier that will correspond to the `id` of the matching [Display](display.md) returned by the [Screen API](../screen.md). On some platforms, this is equivalent to the `XX` portion of the `id` field above and on others it will differ. It will be an empty string if not available.
* `appIcon` [NativeImage](../native-image.md) - An icon image of the application that owns the window or null if the source has a type screen. The size of the icon is not known in advance and depends on what the the application provides.
