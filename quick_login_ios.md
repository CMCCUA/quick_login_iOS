# 1. 开发环境配置 

sdk技术问题沟通QQ群：609994083</br>

**注：SDK在获取token过程中，用户手机必须在打开数据网络情况下才能获取成功，纯wifi环境下会返回错误码**</br>

## 1.1. 环境配置及发布

1. xcode版本需使用9.0以上，否则会报错
2. 导入统一认证framework，直接将统一认证`TYRZSDK.framework`拖到项目中
3. 在Xcode中找到`TARGETS-->Build Setting-->Linking-->Other Linker Flags`在这选项中需要添加`-ObjC`

</br>

## 1.2. Hello 统一认证 

本节内容主要面向新接入统一认证的开发者，介绍快速集成统一认证的基本服务的方法。

### 1.2.1. 统一认证登录流程

![](image/19.png)

由流程图可知，业务客户端集成SDK后只需要完成2步集成实现登录

    1.    调用登录接口获取token
    2.    携带token请求登录

</br>

### 1.2.2. 统一认证登录集成步骤

**第一步：**

在`appDelegate.m`中的`didFinish`函数中添加初始化代码。初始化代码只需要执行一次就可以。

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

     [TYRZSDK registerAppId:APPID appKey:APPKEY];
    return YES;
}
```

**第二步：**

1、根据SDK内部提供的UAAuthViewController类，直接创建控制器或继承该类创建其子类控制器，进行自定义布局登录页。</br>
2、调用取号接口获取手机号码掩码成功后，可调用授权接口获取token及openId。


</br>

<div STYLE="page-break-after: always;"></div>

#2. SDK方法说明

## 2.1. 初始化

### 2.1.1. 方法描述

**功能**

用于初始化appId、appKey设置。

**原型**

```objective-c
+ (void)registerAppId:(NSString *)appId appKey:(NSString *)appKey;
```
</br>

### 2.1.2. 参数说明

**请求参数**


| 参数     | 类型       | 说明       | 是否必填 |
| ------ | -------- | -------- | ---- |
| appID  | NSString | 应用的appid | 是    |
| appKey | NSString | 应用密钥     | 是    |



**响应参数**

无

</br>

## 2.2. 取号


### 2.2.1. 方法描述

**功能**

本方法用于发起取号请求，并返回用户当前网络环境是否具备取号条件。SDK将在后台完成网络判断、数据网络切换等内部操作并向网关请求申请获取用户本机号码。取号请求成功后，开发者就可以调用并弹出由开发者自定义布局的授权页面。


**原型**

```objective-c
+ (void)getPhoneNumberWithTimeout:(NSTimeInterval)duration completion:(void (^)(NSDictionary * sender))completion;
```
</br>

### 2.2.2. 参数说明

**请求参数**

| 参数       | 类型           | 说明                                    |
| ---------- | -------------- | --------------------------------------- |
| duration   | NSTimeInterval | 自定义取号超时时间（默认8000毫秒），单位：毫秒 |
| completion | Block  | 取号回调                                |

**响应参数**

| 参数          | 类型     | 说明                          |
| ------------- | -------- | ----------------------------- |
| resultCode    | NSString | 返回相应的结果码              |
| desc          | NSString | 调用描述                      |
| securityphone | NSString | 手机号码掩码，如“138XXXX0000” |

</br>

### 2.2.3. 示例

**请求示例代码**

```objective-c
 [TYRZSDK getPhoneNumberWithTimeout: 8000 completion: ^ (NSDictionary * _Nonnull sender) {
        if ([sender[@ "resultCode"] isEqualToString: @"103000"]) {
            NSLog(@ "取号成功:%@", sender);
        } else {
            NSLog(@ "取号失败:%@", sender);
        }
    }];
```

</br>

**响应示例代码**

```
{
    resultCode = 103000;
    desc = "success";
    securityphone = "138XXXX0000"
}
```

</br>

## 2.3. 授权登录

### 2.3.1. 方法描述

**功能**

本方法用于实现：

1. 创建加载SDK内部提供的UAAuthViewController空白模板控制器（或继承UAAuthViewController来创建登录页控制器）
2. 用户点击登录授权后返回取号凭证等参数
</br>

**原型**

```objective-c
+ (void)getAuthorizationWithAuthViewController:(UAAuthViewController *_Nullable)authVC completion:(void (^)(NSDictionary *sender))completion;
```
</br>

### 2.3.2. 参数说明

**请求参数**

| 参数     | 类型                 | 说明                                                         |
| -------- | -------------------- | ------------------------------------------------------------ |
| authVC   | UAAuthViewController | 授权页面，由开发者完成页面的设计布局。**当authVC传值为nil时，将不弹出授权页，登录方式为隐式登录** |
| complete | Block        | 登录回调                                                     |

</br>

**响应参数**

| 参数       | 类型     | 说明                                                         | 是否必填   |
| ---------- | -------- | ------------------------------------------------------------ | ---------- |
| resultCode | NSString | 返回相应的结果码                                             | 是         |
| token      | NSString | 成功时返回：临时凭证，token有效期2min，一次有效，同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 | 成功时必填 |
| openId     | NSString | 成功时返回：用户身份唯一标识 （隐式登录不返回openId字段）        | 成功时必填 |
| desc       | NSString | 调用描述                                                     | 否         |

</br>

### 2.3.3. 示例

以下提供四种方式的示例代码，供开发者参考：
</br>
**示例代码1：直接创建UAAuthViewController控制器，授权页前置**
```objective-c
-(void)preplan{
    // 1、自定义布局一键登录授权页
    UAAuthViewController * authVC = [[UAAuthViewController alloc]init];
    authVC.view.backgroundColor = UIColor.whiteColor;
    self.authVC = authVC;
    UINavigationController * nav = [[UINavigationController alloc]initWithRootViewController:authVC];
    nav.navigationBar.translucent = NO;
    nav.navigationBar.barStyle = UIBarStyleBlack;
    nav.navigationBar.barTintColor = [UIColor blueColor];
    nav.navigationBar.tintColor = [UIColor whiteColor];

    //  SecurityPhoneLable
    CGFloat maginX = 50.f;
    CGFloat width = authVC.view.frame.size.width;
    UILabel *securityPhoneLable = [[UILabel alloc] init];
    securityPhoneLable.textAlignment = NSTextAlignmentCenter;
    securityPhoneLable.textColor = [UIColor redColor];
    securityPhoneLable.frame = CGRectMake(maginX, 100, width-2*maginX, 40);
    self.securityPhoneLable = securityPhoneLable;
    [authVC.view addSubview:securityPhoneLable];
    
    // AuthLoginButton
    UIButton *authButton = [UIButton buttonWithType:UIButtonTypeCustom];
     authButton.userInteractionEnabled = NO;
    [authButton setTitle:@"授权登录" forState:UIControlStateNormal];
    authButton.layer.cornerRadius = 4;
    authButton.layer.masksToBounds = YES;
    authButton.frame = CGRectMake(maginX, CGRectGetMaxY(securityPhoneLable.frame)+50, width-2*maginX, 40);
    authButton.backgroundColor = [UIColor grayColor];
    [authButton addTarget:self action:@selector(authorizeLoginButtonClick) forControlEvents:UIControlEventTouchUpInside];
    [authVC.view addSubview:authButton];
    
    [self presentViewController:nav animated:YES completion:nil];
    
    // 2、调用取号方法
    __weak typeof(self) weakSelf = self;
    [TYRZSDK getPhoneNumberWithTimeout:8000 completion:^(NSDictionary * _Nonnull sender){
        authButton.userInteractionEnabled = YES;
        authButton.backgroundColor = [UIColor orangeColor];
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"取号成功:%@",sender);
            // 显示手机号码掩码
            weakSelf.securityPhoneLable.text = sender[@"securityphone"];
        } else {
            NSLog(@"取号失败:%@",sender);
            [weakSelf.authVC dismissViewControllerAnimated:YES completion:nil];
        }
    }];
}

// 3、授权登录按钮点击事件，调用授权方法
-(void)authorizeLoginButtonClick{
    __weak typeof(self) weakSelf = self;
    [TYRZSDK getAuthorizationWithAuthViewController:weakSelf.authVC completion:^(NSDictionary * _Nonnull sender) {
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"授权登录成功:%@",sender);
        } else {
            NSLog(@"授权登录失败:%@",sender);
        }
        [weakSelf.authVC dismissViewControllerAnimated:YES completion:nil];
    }];
}
```
</br>

**示例代码2：直接创建UAAuthViewController控制器，授权页后置**
```objective-c
-(void)postposition{
    // 1、调用取号方法
    __weak typeof(self) weakSelf = self;
    [TYRZSDK getPhoneNumberWithTimeout:8000 completion:^(NSDictionary * _Nonnull sender){
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"取号成功:%@",sender);
            // 取号成功则创建授权页
            [weakSelf showAuthVC:sender];
        } else {
            NSLog(@"取号失败:%@",sender);
            [weakSelf dismissViewControllerAnimated:YES completion:nil];
        }
    }];
}

-(void)showAuthVC:(NSDictionary *)dict{
    // 2、创建自定义布局一键登录授权页
    UAAuthViewController * authVC = [[UAAuthViewController alloc]init];
    authVC.view.backgroundColor = UIColor.whiteColor;
    self.authVC = authVC;
    UINavigationController * nav = [[UINavigationController alloc]initWithRootViewController:authVC];
    nav.navigationBar.translucent = NO;
    nav.navigationBar.barStyle = UIBarStyleBlack;
    nav.navigationBar.barTintColor = [UIColor blueColor];
    nav.navigationBar.tintColor = [UIColor whiteColor];
    
    //  SecurityPhoneLable
    CGFloat maginX = 50.f;
    CGFloat width = authVC.view.frame.size.width;
    UILabel *securityPhoneLable = [[UILabel alloc] init];
    securityPhoneLable.text = dict[@"securityphone"]; // 显示手机号码掩码
    securityPhoneLable.textAlignment = NSTextAlignmentCenter;
    securityPhoneLable.textColor = [UIColor redColor];
    securityPhoneLable.frame = CGRectMake(maginX, 100, width-2*maginX, 40);
    self.securityPhoneLable = securityPhoneLable;
    [authVC.view addSubview:securityPhoneLable];
    
    // AuthLoginButton
    UIButton *authButton = [UIButton buttonWithType:UIButtonTypeCustom];
    [authButton setTitle:@"授权登录" forState:UIControlStateNormal];
    authButton.layer.cornerRadius = 4;
    authButton.layer.masksToBounds = YES;
    authButton.frame = CGRectMake(maginX, CGRectGetMaxY(securityPhoneLable.frame)+50, width-2*maginX, 40);
    authButton.backgroundColor = [UIColor orangeColor];
    [authButton addTarget:self action:@selector(authorizeLoginButtonClick) forControlEvents:UIControlEventTouchUpInside];
    [authVC.view addSubview:authButton];
    
    [self presentViewController:nav animated:YES completion:nil];   
  } 
  
  // 3、授权登录按钮点击事件，调用授权方法
-(void)authorizeLoginButtonClick{
    __weak typeof(self) weakSelf = self;
    [TYRZSDK getAuthorizationWithAuthViewController:weakSelf.authVC completion:^(NSDictionary * _Nonnull sender) {
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"授权登录成功:%@",sender);
        } else {
            NSLog(@"授权登录失败:%@",sender);
        }
        [weakSelf.authVC dismissViewControllerAnimated:YES completion:nil];
    }];
}

```
</br>

**示例代码3：继承UAAuthViewController创建控制器，授权页前置**
```objective-c
-(void)preplan{
    // 1、创建一键登录授权页控制器
    // CustomAuthViewController是继承UAAuthViewController的子类
    CustomAuthViewController *authVC = [[CustomAuthViewController alloc]init];
    UINavigationController * nav = [[UINavigationController alloc]initWithRootViewController:authVC];
    nav.navigationBar.translucent = NO;
    nav.navigationBar.barStyle = UIBarStyleBlack;
    nav.navigationBar.barTintColor = [UIColor blueColor];
    nav.navigationBar.tintColor = [UIColor whiteColor];
    [self presentViewController:nav animated:YES completion:nil];
}

// CustomAuthViewController.m文件里:
- (void)viewDidLoad {
    [super viewDidLoad];
    [self setupUI];
    [self getPhonenumber];
}

// 自定义布局UI控件
-(void)setupUI{
    //  SecurityPhoneLable
    CGFloat maginX = 50.f;
    CGFloat width = authVC.view.frame.size.width;
    UILabel *securityPhoneLable = [[UILabel alloc] init];
    securityPhoneLable.textAlignment = NSTextAlignmentCenter;
    securityPhoneLable.textColor = [UIColor redColor];
    securityPhoneLable.frame = CGRectMake(maginX, 100, width-2*maginX, 40);
    self.securityPhoneLable = securityPhoneLable;
    [self.view addSubview:securityPhoneLable];
    
    // AuthLoginButton
    UIButton *authButton = [UIButton buttonWithType:UIButtonTypeCustom];
    [authButton setTitle:@"授权登录" forState:UIControlStateNormal];
    authButton.layer.cornerRadius = 4;
    authButton.layer.masksToBounds = YES;
    authButton.frame = CGRectMake(maginX, CGRectGetMaxY(securityPhoneLable.frame)+50, width-2*maginX, 40);
    authButton.backgroundColor = [UIColor orangeColor];
    [authButton addTarget:self action:@selector(authorizeLoginButtonClick) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:authButton];
}

// 2、调用取号方法
-(void)getPhonenumber{
    __weak typeof(self) weakSelf = self;
    [TYRZSDK getPhoneumberWithTimeout:8000 completion:^(NSDictionary * _Nonnull sender){
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"取号成功:%@",sender);
            // 显示手机号码掩码
            weakSelf.securityPhoneLable.text = sender[@"securityphone"];
        } else {
            NSLog(@"取号失败:%@",sender);
            [weakSelf dismissViewControllerAnimated:YES completion:nil];
        }
    }];
}

// 3、授权登录按钮点击事件，调用授权方法
-(void)authorizeLoginButtonClick{
    __weak typeof(self) weakSelf = self;
    [TYRZSDK getAuthorizationWithAuthViewController:weakSelf.authVC completion:^(NSDictionary * _Nonnull sender) {
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"授权登录成功:%@",sender);
        } else {
            NSLog(@"授权登录失败:%@",sender);
        }
        [weakSelf dismissViewControllerAnimated:YES completion:nil];
    }];
}

```
</br>

**示例代码4：继承UAAuthViewController创建控制器，授权页后置**
```objective-c
-(void)postposition{
    // 1、调用取号方法
    __weak typeof(self) weakSelf = self;
    [TYRZSDK getPhoneNumberWithTimeout:8000 completion:^(NSDictionary * _Nonnull sender){
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"取号成功:%@",sender);
            // 取号成功则创建授权页
            [weakSelf showAuthVC:sender];
        } else {
            NSLog(@"取号失败:%@",sender);
            [weakSelf dismissViewControllerAnimated:YES completion:nil];
        }
    }];
}

-(void)showAuthVC:(NSDictionary *)dict{
    // 2、创建一键登录授权页控制器
    // CustomAuthViewController是继承UAAuthViewController的子类
    CustomAuthViewController *authVC = [[CustomAuthViewController alloc]init];
    authVC.securityPhone =dict[@"securityphone"]; //手机码掩码，传值
    UINavigationController * nav = [[UINavigationController alloc]initWithRootViewController:authVC];
    nav.navigationBar.translucent = NO;
    nav.navigationBar.barStyle = UIBarStyleBlack;
    nav.navigationBar.barTintColor = [UIColor blueColor];
    nav.navigationBar.tintColor = [UIColor whiteColor];
    [self presentViewController:nav animated:YES completion:nil];
}

// CustomAuthViewController.m文件里:
- (void)viewDidLoad {
    [super viewDidLoad];
    [self setupUI];
    self.securityPhoneLable.text = self.securityPhone;
}

// 自定义布局UI控件
-(void)setupUI{
    //  SecurityPhoneLable
    CGFloat maginX = 50.f;
    CGFloat width = authVC.view.frame.size.width;
    UILabel *securityPhoneLable = [[UILabel alloc] init];
    securityPhoneLable.textAlignment = NSTextAlignmentCenter;
    securityPhoneLable.textColor = [UIColor redColor];
    securityPhoneLable.frame = CGRectMake(maginX, 100, width-2*maginX, 40);
    self.securityPhoneLable = securityPhoneLable;
    [self.view addSubview:securityPhoneLable];
    
    // AuthLoginButton
    UIButton *authButton = [UIButton buttonWithType:UIButtonTypeCustom];
    [authButton setTitle:@"授权登录" forState:UIControlStateNormal];
    authButton.layer.cornerRadius = 4;
    authButton.layer.masksToBounds = YES;
    authButton.frame = CGRectMake(maginX, CGRectGetMaxY(securityPhoneLable.frame)+50, width-2*maginX, 40);
    authButton.backgroundColor = [UIColor orangeColor];
    [authButton addTarget:self action:@selector(authorizeLoginButtonClick) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:authButton];
}

// 3、授权登录按钮点击事件，调用授权方法
-(void)authorizeLoginButtonClick{
    __weak typeof(self) weakSelf = self;
    [TYRZSDK getAuthorizationWithAuthViewController:weakSelf.authVC completion:^(NSDictionary * _Nonnull sender) {
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"授权登录成功:%@",sender);
        } else {
            NSLog(@"授权登录失败:%@",sender);
        }
        [weakSelf dismissViewControllerAnimated:YES completion:nil];
    }];
}

```

</br>

## 2.4. 隐式登录

本SDK不再单独提供隐式登录方法，开发者如果需要使用本SDK实现隐式登录做本机号码校验，在调用授权登录方法时，将authVC对象传入值设为nil即可，具体可参考下述代码来实现：

```objective-c
-(void)loginImplicity{
    // 1.调用取号方法
    [TYRZSDK getPhoneumberWithTimeout:8000 completion:^(NSDictionary * _Nonnull sender){
        if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
            NSLog(@"取号成功:%@",sender);
            // 2.调用授权方法
            [TYRZSDK getAuthorizationWithAuthViewController:nil completion:^(NSDictionary * _Nonnull sender) {
                if ([sender[@"resultCode"] isEqualToString:@"103000"]) {
                    NSLog(@"隐式登录成功:%@",sender);
                }else{
                    NSLog(@"隐式登录失败:%@",sender);
                }
            }];
        } else {
            NSLog(@"取号失败:%@",sender);
        }
    }];
}
```


# 3. 平台接口说明

## 3.1. 获取用户信息接口

业务平台或服务端携带用户授权成功后的token来调用统一认证服务端获取用户手机号码等信息。**注：本接口仅适用于5.3.0及以上版本SDK**

### 3.1.1. 业务流程

SDK在获取token过程中，用户手机必须在打开数据网络情况下才能获取成功，纯wifi环境下会返回错误码

![](image/19.png)

### 3.1.2. 接口说明

**请求地址：**https://www.cmpassport.com/unisdk/rsapi/loginTokenValidate

**协议：** HTTPS 

**请求方法：** POST+json,Content-type设置为application/json

**回调地址：**请参考开发者接入流程文档

</br>

### 3.1.3. 参数说明

**请求参数**

| 参数                  |   类型   |  约束  | 说明                                       |
| :------------------ | :----: | :--: | :--------------------------------------- |
| version             | string |  必选  | 填2.0                                     |
| msgid               | string |  必选  | 标识请求的随机数即可(1-36位)                        |
| systemtime          | string |  必选  | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| strictcheck         | string |  必选  | 暂时填写"0"                                  |
| appid               | string |  必选  | 业务在统一认证申请的应用id                           |
| expandparams        | string |  可选  | 扩展参数                                     |
| token               | string |  必选  | 需要解析的凭证值。                                |
| sign                | string |  必选  | 当**encryptionalgorithm≠"RSA"**时，sign = MD5（appid + version + msgid + systemtime + strictcheck + token + appkey)（注：“+”号为合并意思，不包含在被加密的字符串中），输出32位大写字母；</br>当**encryptionalgorithm="RSA"**，业务端RSA私钥签名（appid+token）, 服务端使用业务端提供的公钥验证签名（公钥可以在开发者社区配置）。 |
| encryptionalgorithm | string |  可选  | 开发者如果需要使用非对称加密算法时，填写“RSA”。（当该值不设置为“RSA”时，执行MD5签名校验） |

</br>

**响应参数**

| 参数         | 类型   | 约束 | 说明                                                         |
| ------------ | ------ | ---- | ------------------------------------------------------------ |
| inresponseto | string | 必选 | 对应的请求消息中的msgid                                      |
| systemtime   | string | 必选 | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| resultcode   | string | 必选 | 返回码                                                       |
| msisdn       | string | 可选 | 表示手机号码，如果加密方式为RSA，应用需要用私钥进行解密      |

</br>

### 3.1.4. 示例

**请求示例**

```
{
    appid = 3000******76;
    msgid = 335e06a28f064b999d6a25e403991e4c;
    sign = 213EF8D0CC71548945A83166575DFA68;
    strictcheck = 0;
    systemtime = 20180129112955435;
    token = STsid0000001517196594066OHmZvPMBwn2MkFxwvWkV12JixwuZuyDU;
    version = "2.0";
}
```

**响应示例**

```
{
    inresponseto = 335e06a28f064b999d6a25e403991e4c;
    msisdn = 14700000000;
    resultCode = 103000;
    systemtime = 20180129112955477;
}
```

<div STYLE="page-break-after: always;"></div>

## 3.2. 本机号码校验接口

校验用户输入的号码是否本机号码。
应用将手机号码传给统一认证SDK，统一认证SDK向统一认证服务端发起本机号码校验请求，统一认证服务端通过网关或者短信上行获取本机手机号码和第三方应用传输的手机号码进行校验，返回校验结果。</br>

### 3.2.1. 业务流程

SDK在获取token过程中，用户手机必须在打开数据网络情况下才能获取成功，纯wifi环境下会返回错误码。**注：本业务目前仅支持中国移动号码，建议开发者在使用该功能前，判断当前用户手机运营商**

![1](image\1.1.png)

</br>

### 3.2.2. 接口说明

**调用次数说明：**本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。

**请求地址：** https://www.cmpassport.com/openapi/rs/tokenValidate

**协议：** HTTPS

**请求方法：** POST+json,Content-type设置为application/json

**回调地址：**请参考开发者接入流程文档

</br>

### 3.2.3.  参数说明

*1、json形式的报文交互必须是标准的json格式；*

*2、发送时请设置content type为 application/json*

**请求参数**

| 参数            | 类型     | 层级    | 约束                    | 说明                                       |
| ------------- | ------ | ----- | --------------------- | ---------------------------------------- |
| **header**    |        | **1** | 必选                    |                                          |
| version       | string | 2     | 必选                    | 版本号,初始版本号1.0,有升级后续调整                     |
| msgId         | string | 2     | 必选                    | 使用UUID标识请求的唯一性                           |
| timestamp     | string | 2     | 必选                    | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId         | string | 2     | 必选                    | 应用ID                                     |
| **body**      |        | **1** | 必选                    |                                          |
| openType      | String | 2     | 否，requestertype字段为0时是 | 运营商类型：</br>1:移动;</br>2:联通;</br>3:电信;</br>0:未知 |
| requesterType | String | 2     | 是                     | 请求方类型：</br>0:APP；</br>1:WAP              |
| message       | String | 2     | 否                     | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码      |
| expandParams  | String | 2     | 否                     | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
| phoneNum      | String | 2     | 是                     | 待校验的手机号码的64位sha256值，字母大写。（手机号码 + appKey + timestamp， “+”号为合并意思）（注：建议开发者对用户输入的手机号码的格式进行校验，增加校验通过的概率） |
| token         | String | 2     | 是                     | 身份标识，字符串形式的token                         |
| sign          | String | 2     | 是                     | 签名，HMACSHA256( appId +     msgId + phonNum + timestamp + token + version)，输出64位大写字母 （注：“+”号为合并意思，不包含在被加密的字符串中,appkey为秘钥, 参数名做自然排序（Java是用TreeMap进行的自然排序）） |
|               |        |       |                       |                                          |

**响应参数**

| 参数           | 层级    | 类型     | 约束   | 说明                                       |
| ------------ | ----- | :----- | :--- | :--------------------------------------- |
| **header**   | **1** |        | 必选   |                                          |
| msgId        | 2     | string | 必选   | 对应的请求消息中的msgid                           |
| timestamp    | 2     | string | 必选   | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId        | 2     | string | 必选   | 应用ID                                     |
| resultCode   | 2     | string | 必选   | 规则参见4.3平台返回码                             |
| **body**     | **1** |        | 必选   |                                          |
| resultDesc   | 2     | String | 必选   | 描述参见4.3平台返回码                             |
| message      | 2     | String | 否    | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码      |
| expandParams | 2     | String | 否    | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
|              |       |        |      |                                          |

</br>

### 3.2.4. 示例

**请求示例**

```
{
    body =     {
        openType = 1;
        phoneNum =0A2050AC434A32DE684745C829B3DE570590683FAA1C9374016EF60390E6CE76;
        requesterType = 0;
        sign = 87FCAC97BCF4B0B0D741FE1A85E4DF9603FD301CB3D7100BFB5763CCF61A1488;
        token = STsid0000001517194515125yghlPllAetv4YXx0v6vW2grV1v0votvD;
    };
    header =     {
        appId = 3000******76;
        msgId = f11585580266414fbde9f755451fb7a7;
        timestamp = 20180129105523519;
        version = "1.0";
    };
}
```



**响应示例**

```
{
    body =     {
        message = "";
        resultDesc = "\U662f\U672c\U673a\U53f7\U7801";
    };
    header =     {
        appId = 3000******76;
        msgId = f11585580266414fbde9f755451fb7a7;
        resultCode = 000;
        timestamp = 20180129105523701;
    };
}
```

<div STYLE="page-break-after: always;"></div>

# 4. 返回码说明

## 4.1. SDK返回码

使用SDK时，SDK会在认证结束后将结果回调给开发者，其中结果为JSONObject对象，其中resultCode为结果响应码，103000代表成功，其他为失败。成功时在根据token字段取出身份标识。失败时根据resultCode定位失败原因。

| 错误编号   | 返回码描述                       |
| ------ | --------------------------- |
| 103000 | 成功                          |
| 102101 | 无网络                         |
| 102102 | 网络异常                        |
| 102103 | 未开启数据网络                     |
| 102109 | 网络错误，1. 欠费卡；2. 网络环境不佳       |
| 102121 | 用户取消登录                      |
| 102208 | SDK请求参数错误                   |
| 102302 | 没有进行初始化参数                   |
| 102506 | 请求出错                        |
| 102507 | 请求超时  |
| 103125 | 手机号码格式错误                    |
| 103126 | 手机号码不存在                     |
| 105001 | 联通取号失败                      |
| 200002 | 手机未安装SIM卡                   |
| 200009 | Bundle ID与服务器填写的不一致         |
| 200034 | 取号token失效       |
| 200036 | 取号失败 |
| 200041 | 应用未授权 |
| 105302 | AppId不在白名单|

</br>

## 4.2. 获取用户信息接口返回码

| 返回码    | 返回码描述                          |
| ------ | ------------------------------ |
| 103000 | 返回成功                           |
| 103101 | 签名错误                           |
| 103103 | 用户不存在                          |
| 103104 | 用户不支持这种登录方式                    |
| 103105 | 密码错误                           |
| 103106 | 用户名错误                          |
| 103107 | 已存在相同的随机数                      |
| 103108 | 短信验证码错误                        |
| 103109 | 短信验证码超时                        |
| 103111 | wap 网关IP错误                     |
| 103112 | 错误的请求                          |
| 103113 | Token内容错误                      |
| 103114 | token验证 KS过期                   |
| 103115 | token验证 KS不存在                  |
| 103116 | token验证 sqn错误                  |
| 103117 | mac异常                          |
| 103118 | sourceid不存在                    |
| 103119 | appid不存在                       |
| 103120 | clientauth不存在                  |
| 103121 | passid不存在                      |
| 103122 | btid不存在                        |
| 103123 | redisinfo不存在                   |
| 103124 | ksnaf校验不一致                     |
| 103125 | 手机号格式错误                        |
| 103127 | 证书验证：版本过期                      |
| 103128 | gba:webservice error           |
| 103129 | 获取短信验证码的msgtype异常              |
| 103130 | 新密码不能与当前密码相同                   |
| 103131 | 密码过于简单                         |
| 103132 | 用户注册失败                         |
| 103133 | sourceid不合法                    |
| 103134 | wap方式手机号码为空                    |
| 103135 | 昵称非法                           |
| 103136 | 邮箱非法                           |
| 103138 | appid已存在                       |
| 103139 | sourceid已存在                    |
| 103200 | 不需要更新ks错误                      |
| 103202 | 缓存用户不存在或者验证短信输入失败次数过多          |
| 103203 | 缓存用户不存在                        |
| 103204 | 缓存随机数不存                        |
| 103205 | 服务器异常                          |
| 103207 | 发送短信失败                         |
| 103210 | 修改密码失败                         |
| 103211 | 其他错误                           |
| 103212 | 校验密码失败                         |
| 103213 | 旧密码失败                          |
| 103214 | 访问缓存或数据库错误                     |
| 103226 | sqn过小或过大                       |
| 103265 | 用户已存在                          |
| 103270 | 随机校验凭证过期                       |
| 103271 | 随机校验凭证错误                       |
| 103272 | 随机校验凭证不存在                      |
| 103000 | 通用SUCCESS                      |
| 103901 | 短信验证码下发次数已达上限                  |
| 103303 | sip 用户未开户（获取应用密码）              |
| 103304 | sip 用户未开户（注销用户）                |
| 103305 | sip 开户用户名错误                    |
| 103306 | sip 用户名不能为空（获取应用密码）            |
| 103307 | sip 用户名不能为空（注销用户）              |
| 103308 | sip 手机号不合法                     |
| 103309 | sip opertype 为空                |
| 103310 | sip sourceid 不存在               |
| 103311 | sip sourceid 不合法               |
| 103312 | sip btid 不存在                   |
| 103313 | sip ks 不存在                     |
| 103314 | sip密码变更失败                      |
| 103315 | sip密码推送失败                      |
| 103399 | sip sys错误                      |
| 103400 | authorization 为空               |
| 103401 | 签名消息为空                         |
| 103402 | 无效的 authWay                    |
| 103404 | 加密失败                           |
| 103405 | 保存数据短信手机号为空                    |
| 103406 | 保存数据短信短信内容为空                   |
| 103407 | 此sourceId, appPackage, sign已注册 |
| 103408 | 此sourceId注册已达上限  99次           |
| 103409 | query 为空                       |
| 103412 | 无效的请求                          |
| 103414 | 参数效验异常                         |
| 103505 | 重放攻击                           |
| 103511 | 源IP不合法                         |
| 103810 | 校验失败，接口token版本不一致              |
| 103811 | token为空                        |
| 103899 | aoi token 其他错误                 |
| 103901 | 短信验证码下发次数已达上限                  |
| 103902 | 凭证校验失败                         |
| 103903 | 调用webservice错误                 |
| 103904 | 配置不存在                          |
| 103905 | 获取手机号码错误                       |
| 103906 | 平台迁移访问错误 - （访问旧地址）             |
| 103911 | 请求过于频繁                         |
| 103920 | 没有存在的版本更新                      |
| 103921 | 下载时间戳超时                        |
| 103922 | 自动升级文件没找到                      |
| 104001 | APPID和APPKEY已存在                |
| 104201 | 凭证已失效或不存在                      |
| 104202 | 短信验证失败过多                       |
| 103412 | 无效的请求                          |
| 103413 | 系统异常                           |
| 103810 | 非新版本token，不予与校验                |
| 105001 | 联通网关取号失败                       |
| 105002 | 移动网关取号失败                       |
| 105003 | 电信网关取号失败                       |
| 105004 | 短信上行ip检测不合法                    |
| 105005 | 短信上行发送信息为空                     |
| 105006 | 手机号码为空                         |
| 105007 | 手机号码格式错误                       |
| 105008 | 短信内容为空                         |
| 105009 | 解析失败                           |
| 105010 | phonescript失效或者非法              |
| 105011 | getPhonescript参数加密的私钥失效或者非法    |
| 105012 | 不支持电信取号                        |
| 105013 | 不支持联通取号                        |
| 105014 | 校验本机号码失败                       |
| 105015 | 校验有数三要素失败                      |
| 105018 | 用户权限不够                         |
| 105019 | 应用未授权                          |

## 4.3. 本机号码校验接口返回码

本返回码表仅针对`本机号码校验接口`使用



| 返回码    | 说明              |
| ------ | --------------- |
| 000    | 是本机号码（纳入计费次数）   |
| 001    | 非本机号码（纳入计费次数）   |
| 002    | 取号失败            |
| 003    | 调用内部token校验接口失败 |
| 004    | 加密手机号码错误        |
| 102    | 参数无效            |
| 124    | 白名单校验失败         |
| 302    | sign校验失败        |
| 303    | 参数解析错误          |
| 606    | 验证Token失败       |
| 999    | 系统异常            |
| 102315 | 次数已用完           |
