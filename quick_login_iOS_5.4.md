# 1. 开发环境配置 

sdk技术问题沟通QQ群：609994083</br>

**注：SDK在获取token过程中，用户手机必须在打开数据网络情况下才能获取成功，纯wifi环境下会自动跳转到SDK的短信验证码页面或短信上行取号（如果有配置）或者返回错误码**</br>

## 1.1. 环境配置及发布

1. 导入统一认证framework，直接将统一认证`TYRZSDK.framework`拖到项目中
2. 在Xcode中找到`TARGETS-->Build Setting-->Linking-->Other Linker Flags`在这选项中需要添加`-ObjC`

</br>

## 1.2. Hello 统一认证 

本节内容主要面向新接入统一认证的开发者，介绍快速集成统一认证的基本服务的方法。

### 1.2.1. 统一认证登录流程

![](image/19.png)

由流程图可知，业务客户端集成SDK后只需要完成2步集成实现登录

    1.	调用登录接口获取token
    2.	携带token请求登录

</br>

### 1.2.2. 统一认证登录集成步骤

**第一步：**

在`appDelegate.m`中的`didFinish`函数中添加初始化代码。初始化代码只需要执行一次就可以。

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
     [TYRZUILogin initializeWithAppId:APPID appKey:APPKEY];
    return YES;
}
```

##1.3. 如何使用单点登录

业务A在使用单点登录到业务B时，需要先使用隐式登录/短信验证码登录获取token，再将token传递到业务B，由业务B带着token去换取业务A使用者的用户手机号码，完成单点登录流程。

#2. SDK方法说明

## 2.1. 初始化appid和appkey

### 2.1.1. 方法描述

**功能**

用于初始化appid、appkey。
此方法必须要在Appdelegate调用

**原型**

```objective-c
+ (void)initializeWithAppId:(NSString *)appId appKey:(NSString *)appKey;
```

</br>

### 2.1.2. 参数说明

**请求参数**


| 参数     | 类型       | 说明       | 是否必填 |
| ------ | -------- | -------- | ---- |
| appId  | NSString | 应用的appid | 是    |
| appKey | NSString | 应用密钥     | 是    |


**响应参数**

无

</br>

### 2.1.3. 示例

**请求示例代码**

```objective-c
[TYRZUILogin initializeWithAppId:APPID appKey:APPKEY];
```

**响应示例代码**

无



</br>

## 2.2. 隐式登录

### 2.2.1. 方法描述

**功能**

本方法用于实现**获取用户信息**功能。使用本方法获取到的token，可通过`获取用户信息接口`交换用户信息。如果业务A需要做单点登录时，也可使用该方法获得token后，再将token传递到业务B，由业务B带着token去换取业务A使用者的用户手机号码，完成单点登录流程</br>

**原型**

`TYRZUILogin -- getTokenImpWithComplete:`

```objective-c

+ (void)getTokenImpWithComplete:(void (^)(id sender))complete

```

### 2.2.2 参数说明

**请求参数**

无

</br>

**响应参数**

| 参数        | 类型     | 说明                                                         | 是否必填   |
| ----------- | -------- | ------------------------------------------------------------ | ---------- |
| resultCode  | NSString | 返回相应的结果码                                             | 是         |
| token       | NSString | 成功时返回：临时凭证，token有效期2min，一次有效，同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 | 成功时必填 |
| authType    | NSString | 认证类型：0:其他;</br>1:WiFi下网关鉴权;</br>2:网关鉴权;</br>3:短信上行鉴权;</br>4: WIFI下网关鉴权复用中间件登录;</br>5: 网关鉴权复用中间件登录;</br>6: 短信上行鉴权复用中间件登录;</br>7:短信验证码登录 | 成功时必填 |
| authTypeDes | NSString | 认证类型描述，对应authType                                   | 成功时必填 |
| OpenID      | NSString | 用户身份唯一标识（参数需在开放平台勾选相关能力后开放，如果勾选了一键登录能力，使用本方法时，不返回OpenID） | 成功返回   |


</br>

### 2.2.3. 示例

**请求示例代码**

```objective-c
    [TYRZUILogin getTokenImpWithComplete:^(id sender) {
    //登录逻辑处理
    }];

```


**响应示例代码**

```
{
    authType = 2;
    authTypeDes = "网关鉴权复用中间件登录";
    desc = "隐式登录成功";
    openId = "007KlCV41GU0WwB_pW6B4sMQL2kYj_bM-P3HEFl-EcLx4cisMYfE";
    resultCode = 103000;
    token = 84840100013302000130030002353004003C4D7A524452455643526A6B774E4549794F5552454D44684540687474703A2F2F3132302E3139372E3233352E32373A383038302F746573742F403031050004018B95F80600063130303030300700206233363839343039333639373465393639643461386334363233306166643532FF0020F7D5F4D7E49BE6C59184B62341F85929C56C9D5CF1900526F940F75B3FD6E821;
}

```

</br>


## 2.3. 获取短信验证码

### 2.3.1. 方法描述

**功能**

该方法用于短信验证码登录时，通过输入手机号码来获取对应的短信验证码

</br>

**原型**

`TYRZUILogin -- getSmsCodeWithPhoneNumber `

```objective-c

+ (void)getSmsCodeWithPhoneNumber:(NSString *)phoneNumber complete:(void(^)(id sender))complete;

```

</br>

### 2.3.2. 参数说明

**请求参数**

| 参数     | 类型   | 说明                                       | 是否必填   |
| ------ | ---- | ---------------------------------------- | ------ |
|phoneNumber |NSString | 用户的手机号码 | 是</br> |

**响应参数**

| 参数       | 类型     | 说明             | 是否必填   |
| ---------- | -------- | ---------------- | ---------- |
| resultCode | NSString | 返回相应的结果码 | 是         |
| desc       | NSString | 返回结果描述     | 成功时必填 |
</br>

### 2.3.3. 示例

**请求示例代码**

```objective-c
    [TYRZUILogin getSmsCodeWithPhoneNumber:phoneNum complete:^(id sender) {
    }];

```


**响应示例代码**

```
{
    desc = "获取短信验证码成功";
    resultCode = 103000;
}

```

</br>

## 2.4. 短信验证码登录

### 2.4.1. 方法描述

**功能**

该方法用于短信验证码登录，用户应先调用获取短信验证码接口，等待手机接收到短信验证码，然后再调用此方法将短信验证码和手机号码作为参数传进来，即可完成短信验证码的登录。

</br>

**原型**

`TYRZUILogin -- getTokenBySmsCodewithPhoneNumber `

```objective-c

+ (void)getTokenBySmsCodewithPhoneNumber:(NSString *)phoneNumber smsCode:(NSString *)smsCode  complete:(void(^)(id sender))complete;


```

</br>

### 2.4.2. 参数说明

**请求参数**

| 参数     | 类型   | 说明                                       | 是否必填   |
| ------ | ---- | ---------------------------------------- | ------ |
|phoneNumber |NSString | 用户的手机号码 | 是</br> |
|smsCode |NSString | 手机验证码 | 是</br> |

**响应参数**

| 参数        | 类型     | 说明                                                         | 是否必填   |
| ----------- | -------- | ------------------------------------------------------------ | ---------- |
| resultCode  | NSString | 返回相应的结果码                                             | 是         |
| token       | NSString | 成功时返回：临时凭证，token有效期2min，一次有效，同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 | 成功时必填 |
| authType    | NSString | 认证类型：0:其他;</br>1:WiFi下网关鉴权;</br>2:网关鉴权;</br>3:短信上行鉴权;</br>4: WIFI下网关鉴权复用中间件登录;</br>5: 网关鉴权复用中间件登录;</br>6: 短信上行鉴权复用中间件登录;</br>7:短信验证码登录 | 成功时必填 |
| authTypeDes | NSString | 认证类型描述，对应authType                                   | 成功时必填 |
| OpenID      | NSString | 用户身份唯一标识（参数需在开放平台勾选相关能力后开放，如果勾选了一键登录能力，使用本方法时，不返回OpenID） | 成功返回   |


</br>

### 2.4.3. 示例

**请求示例代码**

```objective-c
[TYRZUILogin getTokenBySmsCodewithPhoneNumber:phoneNum smsCode:smsCode complete:^(id sender) {
    }];

```


**响应示例代码**

```
{
    authType = 7;
    authTypeDes = "短信验证码登录";
    openId = "TddPLEIu1bG5jaPLfc1IxuOqDnZJa7dXhAnIcUOG_sqymDFoBowA";
    resultCode = 103000;
    token = 84840100013602003A4E444A474D6A424451305135515559354D5468434D304A4440687474703A2F2F3132302E3139372E3233352E32373A383038302F72732F4030320300040555189D040012383030313230313730383036313030393533050020623336383934303933363937346539363964346138633436323330616664353206000132070003323030FF002096E3021565A620381ACBCECB7DD5C90C0689D9BC1CA2DD8E3E5119AF7BD796A7;
}

```

</br>

## 2.5. 是否中间件优先登录

### 2.5.1. 方法描述

**功能**

该方法控制登录时是否优先使用中间件登录，默认为YES。如果传入YES，则每次登录时都会先检查是否已经有中间件，如果有则使用上次登录的中间件登录，否则按照无中间件正常流程登录。如果传入NO，则不使用中间件登录，不会使用缓存中间件信息。

</br>

***中间件是什么？***

```
为了减少应用在登录或做单点登录时，频繁做网关取号操作，减少取号失败概率，应用首次登录时，SDK会将用户加密的登录信息保存在本地，称为中间件；
中间件会在用户每天首次登录/单点登录时会去访问服务端判断是否失效；
服务端中间件有效期30天；
服务端中间件失效后，用户登录或使用单点登录时，将需要重新走网关取号流程
```

</br>

**原型**

`TYRZUILogin -- middleWarePriorityShouldFirst `

```objective-c

+ (void)middleWarePriorityShouldFirst:(BOOL)flag;


```

</br>

### 2.5.2. 参数说明

**请求参数**

| 参数     | 类型   | 说明                                       | 是否必填   |
| ------ | ---- | ---------------------------------------- | ------ |
|flag |BOOL | 是否中间件优先登录 | 是</br> |

**响应参数**

无

</br>

### 2.5.3. 示例

**请求示例代码**

```objective-c
    [TYRZUILogin middleWarePriorityShouldFirst:YES];

```


**响应示例代码**

无

</br>

## 2.6. 清除中间件缓存信息

### 2.6.1. 方法描述

**功能**

该方法用于清除中间件缓存信息，当用户不想使用中间件信息进行登录或者用户换了另外的sim卡的时候，可以使用该方法将中间件信息清除，然后再进行登录操作。中间件清除后，用户再次登录时，将走网关/短信逻辑重新取号。

**原型**

`TYRZUILogin -- cleanSSOSync `

```objective-c

+ (BOOL)cleanSSOSync;

```

</br>

### 2.6.2. 参数说明

**请求参数**

无

**响应参数**


| 参数     | 类型   | 说明                                       | 是否必填   |
| ------ | ---- | ---------------------------------------- | ------ |
|无 |BOOL | 清除中间件信息是否成功 | 是</br> |


</br>

### 2.6.3. 示例

**请求示例代码**

```objective-c
    [TYRZUILogin cleanSSOSync];

```


**响应示例代码**

无

</br>

## 2.7. 获取SDK版本号

供接入方区分SDK的版本号，便于反馈SDK的相关信息

### 2.7.1. 方法说明

**功能**

获取SDK版本号

**原型**

``` objective-c

@property (nonatomic,class,readonly) NSString *sdkVersion;

```

**调用示例**

``` objective-c

NSString *sdkVersion = TYRZUILogin.sdkVersion;

```

<div STYLE="page-break-after: always;"></div>

# 3. 平台接口说明

## 3.1. 获取用户信息接口

业务平台或服务端携带用户授权成功后的token来调用统一认证服务端获取用户手机号码等信息。**注：本接口仅适用于5.4.0及以上版本SDK**

### 3.1.1. 业务流程

SDK在获取token过程中，用户手机必须在打开数据网络情况下才能获取成功，纯wifi环境下会自动跳转到SDK的短信验证码页面（如果有配置）或者返回错误码

![](image/19.png)

### 3.1.2. 接口说明

**生产环境请求地址：**

https://wap.cmpassport.com:8443/uniapi/uniTokenValidate 

http://wap.cmpassport.com:8080/uniapi/uniTokenValidate 

**联调环境请求地址：**http://218.205.115.220:10085/test/api/uniTokenValidate

**协议：** HTTPS+ application/json

**请求方法：** POST+json

</br>

### 3.1.3. 参数说明

**请求参数**

| 参数名称            | 约束 | 层级 | 参数类型 | 说明                                                         |
| ------------------- | ---- | ---- | -------- | ------------------------------------------------------------ |
| header              | 必选 | 1    |          |                                                              |
| version             | 必选 | 2    | string   | 填1.0                                                        |
| msgid               | 必选 | 2    | string   | 标识请求的随机数即可(1-36位)                                 |
| systemtime          | 必选 | 2    | string   | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| id                  | 必选 | 2    | string   | 融合版sdk token校验时，该字段填appId                         |
| idtype              | 必选 | 2    | string   | 融合版sdk token校验时，该字段填"1"                           |
| apptype             | 必选 | 2    | string   | app类型：1:BOSS</br> 2:web</br> 3:wap</br> 4:pc客户端</br> 5:手机客户端 |
| userip              | 可选 | 2    | string   | 客户端用户来源ip                                             |
| message             | 可选 | 2    | string   | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandparams        | 可选 | 2    | json     | 业务扩展参数，json格式。                                     |
| sign                | 必选 | 2    | string   | 当**encryptionalgorithm≠"RSA"**时，sign = MD5（appid + version + msgid + systemtime + strictcheck + token + appkey)（注：“+”号为合并意思，不包含在被加密的字符串中），输出32位大写字母；</br>当**encryptionalgorithm="RSA"**，业务端RSA私钥签名（appid+token）, 服务端使用业务端提供的公钥验证签名（公钥可以在开发者社区配置）。 |
| encryptionalgorithm | 可选 |      |          | 开发者如果需要使用非对称加密算法时，填写“RSA”。（当该值不设置为“RSA”时，执行MD5签名校验） |
| body                | 必选 | 1    |          |                                                              |
| token               | 必选 | 2    | string   | 需要解析的凭证值。                                           |

</br>

**响应参数**

| 参数名称            | 约束 | 层级 | 参数类型 | 说明                                                         |
| ------------------- | ---- | ---- | -------- | ------------------------------------------------------------ |
| header              | 必选 | 1    |          |                                                              |
| version             | 必选 | 2    | string   | 1.0有升级时调整                                              |
| inresponseto        | 必选 | 2    | string   | 对应的请求消息中的msgid                                      |
| systemtime          | 必选 | 2    | string   | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| resultcode          | 必选 | 2    | string   | 返回码，返回码对应说明见附录A                                |
| body                | 必选 | 1    |          |                                                              |
| openid              | 可选 | 2    | string   | 成功时返回：用户身份唯一标识                                 |
| msisdntype          | 可选 | 2    | string   | 手机号码的归属运营商：</br> 0：中国移动</br>1：中国电信</br>2：中国联通</br> 99：未知的异网手机号码 |
| usessionid          | 可选 | 2    | string   | 暂忽略                                                       |
| passid              | 必选 | 2    | string   | 用户统一账号的系统标识                                       |
| loginidtype         | 可选 | 2    | string   | 登录使用的用户标识：</br>0：手机号码</br>1：邮箱             |
| authtime            | 可选 | 2    | string   | 统一认证平台认证用户的时间                                   |
| lastactivetime      | 可选 | 2    | string   | 暂无                                                         |
| authtype            | 可选 | 2    | string   | 认证方式                                                     |
| relateToAndPassport | 可选 | 2    | string   | 是否已经关联到统一账号，暂无用处                             |
| msisdn              | 可选 | 2    | string   | 手机号                                                       |

</br>

### 3.1.4. 示例

**请求示例**

```
{
	"body": {
		"token": "84840100013602003A4D4456474F4442464E5463784F555244517A42434D54564540687474703A2F2F3132302E3139372E3233352E32373A383038302F72732F40303103000402C1FE800400123830303132303137313031373130333137330500206233363839343039333639373465393639643461386334363233306166643532060001300700023530FF0020321FDF1384B40B343B6D7ADDC69AF3D6C87DD82DCB98869F5A764A522E86067B"
	},
	"header": {
		"msgid": "c1eeff4eeb3b4dc1897f9013673d5eb3",
		"idtype": "1",
		"id": "300011012484",
		"apptype": "5",
		"version": "1.0",
		"sign": "4368018959d0e51b8aafd26193c2e8b4",
		"systemtime": "20180227193858817"
	}
}
```

**响应示例**

```
{
	"body": {
		"msisdntype": "1",
		"usessionid": "QzJCQ0VENzhDRjNFOTY0NDNC@http://120.197.235.27:8080/rs/@01",
		"passid": "1378119606",
		"loginidtype": "0",
		"authtime": "2018-02-27 20:13:53",
		"msisdn": "13800138000",
		"lastactivetime": "",
		"authtype": "WAPGW",
		"relateToAndPassport": "1"
	},
	"header": {
		"inresponseto": "807806355a9e47678a26e1735920733d",
		"resultcode": "103000",
		"systemtime": "20180227201357620",
		"version": "1.0"
	}
}
```

<div STYLE="page-break-after: always;"></div>

# 4. 返回码说明

## 4.1. SDK返回码

使用SDK时，SDK会在认证结束后将结果回调给开发者，其中结果为JSONObject对象，其中resultCode为结果响应码，103000代表成功，其他为失败。成功时在根据token字段取出身份标识。失败时根据resultCode定位失败原因。

| 错误码 | 返回错误码的描述                                             |
| ------ | ------------------------------------------------------------ |
| 103000 | 通用 成功返回                                                |
| 102000 | 旧版（5.0.X）获取token成功返回，新版已经不使用此返回码，改为103000 |
| 102101 | 无网络                                                       |
| 102102 | 网络异常                                                     |
| 102103 | 非移动网络                                                   |
| 102109 | 网络错误                                                     |
| 102121 | 用户取消登录                                                 |
| 102201 | 自动登陆失败，用户选择自定义界面时，需要继续处理手动登陆流程，比如弹出登陆界面。 |
| 102202 | APP签名验证不通过                                            |
| 102203 | 接口入参错误                                                 |
| 102204 | 正在进行GetToken动作，请稍后                                 |
| 102205 | 当前环境不支持指定的登陆方式                                 |
| 102206 | 选择用户登陆时，本地不存在指定的用户                         |
| 102207 | 获取的中间件值错误                                           |
| 102208 | 参数错误                                                     |
| 102209 | 没有sim卡                                                    |
| 102210 | 不支持短信发送                                               |
| 102211 | 短信验证码验证成功后返回随机码为空                           |
| 102222 | http响应头中没有结果码                                       |
| 102223 | 数据解析异常                                                 |
| 102299 | other failed                                                 |
| 102302 | 调用service超时                                              |
| 102303 | 用户名为空                                                   |
| 102304 | 密码为空                                                     |
| 102305 | 验证码获得成功                                               |
| 102306 | 验证码获得失败                                               |
| 102307 | 用户名格式错误                                               |
| 102308 | 验证码为空                                                   |
| 102309 | 验证码格式错误                                               |
| 102310 | 用户名和验证码格式错误                                       |
| 102311 | 密码格式错误                                                 |
| 102312 | 用户名和密码格式错误                                         |
| 102506 | 请求出错                                                     |
| 102507 | 请求超时                                                     |
| 102508 | 数据网络切换失败                                             |
| 102509 | 未知错误，一般出现在线程捕获异常，请配合异常打印分析         |
| 200001 | imsi为空，跳到短信验证码登录                                 |
| 200002 | imsi为空，没有短信验证码登录功能                             |
| 200003 | 复用中间件首次登录                                           |
| 200004 | 复用中间件二次登录                                           |
| 200005 | 用户未授权 READ_PHONE_STATE                                  |
| 200006 | 用户未授权 SEND_SMS                                          |
| 200007 | 不支持的认证方式跳到短信验证码登录                           |
| 200008 | 不支持的认证方式没有短信验证码登录功能                       |
| 200009 | 应用合法性校验失败                                           |
| 200010 | 当前网络环境下不支持的认证方式                               |
| 200012 | 取号失败，跳短信验证码登录                                   |
| 200013 | 短信上行发送短信失败                                         |
| 200014 | 手机号码格式错误                                             |
| 200015 | 短信验证码格式错误                                           |
| 200016 | 更新KS失败                                                   |
| 200017 | 非移动卡不支持短信上行                                       |
| 200018 | 不支持网关登录                                               |
| 200019 | 不支持短信验证码登录                                         |
| 200020 | 用户取消认证                                                 |
| 200021 | 数据解析异常                                                 |
| 200022 | 无网络状态                                                   |
| 200023 | 请求超时                                                     |
| 200024 | 数据网络切换失败                                             |
| 200025 | 未知错误一般出现在线程捕获异常，请配合异常打印分析           |
| 200026 | 输入参数错误                                                 |
| 200027 | 预取号只开启WIFI                                             |
| 200028 | 网络请求出错                                                 |
| 200029 | 请求出错,上次请求未完成                                      |
| 200030 | 没有初始化参数                                               |
| 200031 | 生成token失败                                                |
| 200032 | KS缓存不存在                                                 |
| 200033 | 复用中间件获取Token失败                                      |
| 200034 | 预取号token失效                                              |

</br>

## 4.2. 获取用户信息接口返回码

| 平台错误码 | 描述                                       |
| ---------- | ------------------------------------------ |
| 103101     | 签名错误(token信息不对)                    |
| 103012     | 包签名错误                                 |
| 103103     | 用户不存在                                 |
| 103104     | 用户不支持这种登录方式                     |
| 103105     | 密码错误                                   |
| 103106     | 用户名错误                                 |
| 103107     | 已存在相同的随机数                         |
| 103108     | 短信验证码错误                             |
| 103109     | 短信验证码超时                             |
| 103111     | wap 网关IP错误                             |
| 103112     | 错误的请求                                 |
| 103113     | Token内容错误                              |
| 103114     | token验证 KS过期                           |
| 103115     | token验证 KS不存在                         |
| 103116     | token验证 sqn错误                          |
| 103117     | mac异常                                    |
| 103118     | sourceid不存在                             |
| 103119     | appid不存在                                |
| 103120     | clientauth不存在                           |
| 103121     | passid不存在                               |
| 103122     | btid不存在                                 |
| 103123     | redisinfo不存在                            |
| 103124     | ksnaf校验不一致                            |
| 103125     | 手机号格式错误                             |
| 103127     | 证书验证：版本过期                         |
| 103128     | gba:webservice error                       |
| 103129     | 获取短信验证码的msgtype异常                |
| 103130     | 新密码不能与当前密码相同                   |
| 103131     | 密码过于简单                               |
| 103132     | 用户注册失败                               |
| 103133     | sourceid不合法                             |
| 103134     | wap方式手机号码为空                        |
| 103135     | 昵称非法                                   |
| 103136     | 邮箱非法                                   |
| 103138     | appid已存在                                |
| 103139     | sourceid已存在                             |
| 103200     | 不需要更新ks错误                           |
| 103202     | 缓存用户不存在或者验证短信输入失败次数过多 |
| 103203     | 缓存用户不存在                             |
| 103204     | 缓存随机数不存                             |
| 103205     | 服务器异常                                 |
| 103207     | 发送短信失败                               |
| 103210     | 修改密码失败                               |
| 103211     | 其他错误                                   |
| 103212     | 校验密码失败                               |
| 103213     | 旧密码失败                                 |
| 103214     | 访问缓存或数据库错误                       |
| 103226     | sqn过小或过大                              |
| 103265     | 用户已存在                                 |
| 103270     | 随机校验凭证过期                           |
| 103271     | 随机校验凭证错误                           |
| 103272     | 随机校验凭证不存在                         |
| 103000     | 通用SUCCESS                                |
| 103901     | 短信验证码下发次数已达上限                 |
| 103303     | sip 用户未开户（获取应用密码）             |
| 103304     | sip 用户未开户（注销用户）                 |
| 103305     | sip 开户用户名错误                         |
| 103306     | sip 用户名不能为空（获取应用密码）         |
| 103307     | sip 用户名不能为空（注销用户）             |
| 103308     | sip 手机号不合法                           |
| 103309     | sip opertype 为空                          |
| 103310     | sip sourceid 不存在                        |
| 103311     | sip sourceid 不合法                        |
| 103312     | sip btid 不存在                            |
| 103313     | sip ks 不存在                              |
| 103314     | sip密码变更失败                            |
| 103315     | sip密码推送失败                            |
| 103399     | sip sys错误                                |
| 103400     | authorization 为空                         |
| 103401     | 签名消息为空                               |
| 103402     | 无效的 authWay                             |
| 103404     | 加密失败                                   |
| 103405     | 保存数据短信手机号为空                     |
| 103406     | 保存数据短信短信内容为空                   |
| 103407     | 此sourceId, appPackage, sign已注册         |
| 103408     | 此sourceId注册已达上限  99次               |
| 103409     | query 为空                                 |
| 103412     | 无效的请求                                 |
| 103414     | 参数效验异常                               |
| 103505     | 重放攻击                                   |
| 103511     | 源IP不合法                                 |
| 103810     | 校验失败，接口token版本不一致              |
| 103811     | token为空                                  |
| 103899     | aoi token 其他错误                         |
| 103901     | 短信验证码下发次数已达上限                 |
| 103902     | 凭证校验失败                               |
| 103903     | 调用webservice错误                         |
| 103904     | 配置不存在                                 |
| 103905     | 获取手机号码错误                           |
| 103906     | 平台迁移访问错误 - （访问旧地址）          |
| 103911     | 请求过于频繁                               |
| 103920     | 没有存在的版本更新                         |
| 103921     | 下载时间戳超时                             |
| 103922     | 自动升级文件没找到                         |
| 104001     | APPID和APPKEY已存在                        |
| 104201     | 凭证已失效或不存在                         |
| 104202     | 短信验证失败过多                           |
| 103412     | 无效的请求                                 |
| 103413     | 系统异常                                   |
| 103810     | 非新版本token，不予与校验                  |
| 105001     | 联通网关取号失败                           |
| 105002     | 移动网关取号失败                           |
| 105003     | 电信网关取号失败                           |
| 105004     | 短信上行ip检测不合法                       |
| 105005     | 短信上行发送信息为空                       |
| 105006     | 手机号码为空                               |
| 105007     | 手机号码格式错误                           |
| 105008     | 短信内容为空                               |
| 105009     | 解析失败                                   |
| 105010     | phonescript失效或者非法                    |
| 105011     | getPhonescript参数加密的私钥失效或者非法   |
| 105012     | 不支持电信取号                             |
| 105013     | 不支持联通取号                             |
| 105014     | 校验本机号码失败                           |
| 105015     | 校验有数三要素失败                         |
| 105018     | 用户权限不足                               |
| 105019     | 应用未授权                                 |
| 105017     | 短信随机数校验非法失败                     |
| 105016     | 达到使用次数限制                           |