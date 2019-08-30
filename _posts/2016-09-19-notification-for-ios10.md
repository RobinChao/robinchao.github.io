---
layout: post
author: Robin
title: Notification for iOS 10 --- 关于iOS 10 推送的那些事
tags: APNs 开发知识 iOS
categories:
  - 开发知识
  - APNs
--- 

iOS 10 新加入了`UserNotifications.framework`用于本地和远程推送的专属框架，可以说是在iOS历史上首次完全重构推送通知的框架结构。
 

#### 基本流程

iOS 10 中的操作流程遵循以下的流程：

![](/assets/notification-flow.png)

1. 需要向用户请求推送权限：以往的方式是直接请求，并没有针对用户的行为进行反馈，而iOS 10 中增加了全新的请求注册方式，会有一个用户是否同意推送的反馈，可以实时的获取。
2. 权限申请成功之后，可以创建本地通知并且发起推送；对于远程推送来说，APNs会反馈`DeviceToken`到客户端，和之前的调用方式相同。
3. 接受推送消息并根据情况展示：之前的消息接收统一在单一的回调接口中，而iOS 10 分离了不同的应用运行状态，如果你的应用在前台正在运行，可以自行决定是否显示这个推送，如果在后台或者没有运行，系统会通过你注册通知时指定的展示方式进行显示。

#### 权限申请

iOS 10 进一步消除了本地推送和远程推送的区别，因为两者在申请权限的时候都是在打断用户的行为。因此iOS 10统一了推送权限的申请：


{% highlight swift  %} 
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) {
            granted, error in
            if granted {
                UIApplication.shared.registerForRemoteNotifications()
            } else {
                if let error = error {
                    UIAlertController.showConfirmAlert(message: error.localizedDescription, in: self)
                }
            }
        }
{% endhighlight %}

当使用`UN`开头的API时，需要引入 `UserNotifications`框架。

{% highlight swift  %} 
import UserNotifications
{% endhighlight %}

当首次调用的时候，会弹出权限申请的弹窗，但是当首次用户没有同意权限，弹窗不会自动再次弹出，如果需要打开权限，需要用户去设置中相应的位置进行开启。

#### 远程推送

当权限申请成功后，直接就可以使用本地推送了，但是对于远程推送来说，还需要申请DeviceToken，服务器需要将此Token连同消息Payload上报给APNs，APNs才可以下发推送到对应的用户。

需要获取DeviceToken的时候，需要调用注册远程推送的API：


{% highlight swift  %} 
// 向 APNs 请求 token：
UIApplication.shared.registerForRemoteNotifications()
{% endhighlight %}

接受DeviceToken需要在ApplicationDelegate回调方法中获取：


{% highlight swift  %} 
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {  
        print("Get Push token: \(deviceToken)")
    }
{% endhighlight %}

iOS 10 还可以实时获取用户当前的推送设置，比如允许横幅显示而不允许弹窗等，都可以直接获取：


{% highlight swift  %} 
UNUserNotificationCenter.current().getNotificationSettings {
    settings in 
    print(settings.authorizationStatus) // .authorized | .denied | .notDetermined
    print(settings.badgeSetting) // .enabled | .disabled | .notSupported
    // etc...
}
{% endhighlight %}

这样在某些场景下，可以对用户的设置进行检查。

#### 消息内容

iOS 10之前的推送内容比较单一，仅仅支持一段文字的消息推送，最好带上badge和sound，以及一些自定义的字段内容，而在iOS 10中，Apple丰富了推送内容，并且增加了更多的显示方式，因此iOS 10的推送也被称为`Rich Notifications`。

iOS 10 之前的基础Payload：


{% highlight json  %} 
{
  "aps":{
    "alert":{
    	"body": "This is a message"
    },
    "sound":"default",
    "badge":1
  }
}
{% endhighlight %}

iOS 10 基础Payload：


{% highlight json  %} 
{
  "aps":{
    "alert":{
      "title":"I am title",
      "subtitle":"I am subtitle",
      "body":"I am body"
    },
    "sound":"default",
    "badge":1
  }
}
{% endhighlight %}

> `title`和`subtitle`并不是必须的。


#### 取消和更新推送内容

iOS 10 中，UserNotifications 框架提供了一系列管理通知的 API，你可以做到：

* 取消还未展示的通知
* 更新还未展示的通知
* 移除已经展示过的通知
* 更新已经展示过的通知

其中关键就在于在创建请求时使用同样的标识符。

对于远程推送来说，目前仅支持推送内容的更新操作，当需要更新某一条之前的推送的时候，只需要在HTTP/2的header中增加`apns-collapse-id`字段，其值为某次推送的标识符。

取消推送的操作目前仅支持本地推送。示例：


{% highlight swift  %} 
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 3, repeats: false)
let identifier = "com.onevcat.usernotification.notificationWillBeRemovedAfterDisplayed"
let request = UNNotificationRequest(identifier: identifier, content: content, trigger: trigger)

UNUserNotificationCenter.current().add(request) { error in
    if error != nil {
        print("Notification request added: \(identifier)")
    }
}

delay(4) {
    print("Notification request removed: \(identifier)")
    UNUserNotificationCenter.current().removeDeliveredNotifications(withIdentifiers: [identifier])
}
{% endhighlight %}


#### 处理推送

推送内容的处理是变化巨大的，比如之前当应用处于前台的时候，收到的推送并不能展示，但是在iOS10中这种情况下的推送展示有了可能，但是需要做一些额外的工作进行支持。

在`UNUserNotificationCenterDelegate`中提供了两个方法，分别对用了如何在应用内展示通知和收到通知后要进行怎样的处理。比如应用在前台运行的时候，收到推送播放声音并进行弹窗：


{% highlight swift  %} 
class NotificationHandler: NSObject, UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, 
                       willPresent notification: UNNotification, 
                       withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) 
    {
        completionHandler([.alert, .sound])

        // 如果不想显示某个通知，可以直接用空 options 调用 completionHandler:
        // completionHandler([])
    }
}
{% endhighlight %}

#### 对通知的响应

在`UNUserNotificationCenterDelegate`中还有一个方法`userNotificationCenter(_:didReceive:withCompletionHandler:)`专门用来和通知进行响应交互的操作，包括用户通过点击推送打开应用，或者点击或者触发推送所支持的某个action等等。

其中最简单的方式就是，所有的交互直接交给系统去处理：


{% highlight swift  %} 
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    completionHandler()
}
{% endhighlight %}

但是在某些情况下，需要根据不同的情况进行自己处理，就需要在推送的Payload中携带一些额外的数据，以便支持：


{% highlight swift  %} 
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    if let name = response.notification.request.content.userInfo["name"] as? String {
        print("I know it's you! \(name)")
    }
    completionHandler()
}
{% endhighlight %}

### Notification Extension

iOS 10 中添加了很多 extension，作为应用与系统整合的入口。与通知相关的 extension 有两个：Service Extension 和 Content Extension。前者可以让我们有机会在收到远程推送的通知后，展示之前对通知内容进行修改；后者可以用来自定义通知视图的样式。

![](/assets/notification-extensions.png)

#### 截取并修改通知内容

`NotificationService` 的模板已经为我们进行了基本的实现：


{% highlight swift  %} 
class NotificationService: UNNotificationServiceExtension {

    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?

    // 1
    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)

        if let bestAttemptContent = bestAttemptContent {
            if request.identifier == "mutableContent" {
                bestAttemptContent.body = "\(bestAttemptContent.body), onevcat"
            }
            contentHandler(bestAttemptContent)
        }
    }

    // 2
    override func serviceExtensionTimeWillExpire() {
        // Called just before the extension will be terminated by the system.
        // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }
}
{% endhighlight %}

1. didReceive: 方法中有一个等待发送的通知请求，我们通过修改这个请求中的 content 内容，然后在限制的时间内将修改后的内容调用通过 contentHandler 返还给系统，就可以显示这个修改过的通知了。
2. 在一定时间内没有调用 contentHandler 的话，系统会调用这个方法，来告诉你大限已到。你可以选择什么都不做，这样的话系统将当作什么都没发生，简单地显示原来的通知。可能你其实已经设置好了绝大部分内容，只是有很少一部分没有完成，这时你也可以像例子中这样调用 contentHandler 来显示一个变更“中途”的通知。

Service Extension 现在只对远程推送的通知起效，你可以在推送 payload 中增加一个 mutable-content 值为 1 的项来启用内容修改：


{% highlight json  %} 
{
  "aps":{
    "alert":{
      "title":"Greetings",
      "body":"Long time no see"
    },
    "mutable-content":1
  }
}
{% endhighlight %}

> 使用在本机截取推送并替换内容的方式，可以完成端到端 (end-to-end) 的推送加密。你在服务器推送 payload 中加入加密过的文本，在客户端接到通知后使用预先定义或者获取过的密钥进行解密，然后立即显示。这样一来，即使推送信道被第三方截取，其中所传递的内容也还是安全的。使用这种方式来发送密码或者敏感信息，对于一些金融业务应用和聊天应用来说，应该是必备的特性。


### 在通知中展示图片/视频/音频

iOS 10 最让人兴奋的地方就是在这里，能够直接在推送的通知内容里面传递多媒体资源，在客户端进行直接的展示，极大丰富了推送内容的可读性和趣味性。

通过远程推送的方式，你也可以显示图片等多媒体内容。这要借助于上一节所提到的通过 Notification Service Extension 来修改推送通知内容的技术。一般做法是，我们在推送的 payload 中指定需要加载的图片资源地址，这个地址可以是应用 bundle 内已经存在的资源，也可以是网络的资源。不过因为在创建 UNNotificationAttachment 时我们只能使用本地资源，所以如果多媒体还不在本地的话，我们需要先将其下载到本地。在完成 UNNotificationAttachment 创建后，我们就可以和本地通知一样，将它设置给 attachments 属性，然后调用 contentHandler 了。

简单的示例 payload 如下：

{% highlight json  %} 
{
  "aps":{
    "alert":{
      "title":"Image Notification",
      "body":"Show me an image from web!"
    },
    "mutable-content":1
  },
  "image": "https://pixabay.com/static/uploads/photo/2016/09/15/21/11/hover-fly-1672677_960_720.jpg"
}
{% endhighlight %}

`mutable-content` 表示我们会在接收到通知时对内容进行更改，`image` 指明了目标图片的地址。(图片的地址最好是https的，测试发现非https的不能显示。)

在 `NotificationService` 里，需要下载媒体资源，并缓存在本地磁盘中，然后创建`UNNotificationAttachment`进行本地资源路径指定，返回给系统即可。

关于在通知中展示图片或者视频，有几点想补充说明：

* `UNNotificationContent` 的 `attachments` 虽然是一个数组，但是系统只会展示第一个 attachment 对象的内容。不过你依然可以发送多个 attachments，然后在要展示的时候再重新安排它们的顺序，以显示最符合情景的图片或者视频。另外，你也可能会在自定义通知展示 UI 时用到多个 attachment。我们接下来一节中会看到一个相关的例子。
* 在当前 iOS 10 中，`serviceExtensionTimeWillExpire` 被调用之前，你有 30 秒时间来处理和更改通知内容。对于一般的图片来说，这个时间是足够的。但是如果你推送的是体积较大的视频内容，用户又恰巧处在糟糕的网络环境的话，很有可能无法及时下载完成。
* 如果你想在远程推送来的通知中显示应用 bundle 内的资源的话，要注意 extension 的 bundle 和 app main bundle 并不是一回事儿。你可以选择将图片资源放到 extension bundle 中，也可以选择放在 main bundle 里。总之，你需要保证能够获取到正确的，并且你具有读取权限的 url。关于从 extension 中访问 main bundle，可以参看这篇回答。
* 系统在创建 `attachement` 时会根据提供的 `url` 后缀确定文件类型，如果没有后缀，或者后缀无法不正确的话，你可以在创建时通过 `UNNotificationAttachmentOptionsTypeHintKey` 来指定资源类型。
* 如果使用的图片和视频文件不在你的 bundle 内部，它们将被移动到系统的负责通知的文件夹下，然后在当通知被移除后删除。如果媒体文件在 bundle 内部，它们将被复制到通知文件夹下。每个应用能使用的媒体文件的文件大小总和是有限制，超过限制后创建 attachment 时将抛出异常。可能的所有错误可以在 UNError 中找到。
* 你可以访问一个已经创建的 `attachment` 的内容，但是要注意权限问题。可以使用 `startAccessingSecurityScopedResource` 来暂时获取以创建的 `attachment` 的访问权限。比如：


{% highlight swift  %} 
let content = notification.request.content
if let attachment = content.attachments.first {  
    if attachment.url.startAccessingSecurityScopedResource() {  
        eventImage.image = UIImage(contentsOfFile: attachment.url.path!)  
        attachment.url.stopAccessingSecurityScopedResource()  
    }  
}  
{% endhighlight %}

另外对于媒体资源来说，大小也是有限制的：

![](/assets/media.png)

### iOS 10 中被标为弃用的 API

* UILocalNotification
* UIMutableUserNotificationAction
* UIMutableUserNotificationCategory
* UIUserNotificationAction
* UIUserNotificationCategory
* UIUserNotificationSettings
* handleActionWithIdentifier:forLocalNotification:
* handleActionWithIdentifier:forRemoteNotification:
* didReceiveLocalNotification:withCompletion:
* didReceiveRemoteNotification:withCompletion: 