---
title: 'Electron Internals: Building Chromium as a Library'
author: zcbenz
date: '2017-03-03'
---

Electron は、Google のオープンソースプロジェクト Chromium をベースにしています。このプロジェクトは、他のプロジェクトで使用することを想定していません。 この記事では、Electron のライブラリとしての Chromium がどのように構築されているのか、またビルドシステムがどのように進化してきたのかを紹介します。

---

## CEF の利用

Chromium Embedded Framework (CEF) は、Chromium をライブラリ化し、Chromium のコードベースに基づいて安定した API を提供するプロジェクトです。 黎明期の Atom エディタや NW.js では CEF を使用していました。

安定した API を維持するために、CEF は Chromium の詳細をすべて隠蔽し、独自のインターフェースで Chromium の API をラップします。 そのため、Node.js をウェブページに統合するような、内部の Chromium API にアクセスする必要があったとき、CEF の利点が障害になりました。

そのため、結局 Electron も NW.js も Chromium の API を直接使うように切り替えました。

## Chromium を部品としてビルドする

Chromium は公式には外部プロジェクトをサポートしていませんが、コードベースはモジュール化されており、Chromium をベースに小さなブラウザを簡単に構築できます。 ブラウザインターフェイスを提供するコアモジュールは、コンテンツモジュールと呼ばれています。

To develop a project with Content Module, the easiest way is to build the project as part of Chromium. This can be done by first checking out Chromium's source code, and then adding the project to Chromium's `DEPS` file.

NW.js and very early versions of Electron are using this way for building.

The downside is, Chromium is a very large codebase and requires very powerful machines to build. For normal laptops, that can take more than 5 hours. So this greatly impacts the number of developers that can contribute to the project, and it also makes development slower.

## Building Chromium as a single shared library

As a user of Content Module, Electron does not need to modify Chromium's code under most cases, so an obvious way to improve the building of Electron is to build Chromium as a shared library, and then link with it in Electron. In this way developers no longer need to build all off Chromium when contributing to Electron.

The [libchromiumcontent](https://github.com/electron/libchromiumcontent) project was created by [@aroben](https://github.com/aroben) for this purpose. It builds the Content Module of Chromium as a shared library, and then provides Chromium's headers and prebuilt binaries for download. The code of the initial version of libchromiumcontent can be found [in this link](https://github.com/electron/libchromiumcontent/tree/873daa8c57efa053d48aa378ac296b0a1206822c).

The [brightray](https://github.com/electron/brightray) project was also born as part of libchromiumcontent, which provides a thin layer around Content Module.

By using libchromiumcontent and brightray together, developers can quickly build a browser without getting into the details of building Chromium. And it removes the requirement of a fast network and powerful machine for building the project.

Apart from Electron, there were also other Chromium-based projects built in this way, like the [Breach browser](https://www.quora.com/Is-Breach-Browser-still-in-development).

## Filtering exported symbols

On Windows there is a limitation of how many symbols one shared library can export. As the codebase of Chromium grew, the number of symbols exported in libchromiumcontent soon exceeded the limitation.

The solution was to filter out unneeded symbols when generating the DLL file. It worked by [providing a `.def` file to the linker](https://github.com/electron/libchromiumcontent/pull/11/commits/85ca0f60208eef2c5013a29bb4cf3d21feb5030b), and then using a script to [judge whether symbols under a namespace should be exported](https://github.com/electron/libchromiumcontent/pull/47/commits/d2fed090e47392254f2981a56fe4208938e538cd).

By taking this approach, though Chromium kept adding new exported symbols, libchromiumcontent could still generate shared library files by stripping more symbols.

## コンポーネントのビルド

Before talking about the next steps taken in libchromiumcontent, it is important to introduce the concept of component build in Chromium first.

As a huge project, the linking step takes very long in Chromium when building. Normally when a developer makes a small change, it can take 10 minutes to see the final output. To solve this, Chromium introduced component build, which builds each module in Chromium as separated shared libraries, so the time spent in the final linking step becomes unnoticeable.

## Shipping raw binaries

With Chromium continuing to grow, there were so many exported symbols in Chromium that even the symbols of Content Module and Webkit were more than the limitation. It was impossible to generate a usable shared library by simply stripping symbols.

In the end, we had to [ship the raw binaries of Chromium](https://github.com/electron/libchromiumcontent/pull/98) instead of generating a single shared library.

As introduced earlier there are two build modes in Chromium. As a result of shipping raw binaries, we have to ship two different distributions of binaries in libchromiumcontent. One is called `static_library` build, which includes all static libraries of each module generated by the normal build of Chromium. The other is `shared_library`, which includes all shared libraries of each module generated by the component build.

In Electron, the Debug version is linked with the `shared_library` version of libchromiumcontent, because it is small to download and takes little time when linking the final executable. And the Release version of Electron is linked with the `static_library` version of libchromiumcontent, so the compiler can generate full symbols which are important for debugging, and the linker can do much better optimization since it knows which object files are needed and which are not.

So for normal development, developers only need to build the Debug version, which does not require a good network or powerful machine. Though the Release version then requires much better hardware to build, it can generate better optimized binaries.

## The `gn` update

Being one of the largest projects in the world, most normal systems are not suitable for building Chromium, and the Chromium team develops their own build tools.

Earlier versions of Chromium were using `gyp` as a build system, but it suffers from being slow, and its configuration file becomes hard to understand for complex projects. After years of development, Chromium switched to `gn` as a build system, which is much faster and has a clear architecture.

One of the improvements of `gn` is to introduce `source_set`, which represents a group of object files. In `gyp`, each module was represented by either `static_library` or `shared_library`, and for the normal build of Chromium, each module generated a static library and they were linked together in the final executable. By using `gn`, each module now only generates a bunch of object files, and the final executable just links all the object files together, so the intermediate static library files are no longer generated.

This improvement however made great trouble to libchromiumcontent, because the intermediate static library files were actually needed by libchromiumcontent.

The first try to solve this was to [patch `gn` to generate static library files](https://github.com/electron/libchromiumcontent/pull/239), which solved the problem, but was far from a decent solution.

The second try was made by [@alespergl](https://github.com/alespergl) to [produce custom static libraries from the list of object files](https://github.com/electron/libchromiumcontent/pull/249). It used a trick to first run a dummy build to collect a list of generated object files, and then actually build the static libraries by feeding `gn` with the list. It only made minimal changes to Chromium's source code, and kept Electron's building architecture still.

## 概要

As you can see, compared to building Electron as part of Chromium, building Chromium as a library takes greater efforts and requires continuous maintenance. However the latter removes the requirement of powerful hardware to build Electron, thus enabling a much larger range of developers to build and contribute to Electron. The effort is totally worth it.

