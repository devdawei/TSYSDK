淘手游SDK - iOS集成文档
=====================

集成准备
-------

####  1. 获取AppID、Secret
集成淘手游SDK前，您需要先在淘手游SDK官网获取AppID、AppSecret。

####  2. 提供游戏充值的URL给淘手游
用户在游戏内购买东西时，淘手游通过此URL通知游戏厂商。

导入SDK
-------

#### 1. 使用CocoaPods自动导入
在您的Podfile里添加此行内容：
	```
	pod 'TSYSDK', :git => 'https://github.com/devdawei/TSYSDK.git'
	```

然后在Terminal下运行命令：
	```
	pod install
	```

#### 2. 下载SDK手动导入
您需要先下载SDK压缩包。[点击下载](http://www.baidu.com)

然后将压缩包中`libs`文件夹下的`TSYSDK`、`AlipaySDK`、`FDFullscreenPopGesture`、`IQKeyboardManager`文件夹和其中的文件拷贝进自己的项目。

在`Build Phases`选项卡的`Link Binary With Libraries`中，增加以下依赖库：
![frameworks](https://raw.githubusercontent.com/devdawei/TSYSDK/master/DocLinkImg/img_build_phases.png)

在`Build Settings`选项卡的`Other Linker Flags`中，添加`-ObjC`标识：
![-ObjC](https://raw.githubusercontent.com/devdawei/TSYSDK/master/DocLinkImg/img_build_settings.png)

工程配置
-------

#### 3. Info的配置
在`Info`选项卡的`URL Types`中，添加`URL Schemes`，在初始化SDK时要用到该参数，例如跳转到支付宝完成支付后，根据该参数跳转回应用。

**NOTE：** 图片中填写的`URL Schemes`仅为测试所用，您在配置时请一定要填写属于您APP自己的`URL Schemes`标识：
![-ObjC](https://raw.githubusercontent.com/devdawei/TSYSDK/master/DocLinkImg/img_info_urlSchemes.png)

在iOS9中，如果使用`URL Schemes`必须在`Info.plist`中将你要在外部调用的`URL Schemes`列为白名单，否则不能使用。所以还需要在`Info.plist`中增加`LSApplicationQueriesSchemes`设置：
```
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>weixin</string>
	<string>wechat</string>
</array>
```

在iOS9中新增了ATS，所以还需要在`Info.plist`增加ATS设置：
```
<key>NSAppTransportSecurity</key>
<dict>
	<key>NSAllowsArbitraryLoads</key>
	<true/>
</dict>
```
示例图片如下：
![LSApplicationQueriesSchemes](https://raw.githubusercontent.com/devdawei/TSYSDK/master/DocLinkImg/img_info_atsAndQueries.png)

基本功能使用
----------

#### 1. 在AppDelegate中初始化SDK
您需要先引入头文件：
```
#import <TSYSDK/TSYSDK.h>
```

然后在APP启动完成后配置SDK：
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
	// Override point for customization after application launch.

	/* 配置淘手游SDK */
	TSYSDKManager *manager = [TSYSDKManager sharedManager];
	[manager configWithAppID:@"56d136d2d6df472adbac8c87"
	              urlSchemes:@"tsysdkdemo"];
	/* 配置淘手游SDK浮窗按钮 */
	TSYSDKMenu *menu = [TSYSDKMenu sharedMenu];
	menu.window = self.window;

	return YES;
}
```

#### 2. 登录接口的使用
调用登录接口，需要先遵守`TSYSDKLoginDelegate`，并实现其中的代理方法：
```
/**
 登录状态码
 
 - TSYSDKLoginStatusSuccess:    登录成功
 - TSYSDKLoginStatusError:      登录失败
 - TSYSDKLoginStatusUserCancel: 用户取消登录
 */
typedef NS_ENUM(NSUInteger, TSYSDKLoginStatus) {
	TSYSDKLoginStatusSuccess,
	TSYSDKLoginStatusError,
	TSYSDKLoginStatusUserCancel
};

/**
 登录的回调方法
 
 @param manager    TSYSDKManager
 @param status     登录状态码
 @param resultDict 登录相关信息
 */
- (void)tsySDKManager:(TSYSDKManager *)manager
      loginWithStatus:(TSYSDKLoginStatus)status
           resultDict:(NSDictionary *)resultDict;
```

然后调用打开登录控制器接口：
```
TSYSDKManager *mgr = [TSYSDKManager sharedManager];
// 设置登录代理
mgr.loginDelegate = self;
// 调用打开登录控制器接口
[mgr openLoginController];
```

关于登录接口说明：
```
/**
 打开登录界面
 */
- (void)openLoginController;
```

#### 3. 进入游戏接口的使用
调用进入游戏接口，需要先遵守`TSYSDKEnterGameDelegate`，并实现其中的代理方法：
```
/**
 进入游戏状态码
 
 - TSYSDKEnterGameStatusSuccess:       进入游戏成功
 - TSYSDKEnterGameStatusError:         进入游戏失败
 - TSYSDKEnterGameStatusUserNeedLogin: 用户需要重新登录
 */
typedef NS_ENUM(NSUInteger, TSYSDKEnterGameStatus) {
	TSYSDKEnterGameStatusSuccess,
	TSYSDKEnterGameStatusError,
	TSYSDKEnterGameStatusUserNeedLogin
};

/**
进入游戏的回调方法

@param manager    TSYSDKManager
@param status     进入游戏状态码
@param resultDict 进入游戏相关信息
*/
- (void)tsySDKManager:(TSYSDKManager *)manager
enterGameWithStatus:(TSYSDKEnterGameStatus)status
		 resultDict:(NSDictionary *)resultDict;
```

然后调用进入游戏接口：
```
// 将用户信息字典转化为JSON字符串
NSDictionary *userInfoDict = @{
	                           @"userName": @"f2383c269392e5d78c154c00",
	                           @"roleId": @"324132145",
	                           @"roleName": @"大英雄",
	                           @"roleLevel": @"3",
	                           @"serverId": @"21",
	                           @"serverName": @"大混服",
	                           @"vipLevel": @"5",
	                           @"roleUnion": @"",
	                           @"roleBalance": @"0.00",
	                           };
NSString *jsonString = nil;
NSError *error = nil;
NSData *jsonData = [NSJSONSerialization dataWithJSONObject:userInfoDict
                                                   options:NSJSONWritingPrettyPrinted
                                                     error:&error];
if (error || !jsonData)
{
	NSLog(@"error: %@", error);
	jsonString = @"";
}
else
{
	jsonString = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
}
NSLog(@"jsonString: \n%@", jsonString);

TSYSDKManager *mgr = [TSYSDKManager sharedManager];
// 设置进入游戏代理
mgr.enterGameDelegate = self;
// 调用进入游戏接口
[mgr enterGameWithUserInfo:jsonString];
```

关于进入游戏接口说明：
```
/**
 进入游戏
 
 @param userInfo 包含用户和游戏角色信息的 JSON 字符串（保留参数，目前可传空字符串(@"")）
 */
- (void)enterGameWithUserInfo:(NSString *)userInfo;
```

#### 4. 支付接口的使用
调用支付接口，需要先遵守`TSYSDKPayDelegate`，并实现其中的代理方法：
```
/**
 支付方式
 - TSYPayTypeUnSelected: 还未选择支付方式
 - TSYPayTypeTSYBalance: 淘手游余额支付
 - TSYPayTypeTenPay:     财付通
 - TSYPayTypeAli:        支付宝
 - TSYPayTypeWeChat:     微信
 - TSYPayTypeUnion:      银联
 */
typedef NS_ENUM(NSUInteger, TSYPayType) {
    TSYPayTypeUnSelected = -1,
    TSYPayTypeTSYBalance = 0,
    TSYPayTypeTenPay = 15,
    TSYPayTypeAli = 16,
    TSYPayTypeWeChat = 17,
    TSYPayTypeUnion = 18
};

/**
 支付的状态码
 
 - TSYPayStatusSuccess:             订单支付成功
 - TSYPayStatusError:               订单支付失败
 - TSYPayStatusProcessing:          订单正在处理中
 - TSYPayStatusUserCancel:          用户中途取消支付
 - TSYPayStatusNetworkError:        网络错误
 - TSYPayStatusUserNeedLogin:       用户需要重新登录
 */
typedef NS_ENUM(NSUInteger, TSYPayStatus) {
    TSYPayStatusSuccess,
    TSYPayStatusError,
    TSYPayStatusProcessing,
    TSYPayStatusUserCancel,
    TSYPayStatusNetworkError,
    TSYPayStatusUserNeedLogin
};

/**
 支付成功的回调方法
 
 @param manager TSYSDKManager
 @param type    支付方式
 */
- (void)tsySDKManager:(TSYSDKManager *)manager
   paySuccessWithType:(TSYPayType)type
               status:(TSYPayStatus)status;

/**
 支付失败的回调方法
 
 @param manager TSYSDKManager
 @param type    支付方式
 @param status  支付失败类型
 */
- (void)tsySDKManager:(TSYSDKManager *)manager
   payFailureWithType:(TSYPayType)type
               status:(TSYPayStatus)status;

```

然后调用支付接口：
```
TSYSDKManager *mgr = [TSYSDKManager sharedManager];
// 设置支付代理
mgr.payDelegate = self;
// 调用支付接口
[mgr payWithServiceAreaId:@"21"
               goodsPrice:@"0.01"
                goodsName:@"测试商品"
               goodsCount:@"1"
                goodsData:@""
                 totalFee:@"0.01"];
```

关于支付接口说明：
```
/**
 支付
 
 @param servicearea_id 区服
 @param goods_price    单价
 @param goods_name     商品名称
 @param goods_count    数量
 @param goods_data     其他参数
 @param total_fee      总价
 */
- (void)payWithServiceAreaId:(NSString *)servicearea_id
                  goodsPrice:(NSString *)goods_price
                   goodsName:(NSString *)goods_name
                  goodsCount:(NSString *)goods_count
                   goodsData:(NSString *)goods_data
                    totalFee:(NSString *)total_fee;
```

#### 4. 切换账号接口的使用
调用切换账号接口，需要先遵守`TSYSDKSwitchAccountDelegate`，并实现其中的代理方法：
```
/**
 切换账号的状态码
 
 - TSYSDKSwitchAccountStatusSuccess: 切换账号成功
 - TSYSDKSwitchAccountStatusError:   切换账号失败
 */
typedef NS_ENUM(NSUInteger, TSYSDKSwitchAccountStatus) {
    TSYSDKSwitchAccountStatusSuccess,
    TSYSDKSwitchAccountStatusError
};

/**
 切换账号（退出账号）
 
 @param manager TSYSDKManager
 @param status  切换账号的状态码
 */
- (void)  tsySDKManager:(TSYSDKManager *)manager
switchAccountWithStatus:(TSYSDKSwitchAccountStatus)status;
```

然后调用切换账号接口：
```
TSYSDKManager *mgr = [TSYSDKManager sharedManager];
// 设置切换账号代理
mgr.switchAccountDelegate = self;
// 调用切换账号接口
[mgr switchAccount];
```

关于切换账号接口说明：
```
/**
 切换账号（退出账号）
 */
- (void)switchAccount;
```

恭喜您，至此已经成功集成和使用淘手游SDK。
