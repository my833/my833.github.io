+++
date = '2026-03-27T17:50:42+08:00'
draft = false
title = '获取APP使用的Flutter版本'
tags = [
  'android', 'flutter'
]
+++
**发现部分应用获取不到，可能有点问题**

查看手机中哪些app使用了flutter，可以使用`FlutterShark`或`FlutterEye`。

若要自己实现，参考[获取 APP 所使用的 Flutter 版本](https://panda912.com/2023/09/18/%E8%8E%B7%E5%8F%96-APP-%E6%89%80%E4%BD%BF%E7%94%A8%E7%9A%84-Flutter-%E7%89%88%E6%9C%AC/)。

可以解析`libapp.so`获取`snapshot_hash`，再根据`snapshot_hash`得到对应的 Flutter 和 Dart 版本。目前看会有多个 `Flutter` `Dart` 版本对应同一个`snapshot_hash`。

## 解析`libapp.so`获取`snapshot_hash`，在查找对应版本
在`libapp.so`中查找`snapshot`魔数`0xdcdcf5f5`, 4字节，注意字节序（`0xf5 0f5 0xdc 0xdc`），其后紧跟8字节的`length`，再其后8字节的kind，再其后32字节即为`snapshot_hash`。

若只需`Flutter`版本，可直接查询[Snapshot hash 与 Flutter 对应关系](https://raw.github.com/mildsunrise/darter/refs/heads/master/info/versions.md)（表一）。`snapshot_hash`即为表中的`Snapshot version`。

若再需要`Dart`版本，再参考[Flutter Release](https://storage.googleapis.com/flutter_infra_release/releases/releases_windows.json)（表二）。 从`libapp.so`中读取到`snapshot_hash`后，在表一中查找对应的`commit`，`commit`在表二中为`hash`，即可找到对应的`dart`版本`dart_sdk_version`。

## 手动生成`Flutter``Dart`版本与`snapshot_hash`对应关系表
需要`clone`三个库 [flutter](https://github.com/flutter/flutter.git), [engine](https://github.com/flutter/engine.git), [dart](https://github.com/dart-lang/sdk.git) ，其中`engine`库再后续版本中已并入`flutter`，在生成之前的版本对应关系时才需要。

可以从表二中获取所有`flutter`版本tag，然后进入`flutter`库，切换到对应tag：`git checkout $tag`，在对应`tag`库中查找对应的`dart`库。
- 如果`flutter`中有`/DEPS`，则可通过`cat DEPS | grep dart_revision | head -n 1 | awk -F"'" '{print $4}'`得到`dart`（`sdk`）对应的revision；
- 若没有`/DEPS`，则通过`engine`库查找，需先把`engine`库切换到`flutter`库中文件`/bin/internal/engine.version`指定的revision，再在`engine`的`/DEPS`中得到`dart`（`sdk`）对应的revision。

得到`dart`（`sdk`）的revision后，进入`sdk`库，切换到对应revision，进入`/tools`目录，执行`python3 make_version.py --format {{SNAPSHOT_HASH}}`，得到`snapshot_hash`，即为`libapp.so`中读到的值。在一些旧版本中，`make_version.py`的执行会不一样，需要先`echo {{SNAPSHOT_HASH}} > inp`，然后`python3 make_version.py --input=inp --output=oup`，输出文件`oup`中即为`snapshot_hash`；在更就一些的版本，需要用`python2`。

小结：
1. 根据`flutter revision`获取到`dart(sdk) revision`，中间可能需要通过`engine revision`；
2. 根据`dart(sdk) revision`获取到`snapshot hash`；即可得到`snapshot hash`与`flutter`的对应关系表；
3. 从`libapp.so`读取到`snapshot hash`，在上一步得到的表中查找得到`flutter`版本号


查找是否包含`libapp.so`时，不仅要查找通过`packageManager.getIntalledPackages`或`packageMangaer.getInstallApplications`得到的List中的apk，还要查找此apk同路径下的所有其他apk，因为Android有`Split APK`机制，可能会把其他资源文件分开成多个apk。

