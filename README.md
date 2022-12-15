# mtpush-cocos2d-x-plugin

EngagLab MTPush's officially supported Cocos2d-x plugin (Android &amp; iOS). EngageLab推送官方支持的 Cocos2d-x 插件（Android &amp; iOS）。


## iOS Project 集成 MTPush 插件

#### 1. 配置基本信息

* 使用 Cocos2d-x 生成 iOS 工程

* 添加必要框架。打开 Xcode，点击 project，选择 (Targets -> Build Phases -> Link Binary With Libraries)，添加以下框架：

		CFNetwork.framework
		CoreFoundation.framework
		CoreTelephony.framework
		CoreGraphics.framework
		Foundation.framework
		UIKit.framework
		Security.framework
		SystemConfiguration.framework
		libz.tbd//Xcode7 以下是 libz.dylib
		libresolv.tbd（Xcode 7 以下版本是 libresolv.dylib）
		libsqlite3.tbd
		UserNotifications.framework（Xcode 8 及以上）
		
* Capabilities
  如使用 Xcode 8 及以上环境开发，请开启 Application Target 的 Capabilities->Push Notifications 选项	
  如使用 Xcode 10 及以上环境开发，请开启 Application Target 的 Capabilities-> Access WIFI Infomation 选项。 
  	
		
* 将插件的 /iOS/MTPushPlugin 文件夹及内容拖拽到 Xcode 工程里，拖拽时 Choose options for adding these files 选择：
	-  Destination：✓ Copy items if needed
	-  Added folders：✓ Create groups
	-  Add to targets：✓ your-proj-name
  
#### 2. 添加代码

* 	在工程的 /ios/AppController.mm (注意不是 AppDelegate.cpp) 中添加头文件：

	```
		// 引入 MTPush 功能所需头文件
#import "MTPushService.h"
// iOS10 注册 APNs 所需头文件
#ifdef NSFoundationVersionNumber_iOS_9_x_Max
#import <UserNotifications/UserNotifications.h>
#endif
// 如果需要使用 idfa 功能所需要引入的头文件（可选）
#import <AdSupport/AdSupport.h>
	```


* 在 AppController.mm 中添加以下代码：（如果方法存在，则只将其中代码添加至方法中；如果方法不存在，则添加方法及其中代码）

	```
	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions{
		
		 // 3.0.0及以后版本注册可以这样写，也可以继续用旧的注册方式
    MTPushRegisterEntity * mtp_entity = [[MTPushRegisterEntity alloc] init];
    if (@available(iOS 12.0, *)) {
      mtp_entity.types = MTPushAuthorizationOptionAlert|MTPushAuthorizationOptionBadge|MTPushAuthorizationOptionSound|MTPushAuthorizationOptionProvidesAppNotificationSettings;
      if (@available(iOS 13.0, *)) {
        mtp_entity.types = mtp_entity.types | MTPushAuthorizationOptionAnnouncement;
      }
    } else {
      mtp_entity.types = MTPushAuthorizationOptionAlert|MTPushAuthorizationOptionBadge|MTPushAuthorizationOptionSound;
    }
    if ([[UIDevice currentDevice].systemVersion floatValue] >= 8.0) {
      // 可以添加自定义categories
      // NSSet<UNNotificationCategory *> *categories for iOS10 or later
      // NSSet<UIUserNotificationCategory *> *categories for iOS8 and iOS9
    }
    if ([[UIDevice currentDevice].systemVersion floatValue] >= 12.0) {
    //      UNNotificationCategory *sumFormatCategory =  [UNNotificationCategory categoryWithIdentifier:@"iOS12_categorySummaryFormat" actions:@[] intentIdentifiers:@[] hiddenPreviewsBodyPlaceholder:nil categorySummaryFormat:@"%u 更多消息啦啦" options:UNNotificationCategoryOptionCustomDismissAction | UNNotificationCategoryOptionAllowInCarPlay | UNNotificationCategoryOptionAllowAnnouncement];
    //    entity.categories = [NSSet setWithObject:sumFormatCategory];
      }
    [MTPushService registerForRemoteNotificationConfig:mtp_entity delegate:self];
    
    [MTPushService setupWithOption:launchOptions appKey:@"您的APPKEY" channel:@"test" apsForProduction:FALSE];
    [MTPushService setDebugMode];
    
    //2.1.9版本新增获取registration id block接口。
    [MTPushService registrationIDCompletionHandler:^(int resCode, NSString *registrationID) {
      if(resCode == 0){
        NSLog(@"registrationID获取成功：%@",registrationID);
        
      }
      else{
        NSLog(@"registrationID获取失败，code：%d",resCode);
      }
    }];
		


    	......
	    return YES;
	}
	```
	```
	- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{
		// Required
		[MTPushService registerDeviceToken:deviceToken];
	}
	```
	```	
	- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
	  	// Required
		[MTPushService handleRemoteNotification:userInfo];
	}
	```
	```
	- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
  		[MTPushService handleRemoteNotification:userInfo];
	  	completionHandler(UIBackgroundFetchResultNewData);
	}
	```
	
```
	#ifdef __IPHONE_10_0
#pragma mark- MTPushRegisterDelegate
- (void)mtpNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(NSInteger))completionHandler {
  NSDictionary * userInfo = notification.request.content.userInfo;
  if([notification.request.trigger isKindOfClass:[UNPushNotificationTrigger class]]) {
    [MTPushService handleRemoteNotification:userInfo];
  }
  UIAlertView *test = [[UIAlertView alloc] initWithTitle:@"mtp willPresent"
                                                 message:@"tt"
                                                delegate:self
                                       cancelButtonTitle:@"yes"
                                       otherButtonTitles:nil, nil];
  [test show];
  completionHandler(UNNotificationPresentationOptionBadge|UNNotificationPresentationOptionSound|UNNotificationPresentationOptionAlert);
}

- (void)mtpNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)())completionHandler {
  NSDictionary * userInfo = response.notification.request.content.userInfo;
  if([response.notification.request.trigger isKindOfClass:[UNPushNotificationTrigger class]]) {
    [MTPushService handleRemoteNotification:userInfo];
  }
  UIAlertView *test = [[UIAlertView alloc] initWithTitle:@"didReceive"
                                                 message:@"tt"
                                                delegate:self
                                       cancelButtonTitle:@"yes"
                                       otherButtonTitles:nil, nil];
  [test show];
  completionHandler();
}

- (void)mtpNotificationAuthorization:(MTPushAuthorizationStatus)status withInfo:(NSDictionary *)info {
  NSLog(@"receive notification authorization status:%lu, info:%@", status, info);
}

#endif

#ifdef __IPHONE_12_0
- (void)mtpNotificationCenter:(UNUserNotificationCenter *)center openSettingsForNotification:(UNNotification *)notification{
  NSString *title = nil;
  if (notification) {
    title = @"从通知界面直接进入应用";
  }else{
    title = @"从通知设置界面进入应用";
  }
  UIAlertView *test = [[UIAlertView alloc] initWithTitle:title
                                                 message:@"pushSetting"
                                                delegate:self
                                       cancelButtonTitle:@"yes"
                                       otherButtonTitles:nil, nil];
  [test show];
  
}
#endif


```
	
* 在需要处理推送回调的类中添加回调函数，相应地调用 MTPush SDK 提供的 API 来实现功能,调用地方需要引入头文件 MTPushBridge.h

		#import "MTPushBridge.h"

		MTPushBridge::registerCallbackFunction(setupCallback, closeCallback,
                                         Register_callback, Login_callback,
                                         ReceiveMessage_callback);
                                         
                                         
* 或者你也可以分别调用每一个回调函数的设置 API 方法

		static void registerSetupCallbackFunction(setupCallback);
		static void registerCloseCallbackFunction(closeCallback);
		static void registerRegisterCallbackFunction(Register_callback);
		static void registerLoginCallbackFunction(Login_callback)
		static void registerCallbackFunction(ReceiveMessage_callback);

* API 参数要符合头文件提供的函数指针

		void setupCallback() { cout << "setup" << endl; }

* 获取 RegistrationID

		void register_callback(const char *registrationID)；
		


