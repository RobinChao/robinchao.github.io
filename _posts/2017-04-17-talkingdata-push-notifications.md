---
layout: post
author: Robin
title: 在iOS上使用TalkingData实现远程推送消息
tags: APNs 开发知识
categories:
  - 开发知识
  - APNs
--- 


`推送通知`是我们的应用程序与我们的用户进行互动的一种重要方式。通过远程推送通知的方式，可以让用户及时的知道一些感兴趣的事情或者重要的事情。也许用户是一个热衷于运动排名的用户，他们希望当排名发生了变化的时候，以通知他们；也许是一个热衷于网购的用户，他们希望及时的获知一些商品促销的活动，等等类似的场景，都非常适合以推送通知的方式来和用户进行交互。

那么如何实现远程推送的功能呢？其中一种方式就是使用[TalkingData](http://www.talkingdata.com/)的推送服务。

在这篇文章中，我将借助[TalkingData](http://www.talkingdata.com/)的推送营销服务，实现一个完整的推送实现流程，其中包括如下7个步骤：

1. 苹果开发者账号配置
2. CSR文件的生成
3. 上传CSR文件
4. 准备推送证书
5. 配置[TalkingData](http://www.talkingdata.com/)推送营销服务
6. 制作[TalkingData](http://www.talkingdata.com/)带有推送服务的App
7. 测试远程推送

我将尽可能的详细解释每一个步骤，希望能够帮到你。好了，让我们开始吧🍻🍻🍻

### 苹果开发者账号配置

第一步是拥有[付费的苹果开发者帐户](https://developer.apple.com/programs/)。您需要注册Apple开发人员计划（每年$ 99）来解锁推送通知功能。

假设您已经拥有付费开发者帐户，请继续[登录到您的Apple开发者帐户](http://developer.apple.com/)。 登录后，您将被重定向到Apple Developer主页。 从那里你应该看到顶部导航栏上的`帐户`。 单击该选项。

现在你应该在你的Apple开发者账户里。

![](/assets/notification-apple-developer.png)

现在看左边菜单栏，第三行应该是“Certificates, IDs & Profiles”，选择该选项。

![](/assets/notification-certificate-option.png)

现在你所在的界面就是“Certificates, Identifiers & Profiles”配置界面。

![](/assets/notification-certificate-profile.png)

看到左侧栏，应该有一个名为“Identifiers”的部分,在该部分下面有一个选项“App IDs”，点击它。

![](/assets/notification-app-id.png)

你可以看到所有的iOS应用ID。

![](/assets/notification-app-id-2.png)

现在在界面的右上角可以看到一个`+`的按钮，点击它，你可以看到如下的界面：

![](/assets/notification-app-id-3.png)

我们需要填写如下的几个选项：

* **App ID Description — Name** 这是可以填写你应用的名称，例如TalkingData Notification Demo

* **App ID Suffix — Explicit App ID — Bundle ID** 在这里，您需要为应用选择唯一的包标识符（例如com.TalkingData.Push）。请确保您使用自己的软件包ID而不是使用我的。

接下来，界面下方，勾选“Push Notifications”，点击继续。

进入确认App ID界面，确认无误后点击下方注册。

现在我们回到我们的“iOS App ID”页面。查找您刚刚创建的应用App ID。点击它，你应该看到一个应用程序服务的下拉菜单。

滑动到界面最下方，点击“Edit”按钮。

![](/assets/notification-push-enabled.png)

"iOS App ID Settings"编辑界面则会显示如下：

![](/assets/notification-push-setting.png)

向下滑动，直到出现“Push Notificaiton”。

现在是我们创建“Client SSL Certificate”的时候了，这将允许我们的通知服务器（TalkingData）连接到Apple Push Notification Service。在开发SSL证书下，点击“Create Certificate…”按钮。

![](/assets/notification-push-ssl.png)

你会看到如下的界面：

![](/assets/notification-push-ssl-2.png)

要生成证书，我们将需要从我们的Mac上构建的证书签名请求（CSR）文件。


### CSR文件的生成

要生成CSR文件，请按"cmd +空格"，并搜索“Keychain Access”。打开钥匙串访问，然后进入菜单，选择“Keychain Access>Certificate Assistant>Request a Certificate From a Certificate Authority…”

![](/assets/notification-keychainaccess.png)

"Certificate Assistant"将会出现一个配置界面：

![](/assets/notification-cert-assistant.png)

填写您的电子邮件地址和名称。选择“Saved to disk”，然后按继续。然后将您的CSR保存在硬盘驱动器上的某个位置。

### 上传CSR文件

现在我们已经生成了CSR，可以回到“Add iOS Certificate”页面。

![](/assets/notification-push-ssl-2.png)

滑动到下方，点击“Continue”，点击“Choose file...”，选择上一步生成的CSR文件：

![](/assets/add-csr-choose-file.png)

点击“Continue”，如果一切正常，界面会显示“Your certificate is ready.”字样：

![](/assets/add-csr-ready.png)

此时，你可以点击下方的“Download”按钮，下载你的证书。

### 准备推送证书

现在您已经创建了iOS证书，然后我们将准备APNs（Apple Push Notifications的简称）证书，稍后将在TalingData配置中使用。打开Finder并找到您之前下载的证书。

![](/assets/locate-cert.png)

双击证书文件（例如：aps_development.cer），证书会自动加入到Mac钥匙串中。

接下来，打开钥匙串，选择“My Certificates”选项，你可以看到你的证书已经添加。它的名字可能是：

	Apple Development IOS Push Services: <your.bundle.id>

右键证书文件，选择“Export ...”：

![](/assets/apns-export-cert.png)

此时会出现一个保存位置配置界面，导出的文件会保存为`.p12`格式的文件，继续点击“Save”：

![](/assets/apns-export-cert-2.png)

接下来为你导出的文件设置密码，然后点击“OK”：

![](/assets/apns-export-cert-3.png)

好了，到此所有的证书准备就已经完成了，接下来让我们进入到TalkingData，继续配置。

### 配置[TalkingData](http://www.talkingdata.com/)推送服务

首先，打开TalkingData首页，点击右上角“登录”，如果没有账号，需要进行“注册”。

![](/assets/talkingdata_main_page.png)

登录之后，TalkingData网页会自动跳转到产品服务界面，这里有多种服务可以选择，我们选择`App Analytics`：

![](/assets/talkingdata_app_analytics.png)

进入后，选择“创建应用”：

![](/assets/talkingdata_app_analytics_create_new_app.png)

填写相关信息：

![](/assets/talkingdata_app_analytics_create_new_app2.png)

确认信息无误后，点击“创建应用”，会出现如下的界面：

![](/assets/talkingdata_app_analytics_create_new_app_done.png)

TalkingData会为每个应用创建一个唯一的应用标识`App ID`，此ID是SDK集成时必须的参数。在界面下方，你可以选择所需要的SDK进行下载，我们这里选择iOS平台下的SDK。

应用创建完成后，我们需要进行TalkingData推送服务的配置。还记的我们之前准备的推送证书吗？这里就是使用它的时候了。在TalkingData网页中点击进入到刚才创建的应用详细界面，可以看到如下的界面：

![](/assets/talkingdata_app_analytics_app_detail.png)

点击顶部“推送营销”选项，进入推送配置界面

![](/assets/talkingdata_app_analytics_apns_conf.png)

可以看到此界面中有“iOS推送配置”区域，此区域可以配置测试和生产的推送证书，我们以测试为例。点击“更新”按钮，选择之前准备的推送证书，输入之前导出证书时你设置的密码，点击“确定”按钮。

![](/assets/talkingdata_app_analytics_apns_conf_done.png)

至此，TalkingData网站上的推送服务配置就完成了，接下来我们制作一个简单的、支持推送服务的Demo。


### 制作[TalkingData](http://www.talkingdata.com/)推送服务App

在Xcode中创建一个新的应用：

![](/assets/talkingdata_app_analytics_xcode.png)

导入下载的TalkingData SDK文件：

![](/assets/talkingdata_app_analytics_xcode_import_lib.png)

然后导入一些所依赖的系统框架：

![](/assets/talkingdata_app_analytics_xcode_import_framwork.png)

完成配置后，在`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(nullable NSDictionary *)launchOptions`方法中编写如下的代码：

``` objc
[TalkingData sessionStarted:@"29066C41F1A64646A47582757E76AB8E" withChannelId:@"Push Demo"];
```

当然，别忘了在引入TalkingData头文件：

``` objc
#import "TalkingData.h"
```

接下来，编写APNs服务标准的代码，由于在不同的系统版本下，注册APNs服务的方式有所不同，因此可能需要根据系统版本区分，这里为了简单，仅编写iOS10以上版本的注册方式，完整代码如下：

``` objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    
    [TalkingData sessionStarted:@"29066C41F1A64646A47582757E76AB8E" withChannelId:@"Push Demo"];
    
    
    //iOS 10
    UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
    [center requestAuthorizationWithOptions:(UNAuthorizationOptionBadge | UNAuthorizationOptionSound | UNAuthorizationOptionAlert) completionHandler:^(BOOL granted, NSError * _Nullable error) {
        if (!error) {
            NSLog(@"request authorization succeeded!");
        }
    }];
    
    return YES;
}

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{
    NSLog(@"DeviceToken: %@", deviceToken);
    [TalkingData setDeviceToken:deviceToken];
}

- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error{
    NSLog(@"Error: %@", [error localizedDescription]);
}
```

注意，`[TalkingData sessionStarted:@"29066C41F1A64646A47582757E76AB8E" withChannelId:@"Push Demo"];`中的第一个参数，就是我们在TalkingData网站上创建用用后得到的那个App ID。

接下来，在Xcode中选择项目的Target > Capabilities，打开"Push Notifications"选项：

![](/assets/talkingdata_app_analytics_xcode_open_push.png)

然后，编译您的应用，在真机上运行Demo。

如果无误，在Xcode的Consol区域中会看到相关的log信息，例如：

![](/assets/talkingdata_app_analytics_xcode_log.png)  

### 测试远程推送

回到TalkingData网页，进入“推送营销”界面，点击界面中的“立即开始”，填写相关的推送活动信息：

![](/assets/talkingdata_app_analytics_push_conf.png)

注意，其中推送通道的选择，一定要谨慎，测试和生产区分开。

确认相关信息无误后，点击“确认，立即提交”按钮，此时手机上就会收到如下的消息： 

![](/assets/talkingdata_app_analytics_push_done.png) 


至此，如何使用TalkingData的推送营销服务，实现iOS平台下的推送基本完成了，其中有些地方是需要细心的，因此在集成的时候，请参考TalkingData官网文档，若有问题，可在TalkingData官网上找到相关的咨询入口，这里不再进行累述。
