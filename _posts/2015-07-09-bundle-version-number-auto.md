---
layout: post
author: Robin
title: Xcode Bundle Version 自动更新
tags: 开发知识 iOS
categories:
  - 开发知识
--- 

在码代码的世界里，一切都是自动的。但是在管理Xcode CFBundleVersion 的时候，却深深有了挫败感，到底如何设置合适？如果统一呢？并且如果单个工程中如果有Watch kit app和extension的话，Xcode页必须保证三个info.plist中的CFBundleVersion一致才可以，无形中增加了工作量。

一段摸索后，终于发现了一个小技巧，短短的几行脚本就可以轻松实现自动管理CFBundleVersion。

设置步骤：
 
 * 打开Xcode项目工程，在工具条左上角target device左边，点击工程名称，在弹出的选项中选择 Edit Scheme…
 * 打开编辑面板后，在左边列表中，找到 Build 选项，并展开它。
 * 在展开的选项中，选择Pre-actions
 * 如果Pre-actions中没有脚本代码，视图中央会有一个New Run Script Action 按钮，点击此按钮。
 * 设置Provide build settings 为你当前的项目
 * 复制下面的脚本代码，到脚本编辑框中：

 ```
formatDate=$(date "+%Y%m%d%H%M%S")
buildNumber="${formatDate}"
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${buildNumber}" "${PROJECT_DIR}/${INFOPLIST_FILE}"
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${buildNumber}" "${SRCROOT}/<YOUR WATCHKIT APP FOLDER>/Info.plist"
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${buildNumber}" "${SRCROOT}/<YOUR WATCHKIT EXTENSION FOLDER>/Info.plist"
 ```
 
 That’s all。 还有一些其他的相关设置，能够进一步提升这段代码的功效，请自行研究哈~~