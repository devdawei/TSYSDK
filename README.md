淘手游SDK - iOS集成文档
=====================

集成准备
-------

###  1. 获取AppID、Secret
集成淘手游SDK前，您需要先在[淘手游SDK官网](http://www.taoshouyou.com)获取AppID、AppSecret。

###  2. 提供游戏充值的URL给淘手游
用户在游戏内购买东西时，淘手游通过此URL通知游戏厂商。

导入SDK
-------

### 1. 使用CocoaPods自动导入
在您的Podfile里添加此行内容：
```
pod 'TSYSDK', :git => 'https://github.com/devdawei/TSYSDK.git'
```

然后在Terminal下运行命令：
```
pod install
```

### 2. 下载SDK手动导入
您需要先[下载SDK压缩包](http://www.taoshouyou.com)。

然后将压缩包中`libs`文件夹下的`TSYSDK`、`AlipaySDK`、`FDFullscreenPopGesture`、`IQKeyboardManager`文件夹和其中的文件拷贝进自己的项目。

在`Build Phases`选项卡的`Link Binary With Libraries`中，增加以下依赖库：
![frameworks](https://raw.githubusercontent.com/devdawei/TSYSDK/master/DocLinkImg/img_build_phases.png)

在`Build Settings`选项卡的`Other Linker Flags`中，添加`-ObjC`标识：
![-ObjC](https://raw.githubusercontent.com/devdawei/TSYSDK/master/DocLinkImg/img_build_settings.png)

工程配置
-------

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

### 1. 在AppDelegate中初始化SDK
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

还需要在AppDelegate中实现处理openURL的代理方法，在代理方法中处理支付结果：
```
// NOTE: iOS 9 之前在此代理方法中处理 openURL
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    // 调用淘手游处理支付结果的方法
    [[TSYSDKManager sharedManager] handleOpenUrl:url options:nil];
    
    return YES;
}

// NOTE: iOS 9 之后使用新API处理 openURL
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options
{
    // 调用淘手游处理支付结果的方法
    [[TSYSDKManager sharedManager] handleOpenUrl:url options:options];
    
    return YES;
}
```

### 2. 登录接口的使用
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

### 3. 进入游戏接口的使用
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

### 4. 支付接口的使用
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

### 5. 切换账号接口的使用
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

**至此，您已成功接入iOS端淘手游SDK。**

淘手游SDK - 服务器端接入
=====================

淘手游提供的Http接口
-----------------

### 1. 淘手游提供的Http接口

**NOTE：**只有这个接口才能保证用户是通过帐号密码验证的，一定要使用。游戏服务器向平台服务器验证登录用户令牌是否有效的接口说明：

接口链接：[http://sdk.taoshouyou.com/user/checktoken](http://sdk.taoshouyou.com/user/checktoken)

提交方式：POST

参数说明：

| 参数名               | 参数类型              | 参数说明 |
| :-----------------: | :------------------: | :----------------------- |
| userid              | String               | 游戏用户对应id |
| token               | String               | 令牌，验证用户是否有效 |
| appid               | String               | 淘手游平台对应的appid |

返回值说明：

| 参数名               | 参数说明 |
| :-----------------: | :------------------ |
| success             | 用户令牌有效 |
| fail                | 用户令牌无效 |

用户登录流程：
![loginProcess](https://raw.githubusercontent.com/devdawei/TSYSDK/master/DocLinkImg/img_service_loginProcess.png)

### 2. 游戏方服务器提供的http接口（游戏内购买接口）

**NOTE：**淘手游服务器完成支付流程后将通知游戏服务器修改玩家账户内相关数据的接口, 只有用户支付成功才会产生通知。参照用户充值流程图中9-10步骤。

接口地址：需游戏方提供

提交方式：POST

参数说明：具体参数可商量约定

| 参数名               | 参数类型              | 参数说明 |
| :-----------------: | :------------------: | :----------------------- |
| appid               | String               | 淘手游平台对应的appid |
| bizno               | String               | 淘手游订单编号，淘手游订单系统的唯一编号 |
| total_fee           | Double               | 玩家充值金额, 保留2位小数 |
| goods_data          | JSON String          | 游戏数据JSON字符串
| signature           | String               | MD5签名值（详细内容查看下面的签名算法）|

签名算法：将除signature外所有参数键值对按键以字典序升序排列后拼在一起，例如："k1=v1&k2=v2&k3=v3"；然后在拼好的字符串末尾直接拼接上appSecret；上述字符串的MD5值即为签名的值，例如：appSecret=443e70f1c0a51337a9d772fee93fcb30，则sign=md5(“k1=v1&k2=v2&k3=v3 443e70f1c0a51337a9d772fee93fcb30”) 

签名验证：将除signature外其它所有参数生成签名值和signature比较,相同则正确，不同则被篡改。

注：签名中的appSecret由平台方提供，请求中间不传递这个参数， 拼接时无键名。

返回值定义：

| 参数名               | 参数说明 |
| :-----------------: | :------------------ |
| success             | 充值通知服务器成功，中止发送通知。同一订单号多次请求游戏已充值成功也要返回success才会中止发送通知 |
| fail                | 充值通知服务器失败，延时1分钟后再次发送通知，如果仍然失败则加倍延时时间后通知，直到返回success为止才会中止通知。 |

用户登录流程：
![payProcess](https://raw.githubusercontent.com/devdawei/TSYSDK/master/DocLinkImg/img_service_payProcess.png)

**至此，您已成功接入服务器端淘手游SDK。**
