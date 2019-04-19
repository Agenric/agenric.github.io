---
title: 自定义统一分享平台
date: 2017-04-02 20:24:45
categories:
 - 技术
tags:
 - iOS小知识
---

鉴于社交网络现在的强大（淫威），现在市面上的APP，分享到社交网络的这个功能可以说是标配。期初我司的APP在添加分享这一块的功能时，leader说咱们开发人员少、工期紧（敏捷开发嘛，大家都懂...），这块就用网上现成的吧，什么友盟啊之类的就行。

暂且不说这期间的种种...

因为我们的需求仅仅是分享一个带有url的string。我就在想，有没有必要因为一个这么小的需求去使用一个体量很大的第三方。宽且是对于我这样一个有略微强迫症的人来说，这太影响代码美观了。哈哈，其实可能并没有...

然而，最终还是没有用，所以我才把这个玩意儿整理出来，感觉拖了n年。

整个AGShareDesk提供三个方法：

## 注册

`- (void)registerWithWeiboAppKey:(NSString *)weiboAppKey weChatAppKey:(NSString *)weChatAppKey tencentAppId:(NSString *)tencentAppId;`

## 分享

`- (void)shareToChannel:(ShareChannel)shareChannel withMessgaeObject:(ShareMessageObject *)message afterDelegate:(id<AGShareDeskDelegate>)afterDelegate;`

## 处理分析结果

`- (void)handleApplication:(UIApplication *)application withOpenURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options;`

然后会有一个`ShareMessageObject`，在你分享之前，你需要创建一个 ShareMessageObject 的对象，该对象可以设置分享的标题、内容、链接以及logo的图片，ShareMessageObject 仅提供一个类方法返回一个该对象的实例。

整体来说就是把之前散落在个个地方的代码整合起来，你只需在 AppDelegate.m 中的 didFinishLaunch 中注册你在各个平台申请的key，然后在 openURL 中告诉 ShareDesk 去处理分享之后的回调即可。其余的所有工作都在 ShareDesk 内部完成。

你可以使用 `pod "AGShareDesk"`来使用它。如果不能满足你的需求也希望能给你一个参考，完整代码在[这里](https://github.com/Agenric/AGShareDesk)

### PS

上文已经谈过，业务需求很简单，所以并没有做很丰富的分享，如果你有其他类型的需求，那么希望这个项目能给你一个参考，如果真的没什么可参考的，那，那就算了...反正能说的我都说了。哈哈