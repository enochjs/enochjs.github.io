---
title: react-native-vector-icons 安装使用
date: 2022-02-16 10:43:15
categories:
  - react-native
tags:
---

### 介绍

1. react-native 字体图标库
2. 支持字体图标库
   - AntDesign
   - Entypo
   - EvilIcons
   - Feather
   - FontAwesome
   - FontAwesome 5
   - Fontisto
   - Foundation
   - Ionicons
   - MaterialIcons
   - MaterialCommunityIcons
   - Octicons
   - Zocial
   - SimpleLineIcons

### 如何安装

1. yarn add react-native-vector-icons (yarn add @types/react-native-vector-icons -D)
2. 执行 ios pod install

   - cd ios
   - pod install

3. 进到 node_modules/react-native-vector-icons，将需要使用的字体文件 copy 到项目中
   - ![](fonts.jpg)
4. 修改 info.plist, 右键打开，open as => source code, 复制以下代码

   ```javascript
   	<key>UIAppFonts</key>
   	<array>
   		<string>Fonts/AntDesign.ttf</string>
   		<string>Fonts/Entypo.ttf</string>
   		<string>Fonts/EvilIcons.ttf</string>
   		<string>Fonts/Feather.ttf</string>
   		<string>Fonts/FontAwesome.ttf</string>
   		<string>Fonts/FontAwesome5_Brands.ttf</string>
   		<string>Fonts/FontAwesome5_Regular.ttf</string>
   		<string>Fonts/FontAwesome5_Solid.ttf</string>
   		<string>Fonts/Foundation.ttf</string>
   		<string>Fonts/Ionicons.ttf</string>
   		<string>Fonts/MaterialIcons.ttf</string>
   		<string>Fonts/MaterialCommunityIcons.ttf</string>
   		<string>Fonts/SimpleLineIcons.ttf</string>
   		<string>Fonts/Octicons.ttf</string>
   		<string>Fonts/Zocial.ttf</string>
   		<string>Fonts/Fontisto.ttf</string>
   	</array>
   ```

5. 关联字体文件

- ![](build_phase.jpg)

6. 删除 ios/build, 重新构建使用

### 参考文档

- [react-native-vector-icons github](https://github.com/oblador/react-native-vector-icons)
