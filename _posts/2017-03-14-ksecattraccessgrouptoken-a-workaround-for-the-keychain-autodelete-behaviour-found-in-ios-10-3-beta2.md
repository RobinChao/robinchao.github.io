---
layout: post
title: kSecAttrAccessGroupToken --- 应对iOS 10.3将自动删除keychain行为的解决方案
date: 2017-03-14 13:31:15 +0800
categories: 开发知识
tags: iOS 开发 移动开发 keychain
---


2017年2月7日，Apple发布了其iOS 10.3 Beta2的版本，随之而来的是keychain的一些变化。

keychain伴随着iOS 7系统的设备发布，其目的是让用户能够安全的存储各种登录凭证的服务，以解决每次都需要输入的问题。Safari使用keychain，并提示用户在登录相关网站的时候保存网站的登录凭证，之后再访问时即可免去再次键入的麻烦，当然开发者也可以使用keychain，开发者可以在应用中午任何用户交互的情况下保存密码、证书、密钥等等。

一直以来keychain都有一个意想不到的副作用，就是在应用删除的情况下，存储在keychain中的数据也不会删除，一直保存在其中。但是这也带来一定的便利性，当下次再安装某个曾经安装过的应用时，该应用就可以直接读取曾经存储在keychain中的相关数据，并使得应用程序继续，就好像什么都没有发生过一样，对于用户来说也是一种无缝衔接。

keychain的这个属性对于开发者非常有用，很多实用开发者非常依赖keychain的这个属性，但是这个依赖将要被改变。在[苹果的开发者论坛](https://forums.developer.apple.com/message/210559)中，苹果的开发人员已经透露，在iOS 10.3中，keychain将会在对应的应用卸载后，自动删除存储的数据，对于应用组来说，当应用组中所有的应用卸载后，keychain也会自动删除keychain中的相关数据。

这对于那些严重依赖keychain的应用来说非常致命，可能导致应用无法正常运行等等问题，所以这里将寻找一个对应的解决方案。


偶然间，遇到了*kSecAttrAccessGroupToken*这个常量，在网路上也没有很多相关的信息，但是在iOS SDK代码注释里，发现了一些有用的信息：

``` objc
/*!
     @enum kSecAttrAccessGroup Value Constants
     @constant kSecAttrAccessGroupToken Represents well-known access group
         which contains items provided by external token (typically smart card).
         This may be used as a value for kSecAttrAccessGroup attribute. Every
         application has access to this access group so it is not needed to
         explicitly list it in keychain-access-groups entitlement, but application
         must explicitly state this access group in keychain queries in order to
         be able to access items from external tokens.
*/
extern const CFStringRef kSecAttrAccessGroupToken
    __OSX_AVAILABLE_STARTING(__MAC_10_12, __IPHONE_10_0);
```

看起来似乎是一个访问组，在iOS 10中引入，手机上的所有应用程序都可以读/写访问。如果是这种情况，那么存储在keychain中的数据将永远保留，因为当前安装的应用程序可能随时需要访问它。

在进行了相关的测试之后，使用*kSecAttrAccessGroupToken*访问组保存的keychain数据已经超出了应用程序卸载的访问，只要不主动的删除，其中保存的数据似乎永远存在。所以为了保存现有相关的密钥等数据，一个简单的方法就是使用*kSecAttrAccessGroupToken*保存数据。但是，不得不说，此方法有一个噪点：

由于系统上所有的应用程序都可以自由的读出和写入此访问组，因此需要考虑存入数据的安全性。如果过于敏感的数据，可能需要进行相关加密处理后，再进行存储。

PS：相关代码这里就不再给出了，如果需要使用*kSecAttrAccessGroupToken*常量属性，可以查阅SDK注释代码说明。