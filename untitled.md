# iOS13 Apple Push Notification Service \(APNS\) 部署指南

## 证书配置

在iOS的profile配置文件中，包含有App Identifier，指定的签名证书，以及需要申请的一些系统权限，其中就包括推送通知权限，所以部署APNS需要从证书的生成开始，登录苹果开发者后台[https://developer.apple.com/account](https://developer.apple.com/account) 生成。

## 生成Certificate Signing Request（CSR）文件

打开Keychain App，选择 _证书助理-&gt;选择从证书颁发机构请求证书-&gt;保存到磁盘_

### 证书申请

1. 点击添加Certificates，选择类别iOS App Developement，选择刚刚的csr文件请求开发者身份证书，并下载到本地备用
2. 点击添加Certificates，选择类别iOS Distribution，选择刚刚的csr文件请求ad hoc和App Store环境用的身份证书，并下载到本地备用
3. 点击添加identifier, 选择App IDs，创建app的bundle identifier
4. 点击添加Certificates，选择类别Apple Push Notification service SSL \(Sandbox & Production\)，生成推送证书，当然也可以分别选择两种环境进行生成
5. 点击添加Devices，把development环境下需要测试设备的udid添加进去
6. 点击添加profile，分别选择developement和app store环境，选择以上生成的证书进行生成，分别生成两种环境下的profile文件

## 项目配置

### 证书设置

1. 把上面证书申请的所有证书双击添加到钥匙串中
2. 在项目的签名**signing & Capabilities**中，取消automatically manage signing, 在下面手动选择对应环境的profile配置

### 权限声明

选中工程中的**signing & Capabilities**, 选择 **+Capability**，在列表中选择_Push Notification_, _Background Modes_添加到列表中，在_Background Modes_勾选_Remote Notification_

### 环境选择

在上面选择了_Push Notification_之后，系统会自动生成一个entitlement文件在工程的根目录下，同时，在**build setting**的_Code Signing Entitlement_中也会自动指向这个文件的路径。在这个文件中，有一个_APS Environment_的值，可以填写development和production，分别对应推送通知的sandbox和production环境。

### framework

在iOS10以上的版本，需要导入UserNotification.framework框架

## 代码实现

此处代码兼容iOS10以下的版本

### framework导入与版本判断

```text
#import <UserNotifications/UserNotifications.h>

#define SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(v)  ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedAscending)
#define SYSTEM_VERSION_LESS_THAN(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedAscending)
```

### 遵循协议

UIApplicationDelegate与UNUserNotificationCenterDelegate

### 清零角标

推送通知的角标需要手动调用 `[application setApplicationIconBadgeNumber:0];` 方法进行清0

### 注册推送通知

在`application: didFinishLaunchingWithOptions:`方法中，调用以下方法进行推送通知的注册，同时需要遵循UIApplicationDelegate协议。在iOS10以上版本需要遵循UNUserNotificationCenterDelegate协议来获取用户处理通知的方式

以下分别采用不同方法注册通知

```text
- (void)registerForRemoteNotifications {
    if (@available(iOS 10, *)) {
        UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
        center.delegate = self;
        [center requestAuthorizationWithOptions:(UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionBadge) completionHandler:^(BOOL granted, NSError * _Nullable error){
            if (!error){
                dispatch_async(dispatch_get_main_queue(), ^{
                    [[UIApplication sharedApplication] registerForRemoteNotifications];
                    NSLog( @"Push registration success." );
                });
            }
            else {
                NSLog( @"Push registration FAILED" );
                NSLog( @"ERROR: %@ - %@", error.localizedFailureReason, error.localizedDescription );
                NSLog( @"SUGGESTIONS: %@ - %@", error.localizedRecoveryOptions, error.localizedRecoverySuggestion );
            }
        }];
    } else {
        dispatch_async(dispatch_get_main_queue(), ^{
            [[UIApplication sharedApplication] registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeSound |    UIUserNotificationTypeAlert | UIUserNotificationTypeBadge) categories:nil]];
            [[UIApplication sharedApplication] registerForRemoteNotifications];
        });
    }
}
```

### 注册成功后，获取设备token

设备token是注册推送后，苹果服务器返回的一组用来识别当前设备的字符串，需要提交给服务器保存以备识别用户

```text
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    const unsigned *tokenBytes = [deviceToken bytes];
    NSString *hexToken = [NSString stringWithFormat:@"%08x%08x%08x%08x%08x%08x%08x%08x",
                          ntohl(tokenBytes[0]), ntohl(tokenBytes[1]), ntohl(tokenBytes[2]),
                          ntohl(tokenBytes[3]), ntohl(tokenBytes[4]), ntohl(tokenBytes[5]),
                          ntohl(tokenBytes[6]), ntohl(tokenBytes[7])];
    NSLog(@"token: %@", hexToken);
}
```

### 接受通知

接受通知分为冷启动和热启动两种场景，在收到通知的时候，可以获取到通知完整的Dictionary信息

#### 冷启动

冷启动是指app完全没有启动的状态收到推送，然后通过点击推送的方式进行启动app

```text
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.


    if (launchOptions) {
        if ([launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey]) {
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                NSDictionary *dict = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
                NSLog(@"launch:%@", dict);
                [self checkUserInfo:dict];
            });
        }
    }

    [self registerForRemoteNotifications];
    return YES;
}
```

#### 热启动

当前App正在运行中收到的通知

1. 在**iOS 10以上**会回调以下方法，正如注释所描述的，在收到通知，用户未点击时，会返回`userNotificationCenter:willPresentNotification: withCompletionHandler:`方法，而在用户操作之后，根据用户是否dismiss通知，会在`userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:`返回不同的action

```text
#pragma mark - UNUserNotificationCenterDelegate
// called when receive notification
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler API_AVAILABLE(ios(10.0)){
    NSLog(@"%s", __func__);
    NSLog(@"User Info : %@",notification.request.content.userInfo);
    completionHandler(UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionBadge);

    [self checkUserInfo:notification.request.content.userInfo];
}

//Called to let your app know which action was selected by the user for a given notification.
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)(void))completionHandler API_AVAILABLE(ios(10.0)){
    NSLog(@"%s", __func__);
    if (response.actionIdentifier == UNNotificationDefaultActionIdentifier) {
        NSLog(@"default action");
    }
    else if (response.actionIdentifier == UNNotificationDismissActionIdentifier){
        NSLog(@"Dismiss action");
    }

    NSLog(@"User Info : %@",response.notification.request.content.userInfo);
    completionHandler();
    [self checkUserInfo:response.notification.request.content.userInfo];
}
```

1. 在**iOS 10**以前的老版本，则会通过`application： didReceiveRemoteNotification：fetchCompletionHandler：`方法触发回调

```text
- (void) application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void(^)(UIBackgroundFetchResult))completionHandler {
    // iOS 10 will handle notifications through other methods
    if( SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO( @"10.0" ) )
    {
        NSLog( @"iOS version >= 10. Let NotificationCenter handle this one." );
        // set a member variable to tell the new delegate that this is background
        return;
    }
    NSLog( @"HANDLE PUSH, didReceiveRemoteNotification: %@", userInfo );

    // custom code to handle notification content

    if( [UIApplication sharedApplication].applicationState == UIApplicationStateInactive )
    {
        NSLog( @"INACTIVE" );
        completionHandler( UIBackgroundFetchResultNewData );
    }
    else if( [UIApplication sharedApplication].applicationState == UIApplicationStateBackground )
    {
        NSLog( @"BACKGROUND" );
        completionHandler( UIBackgroundFetchResultNewData );
    }
    else
    {
        NSLog( @"FOREGROUND" );
        completionHandler( UIBackgroundFetchResultNewData );
    }

    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [[NSNotificationCenter defaultCenter] postNotificationName:@"fgPush" object:nil userInfo:userInfo];
        NSLog(@"launch:%@", userInfo);
    });
}
```

## 常见问题

### 获取token失败

1. 检查profile与证书配置是否正确
2. 检查网络是否通畅
3. 重启手机

### 推送延时

1. 由于推送通知需要连接国外服务器，所以会有一定的延时，大概在1～2秒左右
2. 第三方不是超级会员之类的，大多会有延时，半小时到两小时不定都有可能
3. 遇到延时问题，可以使用自己的apns服务进行推送测试

### 证书识别错误/找不到证书

1. 关闭XCode
2. 重新导入证书和profile
3. 重启XCode

