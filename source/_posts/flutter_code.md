---
title: flutter 生成二维码保存到相册
date: 2020-04-12 14:53:27
tags: 
  - flutter
categories: flutter
top: true
---



## 1.生成二维码

- 引入`qr_flutter`: ^3.2.0 插件(最新版本号请看pub.dev)

  [https://pub.flutter-io.cn/packages/qr_flutter]: 

```dart
dependencies:
  qr_flutter: ^3.2.0
```

-  在页面中引入模块（更多配置参数查询插件）

```dart
import 'package:flutter/rendering.dart';
```

```dart
QrImage(
  data: "1234567890", // 生成二维码内容
  version: QrVersions.auto,
  size: 200.0,
),
```

![](https://cdn.jsdelivr.net/gh/unclemin/images/Img/dartcode.png)

![](https://cdn.jsdelivr.net/gh/unclemin/images/Img/微信截图_20200422102231.png)

- 最终生成效果， 按钮是自己添加



## 2.保存二维码到相册

- 这里需要安装两插件
- 插件地址(更多配置可参考)
  - https://pub.flutter-io.cn/packages/image_gallery_saver
  - https://pub.flutter-io.cn/packages/permission_handler

```dart
image_gallery_saver: ^1.2.2 #保存图片到相册
permission_handler: ^5.0.0+hotfix.3 #获取手机权限
```

#### 	1.获取二维码截图

- 这里是生成的二维码，并不是一张图片，所以需要先截取二维码部分的`Widget`
  - 引入所需模块
  - 初始化`GlobalKey`
  - 使用`RepaintBoundary`包裹我们要截图的组件
  - 这里有个坑，生成的二维码是黑色的方块加白色的底，如果没有`Container`包裹住`QrImage`的话，保存的图片是全黑色的，因此`Container`的`color`一定要给个颜色

```dart
import 'package:flutter/rendering.dart';
import 'dart:ui' as ui;

GlobalKey repaintKey = GlobalKey();

RepaintBoundary(
  key: repaintKey,
  child: Container(
    height: 200,
    width: 200,
    color: Colors.white,
    child: QrImage(
      data: "1234567890",
      version: QrVersions.auto,
      size: 200.0,
    ),
  ),
),
```

![](https://cdn.jsdelivr.net/gh/unclemin/images/Img/123.png)

#### 2.保存到相册

- 引入保存图片和检测权限的插件

```dart
import 'package:image_gallery_saver/image_gallery_saver.dart';
import 'package:permission_handler/permission_handler.dart';
```

- 点击保存方法调用

```dart
FlatButton(
  onPressed: () async {
    await getPerm();
  },
  child: Text(
    '保存二维码到相册',
    style: TextStyle(color: Color.fromRGBO(15, 110, 255, 1)),
  ),
  color: Color.fromRGBO(240, 246, 255, 1),
)
```

- 检测是否有保存权限函数（具体可看插件）,注意运行报错的话，需要 `flutter clean` 然后 `flutter run` 重新运行一下

```dart
Future getPerm() async {
    Map<Permission, PermissionStatus> statuses = await [
      Permission.storage,
    ].request();
    var status = await Permission.storage.status;
    print(status);
    if (status.isUndetermined) {
      openAppSettings();// 没有权限打开设置页面
    } else {
      capturePng();// 已有权限开始保存
    }
}
```

- 保存函数

```dart
  Future<String> capturePng() async {
    try {
      print('开始保存');
      RenderRepaintBoundary boundary =
          repaintKey.currentContext.findRenderObject();
      ui.Image image = await boundary.toImage();
      ByteData byteData =
          await image.toByteData(format: ui.ImageByteFormat.png);
      final result =
          await ImageGallerySaver.saveImage(byteData.buffer.asUint8List());
      print(result);// result是图片地址
      BotToast.showSimpleNotification(title: '保存成功');
    } catch (e) {
      print(e);
    }
    return null;
  } 
```

到此生成二维码并保存到相册就ok了 ,主要就是注意二维码背景色的问题，还有就是打开手机保存权限，注意各个版本插件使用方法按照官方的来，谷歌出来的很多版本太低了，使用方法也不一样,注意检查权限运行报错的话，需要 `flutter clean` 然后 `flutter run` 重新运行一下



