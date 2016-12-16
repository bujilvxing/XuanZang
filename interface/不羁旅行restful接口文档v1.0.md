#不羁旅行restful接口文档v1.0
少量的接口(查询接口)，无需登录，游客用户即可操作。大部分接口(比如：与用户相关的接口)需要登录才能操作。
###header
HTTP请求需要设置以下Header:

- Accept:application/vnd.bjlx.v1+json。随着系统的开发，接口可能出现不同的版本，因此在这个header中携带接口版本信息，v1表示第一个版本。
- Accept-Encoding:gzip。可支持gzip压缩格式，减小数据传输。
- key:不羁旅行自定义Header。若用户登录，则需要指定该Header，用于获取用户的信息以及部分接口的登录权限验证。

###数据格式
所有发送和接收的数据都是通过json来表达，接口域名为api.bujilvxing.com。

###http状态码与系统错误码

**http状态码**

默认情况下，成功的RESTful API请求，应该返回状态码200。一些涉及到创建资源的接口，比如新建用户等，在成功时将返回状态码201。

- 400 Bad Request: API请求出现了语法或格式方面的错误。比如：在POST，PUT等情况下，提交的Json body无法解析。
- 401 Unauthorized: 在调用部分需要用户身份认证的接口时（比如添加好友、修改密码等），没有提供认证信息。
- 409 Conflict:比如新建用户时用户已经存在
- 422: Unprocessable Entity: API请求有效，且服务器可以理解。但提供的参数无效。比如：在POST，PUT等情况下，API请求body中提供了错误的字段。
- 403 Forbidden: 因为权限问题，客户端对资源的访问被拒绝。比如以用户A的身份，试图修改用户B的联系人列表等操作。

**系统错误码**

通常的正常返回结果

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"field":value
			}
		]
	}

返回的错误结果

	{
		"code":100101,
		"msg":"用户已存在",
		"timestamp":1425225600000,
	}

其中code是系统的错误码，0表示正常，其他的都是错误码。msg，正常情况返回"success"，错误时，返回错误信息描述，timestamp是接口返回的时间戳，正常的接口请求会有一个result字段，错误的情况下是没有这个字段的。下面对错误码进行设计。

错误码由6位数字构成，每1位的含义如下：

- 第1位：表示产品编号
- 第2~4位：表示接口编号
- 第5~6位：表示指定接口的错误编号

举例：100101表示第一个产品的第一个接口的第一个错误编号，这样可以根据错误码，定位到是哪个接口出现了什么问题。

###HTTP 方法

接口中，使用到了以下方法。

- GET: 获取资源信息。
- POST: 新建一个资源。
- PATCH: 更新资源的部分字段。
- PUT: 替换某个资源。
- DELETE: 删除资源。

###null值的处理

当某个字段的数据不存在时，该字段被设置为null。在返回的JSON数据中，对null值的处理方式，根据该字段的类型而有所不同：

- 字符串型：如果一个字符串类型的值为null，将被表示为""。
- 数字类型：如果一个数字类型的值为null，将被表示为null。
- 布尔类型：如果一个布尔类型的值为null，将被表示为null。
- 字典类型：如果一个字典类型的值为null，将被表示为{}。
- 数组类型：如果一个数组类型的值为null，将被表示为[]。

###对资源的搜索

大部分情况下，资源搜索都遵循这样的模式。举例，为了搜索景点，API如下：

GET /app/poi/viewspots?query=北京+鼓楼&sortby=rating&sort=desc

- URI: 就是资源集合本身，如上例中的/app/poi/viewspots，app表示这个是移动端app的请求，没有app表示是pc端web的请求。
- 关键字分隔符: +
- Query strings
	- query: 搜索的关键词列表，使用+进行分割（类似Google等）
    - sortby: 指定对搜索结果进行排序时所参照的字段
    - sort: 取值可以是asc或desc，分别表示升序或降序


###资源的主键

绝大多数资源都有名为id的键。这是该资源的主键，为ObjectId类型。


###图片Image

该类型表示一个图像资源。其核心数据是该图片对应的URL。此外，还有以下辅助字段：

- width: 图像的宽度
- height: 图像的高度
- originUrl: 原图链接（可选）

该类型的的JSON表示如下：

	{
	    "url": "http://images.lvxingpai.com/foo/bar.jpg",
	    "width": 800,
	    "height": 600
	}

###时间戳

长整形，epoch到现在为止经历的毫秒数。


###坐标

该类型表示一个经纬度坐标，其字段设计基本参照简单的GeoJSON http://geojson.org/：

	{
    	"coordinates": [ 120.2812, 42.382924 ]
	}

###翻页机制

通过page和pageSize实现翻页控制，page是从0开始的，pageSize默认是100。类似下面这样的API：

GET /app/search?query=北京&page=2&pageSize=50

>返回信息一般有两种形式，一种是资源摘要，比如返回一些列表的信息，此时只含有部分字段，另一种是资源详情。

#接口详细说明

删除资源
>删除接口时，无需真正删除，只需更新资源状态即可，例如删除帖子，实际上是将帖子状态标记为删除状态。

路径
>路径中的{}表示一个变量，例如：/app/users/{userId}，其中userId是用户id，假设用户id为10001，那么路径即为/app/users/10001

***
#登录注册模块
###发送验证码1001
- Path:/app/validationcodes
- Request Method:POST
- Request Headers:无
- Query String:空
- Request Body
> 参数说明

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
account|String|是|无|可以是手机号，也可以是邮箱号
action|Integer|是|无|1表示新用户注册；2表示用户绑定手机号；3表示用户找回密码；4表示用户绑定邮箱
> 示例

	{
	    "account":"13811111111",
	    "action":1
	}

- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|验证码主键
tel|ObjectNode|否|无|电话号码对象
dialCode|Integer|否|86|国家代号
number|String|否|无|手机号
code|String|是|无|验证码
action|Integer|是|无|验证码用途
failCnt|Integer|是|无|验证码验证错误次数
createTime|Long|是|无|创建时间
expireTime|Long|是|无|验证码过期时间
lastSendTime|Long|是|无|上一次发送验证码时间
resendTime|Long|是|无|下一次允许发送验证码时间

> 示例

	{
	    "timestamp": 1477560552146,
	    "code": 0,
	    "result": {
	        "id": "5811a7518edd1f489cd497fe",
	        "tel": {
	            "dialCode": 86,
	            "number": "15300167102"
	        },
	        "code": "714487",
	        "action": 1,
	        "failCnt": 0,
	        "createTime": 1477560552144,
	        "expireTime": 1477560912144,
	        "lastSendTime": 1477560552144,
	        "resendTime": 1477560612144
	    }
	}

错误码|描述|原因
--|--|--
100101|参数账户为空|少传了一个account参数
100102|账户格式不正确|输入了错误的邮箱号或者错误的手机号
100103|参数action为空|少传了一个action参数
100104|参数action的值不合法|action的值只能是1~4
100105|用户已存在|用户已经存在，换一个账号注册
100106|账户不存在|重置密码时接收密码的账户必须是已注册的账户
100107|手机号已存在|手机号已存在，先解除绑定
100108|邮箱已存在|邮箱已存在，先解除绑定
100109|请求过于频繁，请稍后再试|不允许1分钟之内连续请求
100110|请求次数过多|验证码请求次数过多

###检验验证码1002
- Path:/app/tokens
- Request Method:POST
- Request Headers:无
- Query String:空
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
account|String|是|无|可以是手机号，也可以是邮箱号
code|String|是|无|验证码

	{
		"account":"13811111111",   // 可以是手机号，也可以是邮箱号
		"code":"022321"
	}

- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
token|String|是|无|合法的令牌，带合法的令牌请求视为有效，六分钟有效期，可以多次使用

> 示例

	{
	    "timestamp": 1477588349829,
	    "code": 0,
	    "result": {
	        "token": "token::6fae13af204249898754eeca0572d7a3"
	    }
	}

错误码|描述|原因
--|--|--
100201|参数账户为空|没有传account参数
100202|账户格式不正确|输入了错误的邮箱号或者手机号
100203|参数验证码为空|没有传code参数
100204|验证码不合法|验证码不是6位的数字
100205|验证失败|1、验证码错误；2、验证码过期；3、验证码使用过；4、验证码验证错误次数超过10次

###注册1003
- Path:/app/users
- Request Method:POST
- Request Headers:无
- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
account|String|是|无|可以是手机号，也可以是邮箱号
token|String|是|无|令牌
password|String|是|无|密码
promotionCodeSize|Integer|否|8|邀请码长度

	{
		"token":"token::eddf6dce4dbd41b29893d0d3a5b7bcfa",
		"account":"13811111111",   // 可以是手机号，也可以是邮箱号
		"password":"312315",
		"promotionCodeSize":8
	}

- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是||用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
gender|Integer|是|1|1、未选择，2、男 3、女
promotionCode|String|是|无|默认6位的数字或者大写字母，可以自定义长度
loginStatus|Boolean|是|false|登录状态
loginTime|Long|否|0|登录时间
logoutTime|Long|否|0|登出时间
version|Integer|否|0|登录设备版本
roles|Array[Integer]|是|[]|角色
level|Integer|是|1|用户等级
soundNotify|Boolean|是|true|是否声音提醒
vibrateNotify|Boolean|是|true|是否振动提醒
backGround|Object|是||用户背景图片
createTime|Long|是|0|用户创建时间
updateTime|Long|是|0|用户更新时间

> 示例

	{
	    "timestamp": 1481466395798,
	    "code": 0,
	    "result": {
	        "id": "584d621b2395512a982104b5",
	        "tel": {
	            "dialCode": 86,
	            "number": "15300167102"
	        },
	        "userId": 1,
	        "nickName": "不羁1",
	        "avatar": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "KJBPY5AJ",
	        "loginStatus": false,
	        "loginTime": 0,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [],
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 1481466395602,
	        "updateTime": 1481466395602
	    }
	}

错误码|描述|原因
--|--|--
100301|参数账户为空|没有传account参数
100302|账户格式不正确|账户不是合法的手机号或者合法的邮箱号
100303|参数密码为空|没有传password参数
100304|参数令牌为空|没有传token参数
100305|用户已存在|账号已经注册过了，需要更换账号
100306|令牌不合法|token不合法

###登录1004
- Path:/app/login
- Request Method:POST
- Request Headers:无
- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
account|String|是|无|可以是手机号，也可以是邮箱号
password|String|是|无|密码
clientId|String|是|无|个推的clientId，消息推送时使用

	{
		"account":"13811111111",
		"password":"ABCabc123",
		"clientId":"da12a231ce4278678234ca3243b432"
	}

- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是||用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
gender|Integer|是|1|1、未选择，2、男 3、女
promotionCode|String|是|无|默认6位的数字或者大写字母，可以自定义长度
loginStatus|Boolean|是|false|登录状态
loginTime|Long|否|0|登录时间
logoutTime|Long|否|0|登出时间
version|Integer|否|0|登录设备版本
roles|Array[Integer]|是|[]|角色
level|Integer|是|1|用户等级
soundNotify|Boolean|是|true|是否声音提醒
vibrateNotify|Boolean|是|true|是否振动提醒
backGround|Object|是||用户背景图片
createTime|Long|是|0|用户创建时间
updateTime|Long|是|0|用户更新时间
key|String|是|无|授权码

> 示例

	{
	    "timestamp": 1481467046953,
	    "code": 0,
	    "result": {
	        "id": "584d621b2395512a982104b5",
	        "tel": {
	            "dialCode": 86,
	            "number": "13811111111"
	        },
	        "userId": 100001,
	        "nickName": "不羁100001",
	        "avatar": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "KJBPY5AJ",
	        "loginStatus": true,
	        "loginTime": 1481466977003,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [1],
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 1481466395602,
	        "updateTime": 1481466395602,
	        "key": "cb591a364b833a77555d1aeb33e06be139254bd1e2b5e3ec5e0769ff74aef7e6"
	    }
	}

错误码|描述|原因
--|--|--
100401|参数账户为空|没有传account参数
100402|参数密码为空|没有传password参数
100403|参数clientId为空|没有传clientId参数
100404|账户格式不正确|账户不是合法的手机号或者合法的邮箱号
100405|密码不正确|密码输入有误
100406|用户不存在|账户不合法

###第三方登录1005
- Path:/app/oauthlogin
- Request Method:POST
- Request Headers:无
- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
provider|String|是|无|第三方平台名称
oauthId|String|是|无|第三方平台的用户id
nickName|String|是|无|第三方平台的用户昵称
avatar|String|是|无|第三方平台的用户头像
token|String|是|无|第三方平台的用户令牌
clientId|String|是|无|个推的clientId，消息推送时使用

	{
	    "provider":"weixin",
	    "oauthId":"xiaozhi",
	    "nickName":"逍遥",
	    "token":"dadsadadsadasdasdadadadasd",
	    "avatar": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	    "clientId":"da12a231ce4278678234ca3243b432"
	}

- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是||用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
gender|Integer|是|1|1、未选择，2、男 3、女
promotionCode|String|是|无|默认6位的数字或者大写字母，可以自定义长度
loginStatus|Boolean|是|false|登录状态
loginTime|Long|否|0|登录时间
logoutTime|Long|否|0|登出时间
version|Integer|否|0|登录设备版本
roles|Array[Integer]|是|[]|角色
qq|Object|是|{}|第三方平台的信息
provider|String|是|无|第三方平台的名称
oauthId|String|是|无|第三方平台的用户id
nickName|String|否|不羁+userId|第三方平台的用户昵称
avatar|String|否|默认用户头像|第三方平台的用户头像
token|String|是|无|第三方平台的用户令牌
level|Integer|是|1|用户等级
soundNotify|Boolean|是|true|是否声音提醒
vibrateNotify|Boolean|是|true|是否振动提醒
backGround|Object|是||用户背景图片
createTime|Long|是|0|用户创建时间
updateTime|Long|是|0|用户更新时间
key|String|是|无|授权码

> 示例

	{
	    "timestamp": 1481467787272,
	    "code": 0,
	    "result": {
	        "id": "584d678b2395512a982104ba",
	        "userId": 100002,
	        "nickName": "逍遥",
	        "avatar": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 0,
	            "height": 0,
	            "fmt": ""
	        },
	        "gender": 1,
	        "promotionCode": "CC278J84",
	        "loginStatus": false,
	        "loginTime": 0,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [],
	        "weixin": {
	            "provider": "weixin",
	            "oauthId": "xiaozhi",
	            "nickName": "逍遥",
	            "avatar": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "token": "dadsadadsadasdasdadadadasd"
	        },
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 1481467787268,
	        "updateTime": 1481467787268,
	        "key": "0d1badffb1b43e404de6b1307eea4324057d31f555e1a94fbe868403bc4caf50"
	    }
	}

错误码|描述|原因
--|--|--
100501|参数provider为空|没有传provider参数
100502|参数oauthId为空|没有传oauthId参数
100503|参数token为空|没有传token参数
100504|参数clientId为空|没有传clientId参数
100505|参数provider不合法|provider取值只能是"qq","weixin","sina"

###退出登录1006
- Path:/app/logout
- Request Method:POST
- Request Headers

		"key":"bf997008dc3e41953db8d9af8580f0ca4489c53d94d02851ed5e5961804cd96b"
		"userId":100001

- Query String:无
- Request Body:无
- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
100601|用户未登录|没有传key或者userId参数；或者key、userId有误

***
#用户模块
###重置密码1007
- Path:/app/users/password
- Request Method:PUT
- Request Headers:无
- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
account|String|是|无|可以是手机号，也可以是邮箱号
newPassword|String|是|无|新密码
token|String|是|无|令牌

	{
		"account":"13811111111",
		"newPassword":"123343",
		"token":"token::9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
	}
- Response
> 返回字段说明

> 示例

	{
		"code":0,
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--
100701|参数账户为空|没有传account字段
100702|参数新密码为空|没有传newPassword字段
100703|参数令牌为空|没有传token字段
100704|账户格式不正确|账户不是合法的手机号或者合法的邮箱号
100705|用户不存在|账户输入有误或者用户被非法删除
100706|参数令牌不合法|令牌输入有误

###修改密码1008
- Path:/app/users/{userId}/password
- Request Method:PUT
- Request Headers:

	"key":"dbdf3891fcdabd41d2da10169312e3fa5dcfbcde132df142cff41c1261e115e3"

- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
oldPassword|String|是|无|旧密码
newPassword|String|是|无|新密码

	{
		"oldPassword":"das123",
		"newPassword":"123344"
	}
- Response

	{
	    "timestamp": 1478501466144,
	    "code": 0
	}

错误码|描述|原因
--|--|--
100801|参数旧密码为空|没有传旧密码字段
100802|参数新密码为空|没有传新密码字段
100803|参数userId为空|路径参数没有添加用户id
100804|用户未登录|1、用户未登录，2、用户在其他设备登录了
100805|旧密码不正确|旧密码输入有误
100806|用户不存在|用户不存在

###取得用户信息1009
- Path:/app/users/{userId}
- Request Method:GET
- Request Headers:

	"key":"13075ef09a8eff96af43b879fdc4c3151ba79ea36b22a8218625952b781fd0d8"

- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是||用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
gender|Integer|是|1|1、未选择，2、男 3、女
promotionCode|String|是|无|默认6位的数字或者大写字母，可以自定义长度
loginStatus|Boolean|是|false|登录状态
loginTime|Long|否|0|登录时间
logoutTime|Long|否|0|登出时间
version|Integer|否|0|登录设备版本
roles|Array[Integer]|是|[]|角色
qq|Object|是|{}|第三方平台的信息
provider|String|是|无|第三方平台的名称
oauthId|String|是|无|第三方平台的用户id
nickName|String|否|不羁+userId|第三方平台的用户昵称
avatar|String|否|默认用户头像|第三方平台的用户头像
token|String|是|无|第三方平台的用户令牌
level|Integer|是|1|用户等级
soundNotify|Boolean|是|true|是否声音提醒
vibrateNotify|Boolean|是|true|是否振动提醒
backGround|Object|是||用户背景图片
createTime|Long|是|0|用户创建时间
updateTime|Long|是|0|用户更新时间
key|String|是|无|授权码

> 示例

	{
	    "timestamp": 1481470630856,
	    "code": 0,
	    "result": {
	        "id": "584d621b2395512a982104b5",
	        "tel": {
	            "dialCode": 86,
	            "number": "13811111111"
	        },
	        "userId": 100001,
	        "nickName": "不羁100001",
	        "avatar": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "KJBPY5AJ",
	        "loginStatus": true,
	        "loginTime": 1481470605065,
	        "logoutTime": 1481470096458,
	        "version": 0,
	        "roles": [],
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 1481466395602,
	        "updateTime": 1481466395602
	    }
	}

错误码|描述|原因
--|--|--
100901|用户未登录|用户未登录
100902|用户不存在|输入的用户id有误，用户信息被非法删除，用户已注销

###修改用户信息1010
- Path:/app/users/{userId}
- Request Method:PATCH
- Request Headers

	"key":"4a3451488b06cd3b9b69cdc7741278b57ef1afd84555d921ad838fe7274f34cb"
- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
nickName|String|否|无|用户昵称
avatar|Object|否|无|用户头像
key|String|是|无|上传的图片的key, 如果有图片则必需此字段
bucket|String|是|无|上传的图片的七牛空间名称, 如果有图片则必需此字段
url|String|是|无|上传的图片的链接, 如果有图片则必需此字段
width|Integer|是|无|上传的图片的宽度, 如果有图片则必需此字段
height|Integer|是|无|上传的图片的高度, 如果有图片则必需此字段
fmt|String|是|无|上传的图片的格式, 如果有图片则必需此字段
hash|String|是|无|上传的图片的七牛返回哈希值, 如果有图片则必需此字段
size|String|是|无|上传的图片的大小, 如果有图片则必需此字段
gender|Integer|否|无|性别。1表示未选择，2表示男，3表示女
signature|String|否|无|用户签名
residence|String|否|无|用户的居住地
birthday|Long|否|无|用户的生日
level|Integer|否|无|用户等级
zodiac|Integer|否|无|星座。1 水瓶 2 双鱼 3 白羊 4 金牛 5 双子 6 巨蟹 7 狮子 8 处女 9 天秤 10 天蝎 11 射手 12 魔杰
soundNotify|Boolean|否|无|设置声音提醒。true为提醒，false不提醒
vibrateNotify|Boolean|否|无|设置振动提醒。true为提醒，false不提醒
backGround|Object|否|无|背景图片

	{
		"nickName":"魔法师",
		"gender":2,
		"birthday":145000000000,
		"signature":"前世的乡愁",
		"residence":"北京市海淀区闵庄路15号",
		"backGround": {
			"key":"xljb_1001_14500000000.jpg",
			"bucket":"bujilvxing-bucket",
			"fmt":"jpg",
			"hash":"a12231231231dfg3123ff",
			"size":3242424,
			"width":400,
			"height":400,
			"url":"http://1.jpg"
		},
		"avatar": {
			"key":"xljb_1001_14500000000.jpg",
			"bucket":"bujilvxing-bucket",
			"fmt":"jpg",
			"hash":"a12231231231dfg3123ff",
			"size":3242424,
			"width":400,
			"height":400,
			"url":"http://1.jpg"
		},
		"zodiac":1,
		"level":1,
		"soundNotify":true,
		"vibrateNotify":true
	}
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是||用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
gender|Integer|是|1|1、未选择，2、男 3、女
promotionCode|String|是|无|默认6位的数字或者大写字母，可以自定义长度
loginStatus|Boolean|是|false|登录状态
loginTime|Long|否|0|登录时间
logoutTime|Long|否|0|登出时间
version|Integer|否|0|登录设备版本
roles|Array[Integer]|是|[]|角色
qq|Object|是|{}|第三方平台的信息
provider|String|是|无|第三方平台的名称
oauthId|String|是|无|第三方平台的用户id
nickName|String|否|不羁+userId|第三方平台的用户昵称
avatar|String|否|默认用户头像|第三方平台的用户头像
token|String|是|无|第三方平台的用户令牌
level|Integer|是|1|用户等级
soundNotify|Boolean|是|true|是否声音提醒
vibrateNotify|Boolean|是|true|是否振动提醒
backGround|Object|是||用户背景图片
createTime|Long|是|0|用户创建时间
updateTime|Long|是|0|用户更新时间

> 示例

	{
	    "timestamp": 1481470813509,
	    "code": 0,
	    "result": {
	        "id": "584d621b2395512a982104b5",
	        "tel": {
	            "dialCode": 86,
	            "number": "13811111111"
	        },
	        "userId": 100001,
	        "nickName": "魔法师",
	        "avatar": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "KJBPY5AJ",
	        "loginStatus": true,
	        "loginTime": 1481470605065,
	        "logoutTime": 1481470096458,
	        "version": 0,
	        "roles": [1],
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 1481466395602,
	        "updateTime": 1481470813496
	    }
	}

错误码|描述|原因
--|--|--
101001|用户未登录|用户未登录
101002|性别不合法|性别参数的传的值不是1~3
101003|星座不合法|星座参数的传的值不是1~12
101004|用户不存在|用户id输入有误

###绑定手机号1011
- Path:/app/users/{userId}/tel
- Request Method:PUT
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
tel|String|是|无|用户手机号
token|Object|是|无|令牌

	{
		"tel":"13811111111",
		"token":"token::eddf6dce-4dbd-41b2-9893-d0d3a5b7bcfa"
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
101101|参数tel为空|没有传tel字段
101102|参数token为空|没有传token字段
101103|手机号格式不正确|手机号输入有误
101104|用户未登录|用户未登录
101105|手机号已存在|手机号已注册其他账号或者绑定其他账号
101106|令牌不合法|令牌不合法

###绑定邮箱1012
- Path:/app/users/{userId}/email
- Request Method:PUT
- Request Headers:

	"key":"e38f80ccd586bf82155c13bdc5fb78f641a893790f9c0ceae5419b9a2c6d00d8"
- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
email|String|是|无|邮箱号
token|Object|是|无|令牌

	{
		"email":"381364134@qq.com",
		"token":"token::eddf6dce-4dbd-41b2-9893-d0d3a5b7bcfa"
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
101201|参数email为空|没有传email字段
101202|参数token为空|没有传token字段
101203|邮箱号格式不正确|邮箱号输入有误
101204|用户未登录|用户未登录
101205|邮箱号已存在|邮箱号已注册其他账号或者绑定其他账号
101206|令牌不合法|令牌不合法

***
#其他模块
###申请商家1013
- Path:/app/sellers
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
tel|String|是|无|手机号

	{
		"tel":"13811111111"
	}
- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
101301|参数tel为空|没有传tel参数
101302|手机号格式不正确|手机号输入有误
101303|用户未登录|用户未登录

###用户反馈1014
- Path:/app/feedback
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
content|String|是|无|反馈内容
origin|String|否|无|从哪个App反馈过来的, 例如：不羁旅行

	{
		"content":"app做的真烂，能不能上点心",
		"origin":"app"
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
101401|参数content为空|没有传content参数
101402|用户未登录|用户未登录

***
#首页运营模块
###取得专栏1015
- Path:/app/columns
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
itemId|String|是|无|专栏项id
rank|Integer|否|无|排名
itemType|String|是|无|专栏项类型，比如："hotel"、"viewspot"
columnType|String|是|无|专栏类型，只能是"special"
title|String|是|无|标题
linkType|String|是|无|专栏链接类型。"app"表示内部链接，"web"表示网页链接
cover|Object|否|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
status|String|是|无|专栏状态，review表示待审核，pub表示使用中，disabled表示失效

> 示例

	{
	    "timestamp": 1481552586128,
	    "code": 0,
	    "result": [
	        {
	            "itemId": "584eb2c09542330bccd8a391",
	            "rank": 1,
	            "itemType": "viewspot",
	            "columnType": "special",
	            "title": "首页客栈1",
	            "desc": "描述1",
	            "link": "http://hotel1",
	            "linkType": "app",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "status": "pub"
	        }
	    ]
	}

错误码|描述|原因
--|--|--
101501|运营专栏数据为空|运营尚未弄好数据

###取得首页1016
- Path:/app/banners
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
itemId|String|是|无|专栏项id
rank|Integer|否|无|排名
itemType|String|是|无|专栏项类型，比如："hotel"、"viewspot"
columnType|String|是|无|专栏类型，只能是"slide"
title|String|是|无|标题
linkType|String|是|无|专栏链接类型。"app"表示内部链接，"web"表示网页链接
cover|Object|否|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
status|String|是|无|专栏状态，review表示待审核，pub表示使用中，disabled表示失效

> 示例

	{
	    "timestamp": 1481552433970,
	    "code": 0,
	    "result": [
	        {
	            "itemId": "584eb172954233105801a3e6",
	            "rank": 1,
	            "itemType": "hotel",
	            "columnType": "slide",
	            "title": "首页客栈1",
	            "desc": "描述1",
	            "link": "http://hotel1",
	            "linkType": "app",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "status": "pub"
	        }
	    ]
	}

错误码|描述|原因
--|--|--
101601|banner数据为空|运营尚未弄好数据

###取得首页商品列表(特产等)1017
- Path:/app/commodities
- Request Method:GET
- Request Headers:无
- Query String:category=speciality&offset=0&limit=100
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
category|String|是|无|商品分类
id|String|是|无|商品id
firstCategory|String|是|无|商品一级分类
secondCategory|String|是|无|商品二级分类
thirdCategory|String|是|无|商品三级分类
title|String|是|无|标题
desc|String|否|无|商品描述
cover|Object|否|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
price|Double|是|无|价格
marketPrice|Double|是|无|市场价格
status|Integer|否|无|状态
salesVolume|Integer|否|无|销量
rating|Double|否|无|评分

> 示例

	{
	    "timestamp": 1481597738543,
	    "code": 0,
	    "result": [
	        {
	            "id": "584f62d98edd1f1cc40e93dd",
	            "category": "specialities",
	            "commodities": [
	                {
	                    "id": "584f62d98edd1f1cc40e93c9",
	                    "firstCategory": "一级分类",
	                    "secondCategory": "二级分类",
	                    "thirdCategory": "三级分类",
	                    "title": "商品0",
	                    "desc": "商品描述0",
	                    "cover": {
	                        "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                        "width": 100,
	                        "height": 100,
	                        "fmt": "jpg"
	                    },
	                    "price": 1000,
	                    "marketPrice": 2000,
	                    "status": 2,
	                    "salesVolume": 0,
	                    "createTime": 1481597657636,
	                    "updateTime": 0,
	                    "rating": 0,
	                    "commodityType": "specialities"
	                }
	            ]
	        }
	    ]
	}

错误码|描述|原因
--|--|--
101701|首页运营商品数据为空|运营尚未准备数据
101702|首页商品数据为空|商品的数据被非法删除

###取得商品详情(特产等)1018
- Path:/app/commodities/{commodityId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|商品id
firstCategory|String|是|无|商品一级分类
secondCategory|String|是|无|商品二级分类
thirdCategory|String|是|无|商品三级分类
title|String|是|无|标题
desc|String|否|无|商品描述
cover|Object|否|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|ArrayObject|否|无|商品组图
price|Double|是|无|价格
marketPrice|Double|是|无|市场价格
status|Integer|否|无|状态
favorCnt|Integer|否|0|被喜欢的次数
salesVolume|Integer|否|无|销量
plans|ArrayObject|否|无|套餐列表
planId|String|是|无|套餐id
title|String|是|无|套餐标题
desc|String|是|无|套餐描述
pricings|ArrayObject|否|无|套餐价格列表
price|Integer|是|无|价格
timeRange|ArrayLong|否|无|价格的时间区间
marketPrice|Integer|是|无|套餐市场价格
price|Integer|是|无|套餐价格
stockInfos|ArrayObject|否|无|套餐库存列表
status|String|是|无|库存状态。empty,nonempty,plenty
quantity|Integer|是|无|库存量
timeRange|ArrayLong|否|无|库存的有效时间区间
timeRequired|Boolean|是|false|是否有失效性
rating|Double|否|无|评分
version|Integer|是|无|版本

> 示例

	{
	    "timestamp": 1481599196626,
	    "code": 0,
	    "result": {
	        "id": "584f62d98edd1f1cc40e93c9",
	        "firstCategory": "一级分类",
	        "secondCategory": "二级分类",
	        "thirdCategory": "三级分类",
	        "title": "商品0",
	        "desc": "商品描述0",
	        "cover": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "images": [
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        ],
	        "price": 1000,
	        "marketPrice": 2000,
	        "status": 2,
	        "favorCnt": 0,
	        "plans": [
	            {
	                "planId": "584f62d98edd1f1cc40e93c8",
	                "title": "套餐0",
	                "desc": "套餐描述0",
	                "pricings": [
	                    {
	                        "price": 1000,
	                        "timeRange": [
	                            1481597657337,
	                            1482202457337
	                        ]
	                    }
	                ],
	                "marketPrice": 2000,
	                "price": 1000,
	                "stockInfos": [
	                    {
	                        "status": "plenty",
	                        "quantity": 10000,
	                        "timeRange": [
	                            1481597657337,
	                            1482202457337
	                        ]
	                    }
	                ],
	                "timeRequired": true
	            }
	        ],
	        "salesVolume": 0,
	        "createTime": 1481597657636,
	        "updateTime": 0,
	        "rating": 0,
	        "version": 0,
	        "commodityType": "specical"
	    }
	}

错误码|描述|原因
--|--|--
101801|商品不存在|商品被非法删除

###取得首页攻略列表1019
- Path:/app/columnguides
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|攻略id
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
updateTime|Long|否|无|更新时间
title|String|是|无|标题
desc|String|否|无|描述
summary|String|是|无|摘要
detailUrl|String|是|无|详情链接
viewCnt|Integer|否|无|阅读数
shareCnt|Integer|否|无|转发数

> 示例

	{
	    "timestamp": 1481610904356,
	    "code": 0,
	    "result": [
	        {
	            "id": "584f91108edd1f202c8969d4",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "updateTime": 1481609488583,
	            "title": "攻略1",
	            "desc": "攻略描述1",
	            "summary": "攻略摘要1",
	            "detailUrl": "http://XXX",
	            "viewCnt": 0,
	            "shareCnt": 0
	        }
	    ]
	}

错误码|描述|原因
--|--|--
101901|攻略数据为空|运营人员没有弄好数据

###取得攻略详情1020
- Path:/app/guides/{guideId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|攻略id
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|ArrayObject|否|无|攻略图集
updateTime|Long|否|无|更新时间
title|String|是|无|标题
desc|String|否|无|描述
bestTripTime|String|否|无|最佳旅行时间
tips|String|否|无|提示
hotels|ArrayObject|否|无|旅馆列表
zhName|String|是|无|旅馆中文名称
enName|String|否|无|旅馆英文名称
url|String|是|无|旅馆链接
marketPrice|Integer|是|无|市场价
price|Integer|是|无|价格
saleVolume|Integer|是|无|销量
shoppings|ArrayObject|否|无|购物列表
restaurants|ArrayObject|否|无|美食列表
viewspots|ArrayObject|否|无|景点列表
tripPlans|ArrayObject|否|无|行程规划列表
userId|Long|是|无|作者用户id
nickName|String|是|无|作者昵称
avatar|Object|是|无|作者头像
originId|String|否|无|源行程规划的id
originUserId|Long|否|无|源作者用户id
originNickName|String|否|无|源作者昵称
originAvatar|Object|否|无|源作者头像
activities|ArrayObject|否|无|活动列表
joinNum|Integer|是|无|参加人数
favorCnt|Integer|是|0|收藏人数
commentCnt|Integer|是|0|评论人数
viewCnt|Integer|是|0|阅读人数
shareCnt|Integer|是|0|转发人数
voteCnt|Integer|是|0|点赞人数
isFree|Boolean|是|无|是否免费
summary|String|是|无|摘要
detailUrl|String|是|无|详情链接
viewCnt|Integer|否|无|阅读数
shareCnt|Integer|否|无|转发数

> 示例

	{
	    "timestamp": 1481609826938,
	    "code": 0,
	    "result": {
	        "id": "584f91108edd1f202c8969d4",
	        "cover": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "images": [
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        ],
	        "updateTime": 1481609488583,
	        "title": "攻略1",
	        "desc": "攻略描述1",
	        "bestTripTime": "4~5月",
	        "tips": "攻略提示",
	        "hotels": [
	            {
	                "id": "584f91108edd1f202c8969cc",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "zhName": "不羁客栈",
	                "enName": "BuJiLvXing Hotel",
	                "url": "http://xxx",
	                "marketPrice": 400,
	                "price": 200,
	                "saleVolume": 0
	            }
	        ],
	        "shoppings": [
	            {
	                "id": "584f91108edd1f202c8969ce",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "zhName": "优衣库",
	                "enName": "uniqlo",
	                "url": "http://xxx",
	                "marketPrice": 300,
	                "price": 200,
	                "saleVolume": 0
	            }
	        ],
	        "restaurants": [
	            {
	                "id": "584f91108edd1f202c8969cf",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "favorCnt": 0,
	                "zhName": "煌上煌",
	                "enName": "Huang Shang Huang",
	                "url": "http://xxx",
	                "marketPrice": 120,
	                "price": 80,
	                "saleVolume": 0
	            }
	        ],
	        "viewspots": [
	            {
	                "id": "584f91108edd1f202c8969d0",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "zhName": "庐山",
	                "enName": "LuShan",
	                "url": "http://xxx",
	                "marketPrice": 200,
	                "price": 180,
	                "saleVolume": 0
	            }
	        ],
	        "tripPlans": [
	            {
	                "id": "584f91108edd1f202c8969d2",
	                "userId": 100001,
	                "nickName": "魔法师",
	                "avatar": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "title": "庐山行程规划",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "originId": "584f91108edd1f202c8969d1",
	                "originUserId": 100002,
	                "originNickName": "逍遥",
	                "originAvatar": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                }
	            }
	        ],
	        "activities": [
	            {
	                "id": "584f91108edd1f202c8969d3",
	                "title": "活动标题1",
	                "joinNum": 0,
	                "favorCnt": 0,
	                "commentCnt": 0,
	                "viewCnt": 0,
	                "shareCnt": 0,
	                "voteCnt": 0,
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "isFree": true
	            }
	        ],
	        "summary": "攻略摘要1",
	        "detailUrl": "http://XXX",
	        "viewCnt": 0,
	        "shareCnt": 0
	    }
	}

错误码|描述|原因
--|--|--
102001|攻略不存在|攻略被非法删除

***
#POI模块
###取得客栈列表1021
- Path:/app/hotels
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必含|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个开始取
limit|Integer|否|10|取多少个

- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|客栈id
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
contact|Object|否|无|商家联系方式
phoneList|ArrayString|否|无|联系座机电话列表
cellphoneList|ArrayString|否|无|联系手机列表
qq|String|否|无|qq号
weixin|String|否|无|微信号
sina|String|否|无|新浪微博号
fax|String|否|无|传真号
email|String|否|无|邮箱号
website|String|否|无|网址
zhName|String|是|无|中文名
enName|String|否|无|英文名
url|String|是|无|客栈链接
marketPrice|Integer|是|无|市场价
price|Integer|是|无|价格
tags|ArrayString|否|无|客栈标签列表
address|Object|是|无|客栈地址
province|String|否|无|省
city|String|否|无|市
district|String|否|无|区
detail|String|否|无|详细地址
zipCode|String|否|无|邮编
saleVolume|Integer|否|无|销量
discount|Float|否|无|折扣

> 示例

	{
	    "timestamp": 1481612944764,
	    "code": 0,
	    "result": [
	        {
	            "id": "584f9e7a8edd1f19cc29a7fd",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "contact": {
	                "phoneList": [
	                    "010-62737359"
	                ],
	                "cellphoneList": [
	                    "13811111111"
	                ],
	                "qq": "1234213123",
	                "weixin": "mofashi",
	                "sina": "312413123",
	                "fax": "010-62737358",
	                "email": "3813231231@qq.com",
	                "website": "www.bjlx.com"
	            },
	            "zhName": "不羁客栈",
	            "enName": "BuJiLvXing Hotel",
	            "url": "http://xxx",
	            "marketPrice": 400,
	            "price": 200,
	            "tags": [
	                "不羁",
	                "客栈"
	            ],
	            "address": {
	                "province": "江西省",
	                "city": "南昌市",
	                "district": "昌北区",
	                "detail": "机场路16号",
	                "zipCode": "333133"
	            },
	            "saleVolume": 100,
	            "discount": 0.9800000190734863
	        }
	    ]
	}

###取得客栈详情1022
- Path:/app/hotels/{hotelId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|客栈id
lat|Double|否|无|经度
lng|Double|否|无|纬度
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|ArrayObject|否|无|客栈图集
rank|Integer|否|无|客栈排名
rating|Double|是|无|客栈得分
hotness|Double|是|无|客栈热度
favorCnt|Integer|否|无|客栈收藏数
contact|Object|否|无|商家联系方式
phoneList|ArrayString|否|无|联系座机电话列表
cellphoneList|ArrayString|否|无|联系手机列表
qq|String|否|无|qq号
weixin|String|否|无|微信号
sina|String|否|无|新浪微博号
fax|String|否|无|传真号
email|String|否|无|邮箱号
website|String|否|无|网址
zhName|String|是|无|中文名
enName|String|否|无|英文名
url|String|是|无|客栈链接
marketPrice|Integer|是|无|市场价
price|Integer|是|无|价格
priceDesc|String|否|无|价格描述
openTime|String|否|无|值班时间
description|Object|否|无|描述
desc|String|否|无|客栈描述
details|String|否|无|客栈详情
tips|String|否|无|客栈提示
traffic|String|否|无|到达客栈交通方式
tags|ArrayString|否|无|客栈标签列表
alias|ArrayString|否|无|客栈别名
targets|ArrayString|否|无|客栈所在行政区划分
source|String|否|无|客栈信息来源
guideUrl|String|否|无|客栈指南链接
address|Object|是|无|客栈地址
province|String|否|无|省
city|String|否|无|市
district|String|否|无|区
detail|String|否|无|详细地址
zipCode|String|否|无|邮编
saleVolume|Integer|否|无|销量
discount|Float|否|无|折扣
locList|ArrayObject|否|无|从属行政关系
rentCar|Object|否|无|租车信息
price|Integer|是|无|租车价格
minRentDay|Integer|是|无|起租天数
autoInsurance|Boolean|是|无|是否有保险
autoInsurancePrice|Integer|是|无|保险价格
pickup|Boolean|是|无|是否去接客人
availableDays|ArrayObject|否|无|客栈可使用的时间
bookTime|Long|是|无|预定时间
available|Boolean|是|无|是否可使用
price|Integer|是|无|价格

> 示例

	{
	    "timestamp": 1481613399622,
	    "code": 0,
	    "result": {
	        "id": "584f9e7a8edd1f19cc29a7fd",
	        "lat": 160,
	        "lng": 170,
	        "cover": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "images": [
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        ],
	        "rank": 1,
	        "rating": 0.99,
	        "hotness": 0.98,
	        "favorCnt": 0,
	        "contact": {
	            "phoneList": [
	                "010-62737359"
	            ],
	            "cellphoneList": [
	                "13811111111"
	            ],
	            "qq": "1234213123",
	            "weixin": "mofashi",
	            "sina": "312413123",
	            "fax": "010-62737358",
	            "email": "3813231231@qq.com",
	            "website": "www.bjlx.com"
	        },
	        "zhName": "不羁客栈",
	        "enName": "BuJiLvXing Hotel",
	        "url": "http://xxx",
	        "marketPrice": 400,
	        "price": 200,
	        "priceDesc": "双十二特惠",
	        "openTime": "9:00~21:00",
	        "description": {
	            "desc": "客栈描述",
	            "details": "客栈详情",
	            "tips": "客栈提示",
	            "traffic": "到达客栈交通方式"
	        },
	        "tags": [
	            "不羁",
	            "客栈"
	        ],
	        "alias": [
	            "不羁家"
	        ],
	        "targets": [
	            ""
	        ],
	        "source": "bjlx",
	        "guideUrl": "http://xxx",
	        "address": {
	            "province": "江西省",
	            "city": "南昌市",
	            "district": "昌北区",
	            "detail": "机场路16号",
	            "zipCode": "333133"
	        },
	        "locList": [
	            {
	                "id": "584f9e7a8edd1f19cc29a7fb",
	                "zhName": "南昌市",
	                "enName": "NanChang",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "visitCnt": 0,
	                "commentCnt": 0,
	                "favorCnt": 0,
	                "hotness": 0,
	                "rating": 0
	            }
	        ],
	        "saleVolume": 100,
	        "discount": 0.9800000190734863,
	        "rentCar": {
	            "id": "584f9e7a8edd1f19cc29a7fc",
	            "price": 100,
	            "minRentDay": 1,
	            "autoInsurance": true,
	            "autoInsurancePrice": 10,
	            "pickup": true
	        },
	        "locality": {
	            "id": "584f9e7a8edd1f19cc29a7fb",
	            "zhName": "南昌市",
	            "enName": "NanChang",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "visitCnt": 0,
	            "commentCnt": 0,
	            "favorCnt": 0,
	            "hotness": 0,
	            "rating": 0
	        },
	        "availableDays": [
	            {
	                "bookTime": 1481612922366,
	                "available": true,
	                "price": 100
	            }
	        ]
	    }
	}

错误码|描述|原因
--|--|--
102201|宾馆不存在|参数输入有误或者客栈被非法删除了

###取得目的地列表1023
- Path:/app/localities
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必含|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个开始取
limit|Integer|否|10|取多少个

- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|目的地id
zhName|String|是|无|中文名
enName|String|否|无|英文名
alias|Array String|否|无|别名
rating|Double|是|无|客栈得分
hotness|Double|是|无|客栈热度
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
desc|String|是|无|描述
lat|Double|否|无|经度
lng|Double|否|无|纬度

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id": "646f2da8b8ce0440eddb287f",
				"zhName":"南昌",
				"enName":"NanChang",
				"alias":[],
				"hotness":0.97,
				"rating":0.97,
				"tags":["红色摇篮"],
				"desc":"",
				"cover": {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"lat":115.27,
				"lng":28.09
			}
		]
	}

###取得目的地详情1024
- Path:/app/localities/{localityId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|目的地id
zhName|String|是|无|中文名
enName|String|否|无|英文名
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|ArrayObject|否|无|目的地图集
lat|Double|否|无|经度
lng|Double|否|无|纬度
rank|Integer|否|无|排名
remoteTraffic|ArrayObject|否|无|其他城市到目的地的交通方式
title|String|是|无|标题
desc|String|是|无|描述
localTraffic|ArrayObject|否|无|目的地本地的交通方式
shoppingIntro|String|否|无|购物简介
diningIntro|String|否|无|美食简介
cuisines|ArrayObject|否|无|特色菜系
activities|ArrayObject|否|无|特色活动列表
maxNum|Integer|是|无|活动最大允许人数
joinNum|Integer|是|无|已经报名活动人数
favorCnt|Integer|是|无|活动收藏人数
viewCnt|Integer|是|无|活动浏览人数
poster|Object|是|无|活动海报
tips|ArrayObject|否|无|小贴士
geoHistory|ArrayObject|否|无|历史文化
specials|ArrayObject|否|无|目的地亮点
alias|Array String|否|无|别名
visitCnt|Integer|是|无|去过目的地的人数
commentCnt|Integer|是|无|目的地的评论数
favorCnt|Integer|是|无|目的地的收藏数
rating|Double|是|无|客栈得分
hotness|Double|是|无|客栈热度
superAdm|Object|否|无|父行政区
tags|ArrayString|否|无|标签列表
desc|String|是|无|描述
travelMonth|String|是|无|适合旅行的月份
timeCostDesc|String|是|无|适合旅行的月份
timeCost|Integer|是|无|建议游玩时间

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id": "646f2da8b8ce0440eddb287f",
			"zhName":"南昌",
			"enName":"NanChang",
			"cover": {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"images":[
				{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			],
			"lat":115.27,
			"lng":28.09,
			"rank":1,
			"remoteTraffic":[
				{
					"cover": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"images":[
						{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					],
					"title":"",
					"desc":""
				}
			],
			"localTraffic":[
				{
					"cover": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"images":[
						{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					],
					"title":"",
					"desc":""
				}
			],
			"shoppingIntro":"",
			"diningIntro":"",
			"cuisines":[
				{
					"cover": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"images":[
						{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					],
					"title":"",
					"desc":""
				}
			],
			"activities":[
				{
					"id":"646f2da8b8ce0440eddb287f",
					"title":"亲子游活动",
					"maxNum":200,
					"joinNum" : 106,
					"favorCnt":100001,
					"viewCnt":88888,
					"poster":{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					}
				}
			],
			"tips":[
				{
					"cover": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"images":[
						{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					],
					"title":"",
					"desc":""
				}
			],
			"geoHistory":[
				{
					"cover": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"images":[
						{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					],
					"title":"",
					"desc":""
				}
			],
			"specials":[
				{
					"cover": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"images":[
						{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					],
					"title":"",
					"desc":""
				}
			],
			"alias":[],
			"visitCnt":19822,
			"commentCnt":2312,
			"favorCnt":3131,
			"hotness":0.97,
			"rating":0.97,
			"superAdm":{
				"id": "646f2da8b8ce0440eddb287f",
				"zhName":"南昌",
				"enName":"NanChang",
				"alias":[],
				"hotness":0.97,
				"rating":0.97,
				"tags":["红色摇篮"],
				"desc":"",
				"cover": {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"lat":115.27,
				"lng":28.09
			},
			"tags":["红色摇篮"],
			"desc":"",
			"travelMonth":"4月中旬到10月下旬",
			"timeCostDesc":"3到4天",
			"timeCost":4
		}
	}

错误码|描述|原因
--|--|--
102401|目的地不存在|参数输入有误或者目的地被非法删除了

###取得景点列表1025
- Path:/app/viewspots
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必含|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个开始取
limit|Integer|否|10|取多少个

- Request Body:无
- Response
> 返回字段说明

见1021

> 示例

	{
	    "timestamp": 1481614252371,
	    "code": 0,
	    "result": [
	        {
	            "id": "584fa3998edd1f2740896e8a",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "contact": {
	                "phoneList": [
	                    "010-62737359"
	                ],
	                "cellphoneList": [
	                    "13811111111"
	                ],
	                "qq": "1234213123",
	                "weixin": "mofashi",
	                "sina": "312413123",
	                "fax": "010-62737358",
	                "email": "3813231231@qq.com",
	                "website": "www.bjlx.com"
	            },
	            "zhName": "庐山",
	            "enName": "Lu Shan",
	            "url": "http://xxx",
	            "marketPrice": 180,
	            "price": 160,
	            "saleVolume": 0,
	            "discount": 0.8799999952316284
	        }
	    ]
	}

###取得景点详情1026
- Path:/app/viewspots/{viewspotId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

见1022

> 示例

	{
	    "timestamp": 1481614334614,
	    "code": 0,
	    "result": {
	        "id": "584fa3998edd1f2740896e8a",
	        "lat": 160,
	        "lng": 170,
	        "cover": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "images": [
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        ],
	        "rank": 1,
	        "rating": 0.99,
	        "hotness": 0.98,
	        "favorCnt": 0,
	        "contact": {
	            "phoneList": [
	                "010-62737359"
	            ],
	            "cellphoneList": [
	                "13811111111"
	            ],
	            "qq": "1234213123",
	            "weixin": "mofashi",
	            "sina": "312413123",
	            "fax": "010-62737358",
	            "email": "3813231231@qq.com",
	            "website": "www.bjlx.com"
	        },
	        "zhName": "庐山",
	        "enName": "Lu Shan",
	        "url": "http://xxx",
	        "marketPrice": 180,
	        "price": 160,
	        "priceDesc": "双十二特惠",
	        "openTime": "9:00~21:00",
	        "description": {
	            "desc": "景点描述",
	            "details": "景点详情",
	            "tips": "景点提示",
	            "traffic": "到达景点交通方式"
	        },
	        "tags": [
	            "不羁",
	            "客栈"
	        ],
	        "targets": [
	            "不羁家"
	        ],
	        "source": "bjlx",
	        "guideUrl": "http://xxx",
	        "address": {
	            "province": "江西省",
	            "city": "九江市",
	            "district": "庐山区",
	            "detail": "牯岭镇16号",
	            "zipCode": "333133"
	        },
	        "locList": [
	            {
	                "id": "584fa3998edd1f2740896e88",
	                "zhName": "九江市",
	                "enName": "JiuJiang",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "visitCnt": 0,
	                "commentCnt": 0,
	                "favorCnt": 0,
	                "hotness": 0,
	                "rating": 0
	            }
	        ],
	        "saleVolume": 0,
	        "discount": 0.8799999952316284,
	        "locality": {
	            "id": "584fa3998edd1f2740896e88",
	            "zhName": "九江市",
	            "enName": "JiuJiang",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "visitCnt": 0,
	            "commentCnt": 0,
	            "favorCnt": 0,
	            "hotness": 0,
	            "rating": 0
	        }
	    }
	}

错误码|描述|原因
--|--|--
102601|景点不存在|参数输入有误或者景点被非法删除了

###取得餐厅列表1027
- Path:/app/restaurants
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必含|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个开始取
limit|Integer|否|10|取多少个

- Request Body:无
- Response
> 返回字段说明

见1021

> 示例

	{
	    "timestamp": 1481615608587,
	    "code": 0,
	    "result": [
	        {
	            "id": "584fa8cf8edd1f122cd8df34",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "contact": {
	                "phoneList": [
	                    "010-62737359"
	                ],
	                "cellphoneList": [
	                    "13811111111"
	                ],
	                "qq": "1234213123",
	                "weixin": "mofashi",
	                "sina": "312413123",
	                "fax": "010-62737358",
	                "email": "3813231231@qq.com",
	                "website": "www.bjlx.com"
	            },
	            "favorCnt": 0,
	            "zhName": "季季红",
	            "enName": "JiJiHong",
	            "url": "http://xxx",
	            "marketPrice": 50,
	            "price": 34,
	            "saleVolume": 0
	        }
	    ]
	}

###取得餐厅详情1028
- Path:/app/restaurants/{restaurantId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

见1022

> 示例

	{
	    "timestamp": 1481615683218,
	    "code": 0,
	    "result": {
	        "id": "584fa8cf8edd1f122cd8df34",
	        "lat": 160,
	        "lng": 170,
	        "cover": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "images": [
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        ],
	        "rank": 1,
	        "rating": 0.99,
	        "hotness": 0.98,
	        "contact": {
	            "phoneList": [
	                "010-62737359"
	            ],
	            "cellphoneList": [
	                "13811111111"
	            ],
	            "qq": "1234213123",
	            "weixin": "mofashi",
	            "sina": "312413123",
	            "fax": "010-62737358",
	            "email": "3813231231@qq.com",
	            "website": "www.bjlx.com"
	        },
	        "zhName": "季季红",
	        "enName": "JiJiHong",
	        "url": "http://xxx",
	        "marketPrice": 50,
	        "price": 34,
	        "priceDesc": "双十二特惠",
	        "openTime": "9:00~21:00",
	        "description": {
	            "desc": "美食描述",
	            "details": "美食详情",
	            "tips": "美食提示",
	            "traffic": "到达美食交通方式"
	        },
	        "tags": [
	            "火锅",
	            "季季红"
	        ],
	        "targets": [
	            "季季红火锅"
	        ],
	        "source": "bjlx",
	        "guideUrl": "http://xxx",
	        "address": {
	            "province": "江西省",
	            "city": "九江市",
	            "district": "庐山区",
	            "detail": "优衣库",
	            "zipCode": "333133"
	        },
	        "locList": [
	            {
	                "id": "584fa8cf8edd1f122cd8df32",
	                "zhName": "九江市",
	                "enName": "JiuJiang",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "visitCnt": 0,
	                "commentCnt": 0,
	                "favorCnt": 0,
	                "hotness": 0,
	                "rating": 0
	            }
	        ],
	        "saleVolume": 0,
	        "locality": {
	            "id": "584fa8cf8edd1f122cd8df32",
	            "zhName": "九江市",
	            "enName": "JiuJiang",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "visitCnt": 0,
	            "commentCnt": 0,
	            "favorCnt": 0,
	            "hotness": 0,
	            "rating": 0
	        }
	    }
	}

错误码|描述|原因
--|--|--
102801|美食不存在|参数输入有误或者美食被非法删除了

###取得商场列表1029
- Path:/app/shoppings
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必含|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个开始取
limit|Integer|否|10|取多少个

- Request Body:无
- Response
> 返回字段说明

见1021

> 示例

	{
	    "timestamp": 1481614897912,
	    "code": 0,
	    "result": [
	        {
	            "id": "584fa6268edd1f0abc14cd8b",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "contact": {
	                "phoneList": [
	                    "010-62737359"
	                ],
	                "cellphoneList": [
	                    "13811111111"
	                ],
	                "qq": "1234213123",
	                "weixin": "mofashi",
	                "sina": "312413123",
	                "fax": "010-62737358",
	                "email": "3813231231@qq.com",
	                "website": "www.bjlx.com"
	            },
	            "zhName": "优衣库",
	            "enName": "UNIQLO",
	            "url": "http://xxx",
	            "marketPrice": 180,
	            "price": 160,
	            "saleVolume": 0,
	            "discount": 0.8799999952316284
	        }
	    ]
	}

###取得商场详情1030
- Path:/app/shoppings/{shoppingId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

见1022

> 示例

	{
	    "timestamp": 1481614965842,
	    "code": 0,
	    "result": {
	        "id": "584fa6268edd1f0abc14cd8b",
	        "lat": 160,
	        "lng": 170,
	        "cover": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "images": [
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        ],
	        "rank": 1,
	        "rating": 0.99,
	        "hotness": 0.98,
	        "favorCnt": 0,
	        "contact": {
	            "phoneList": [
	                "010-62737359"
	            ],
	            "cellphoneList": [
	                "13811111111"
	            ],
	            "qq": "1234213123",
	            "weixin": "mofashi",
	            "sina": "312413123",
	            "fax": "010-62737358",
	            "email": "3813231231@qq.com",
	            "website": "www.bjlx.com"
	        },
	        "zhName": "优衣库",
	        "enName": "UNIQLO",
	        "url": "http://xxx",
	        "marketPrice": 180,
	        "price": 160,
	        "priceDesc": "双十二特惠",
	        "openTime": "9:00~21:00",
	        "description": {
	            "desc": "购物描述",
	            "details": "购物详情",
	            "tips": "购物提示",
	            "traffic": "到达购物交通方式"
	        },
	        "tags": [
	            "服装",
	            "优衣库"
	        ],
	        "targets": [
	            "优衣库"
	        ],
	        "source": "bjlx",
	        "guideUrl": "http://xxx",
	        "address": {
	            "province": "江西省",
	            "city": "九江市",
	            "district": "庐山区",
	            "detail": "优衣库",
	            "zipCode": "333133"
	        },
	        "locList": [
	            {
	                "id": "584fa6268edd1f0abc14cd89",
	                "zhName": "九江市",
	                "enName": "JiuJiang",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "visitCnt": 0,
	                "commentCnt": 0,
	                "favorCnt": 0,
	                "hotness": 0,
	                "rating": 0
	            }
	        ],
	        "saleVolume": 0,
	        "discount": 0.8799999952316284,
	        "locality": {
	            "id": "584fa6268edd1f0abc14cd89",
	            "zhName": "九江市",
	            "enName": "JiuJiang",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "visitCnt": 0,
	            "commentCnt": 0,
	            "favorCnt": 0,
	            "hotness": 0,
	            "rating": 0
	        }
	    }
	}

错误码|描述|原因
--|--|--
103001|购物不存在|参数输入有误或者购物被非法删除了

***
#活动模块
###发布活动1031
- Path:/app/activities
- Request Method:POST
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
title|String|是|无|活动标题
maxNum|Integer|是|无|活动最大允许人数
startTime|Long|是|无|活动开始时间
endTime|Long|是|无|活动结束时间
address|Object|是|无|活动地址
province|String|是|无|省
city|String|是|无|市
district|String|是|无|区
detail|String|是|无|详细地址
zipCode|String|是|无|邮编
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
posters|Object|是|无|海报
theme|String|否|无|活动主题
category|String|否|无|活动分类
tags|ArrayString|否|无|活动标签
visiable|Integer|是|0|活动是否可见，1表示不可见，2表示可见
desc|String|否|无|活动描述
ticketIds|ArrayString|否|无|门票的id列表
isFree|Boolean|是|无|是否免费
isPhoneList|Boolean|是|无|报名者是否需要填写电话号
isCellphoneList|Boolean|是|无|报名者是否需要填写手机号
isQq|Boolean|是|无|报名者是否需要填写qq号
isWeixin|Boolean|是|无|报名者是否需要填写微信号
isSina|Boolean|是|无|报名者是否需要填写新浪微博号
isFax|Boolean|是|无|报名者是否需要填写传真号
isEmail|Boolean|是|无|报名者是否需要填写邮箱号
isWebsite|Boolean|是|无|报名者是否需要填写个人网址

	{
		"title":"北京冰雪嘉年华",
		"maxNum":200,
		"startTime":1481704641779,
		"endTime":1481704641779,
		"address": {
            "province": "江西省",
            "city": "南昌市",
            "district": "昌北区",
            "detail": "机场路16号",
            "zipCode": "333133"
        },
		"cover": {
            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
            "width": 100,
            "height": 100,
            "fmt": "jpg"
        },
		"posters":[
			{
                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
                "width": 100,
                "height": 100,
                "fmt": "jpg"
            },
            {
                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
                "width": 100,
                "height": 100,
                "fmt": "jpg"
            }
		],
		"theme":"音乐",
		"category":"摇滚",
		"tags": [
            "摇滚",
            "崔健"
        ],
		"visiable":2,
		"desc":"",
		"ticketIds":[
			"58510fb08edd1f21907f440c"
		],
		"isFree":true,
		"isPhoneList": true,
		"isCellphoneList":"true",
	    "isQq": true,
	    "isWeixin": true,
	    "isSina": true,
	    "isFax": true,
	    "isEmail": true,
	    "isWebsite": true
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|活动主键
title|String|是|无|活动标题
maxNum|Integer|是|无|活动最大允许人数
joinNum|Integer|是|无|报名人数
startTime|Long|是|无|活动开始时间
endTime|Long|是|无|活动结束时间
address|Object|是|无|活动地址
province|String|是|无|省
city|String|是|无|市
district|String|是|无|区
detail|String|是|无|详细地址
zipCode|String|是|无|邮编
favorCnt|Integer|否|无|收藏次数
commentCnt|Integer|否|无|评论次数
viewCnt|Integer|否|无|浏览次数
shareCnt|Integer|否|无|转发次数
voteCnt|Integer|否|无|点赞次数
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
posters|Object|是|无|海报
theme|String|否|无|活动主题
category|String|否|无|活动分类
tags|ArrayString|否|无|活动标签
visiable|Integer|是|0|活动是否可见，1表示不可见，2表示可见
desc|String|否|无|活动描述
creator|Object|否|无|发布活动用户信息
userId|Long|是|无|发布活动者用户id
nickName|String|是|无|发布活动者用户昵称
avatar|Object|否|无|发布活动用户头像
ticketIds|ArrayString|否|无|门票的id列表
isFree|Boolean|是|无|是否免费
publishTime|Long|是|无|发布时间
isPhoneList|Boolean|是|无|报名者是否需要填写电话号
isCellphoneList|Boolean|是|无|报名者是否需要填写手机号
isQq|Boolean|是|无|报名者是否需要填写qq号
isWeixin|Boolean|是|无|报名者是否需要填写微信号
isSina|Boolean|是|无|报名者是否需要填写新浪微博号
isFax|Boolean|是|无|报名者是否需要填写传真号
isEmail|Boolean|是|无|报名者是否需要填写邮箱号
isWebsite|Boolean|是|无|报名者是否需要填写个人网址

	{
	    "timestamp": 1481709140409,
	    "code": 0,
	    "result": {
	        "id": "585116548edd1f2514778e74",
	        "title": "北京冰雪嘉年华",
	        "maxNum": 200,
	        "joinNum": 0,
	        "startTime": 1481704641779,
	        "endTime": 1481704641779,
	        "address": {
	            "province": "江西省",
	            "city": "南昌市",
	            "district": "昌北区",
	            "detail": "机场路16号",
	            "zipCode": "333133"
	        },
	        "favorCnt": 0,
	        "commentCnt": 0,
	        "viewCnt": 0,
	        "shareCnt": 0,
	        "voteCnt": 0,
	        "cover": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "posters": [
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        ],
	        "theme": "音乐",
	        "category": "摇滚",
	        "tags": [
	            "摇滚",
	            "崔健"
	        ],
	        "visiable": 2,
	        "desc": "",
	        "creator": {
	            "id": "584e09778edd1f157cb87451",
	            "userId": 100001,
	            "nickName": "逍遥",
	            "avatar": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 0,
	                "height": 0,
	                "fmt": ""
	            }
	        },
	        "ticketIds": [
	            "58510fb08edd1f21907f440c"
	        ],
	        "isFree": true,
	        "publishTime": 1481709140365,
	        "isPhoneList": true,
	        "isCellphoneList": true,
	        "isQq": true,
	        "isWeixin": true,
	        "isSina": true,
	        "isFax": true,
	        "isEmail": true,
	        "isWebsite": true
	    }
	}

错误码|描述|原因
--|--|--
103101|用户未登录|用户未登录
103102|用户不存在|用户被非法删除

###取得活动列表1032
- Path:/app/activities
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必含|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个开始取
limit|Integer|否|10|取多少个

- Request Body:无
- Response
> 返回字段说明

字段名|类型|必需|默认值|描述
--|--|--|--|--
id|String|是|无|活动主键
title|String|是|无|活动标题
maxNum|Integer|是|无|活动最大允许人数
joinNum|Integer|是|无|报名人数
favorCnt|Integer|否|无|收藏次数
commentCnt|Integer|否|无|评论次数
viewCnt|Integer|否|无|浏览次数
shareCnt|Integer|否|无|转发次数
voteCnt|Integer|否|无|点赞次数
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
isFree|Boolean|是|无|是否免费

> 示例

	{
	    "timestamp": 1481704645826,
	    "code": 0,
	    "result": [
	        {
	            "id": "585104c18edd1f20bc6c8638",
	            "title": "活动标题1",
	            "maxNum": 100,
	            "joinNum": 20,
	            "favorCnt": 0,
	            "commentCnt": 0,
	            "viewCnt": 0,
	            "shareCnt": 0,
	            "voteCnt": 0,
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "isFree": false
	        }
	    ]
	}

###取得活动详情1033
- Path:/app/activities/{activityId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必需|默认值|描述
--|--|--|--|--
activity|Object|是|无|活动信息
id|String|是|无|活动主键
title|String|是|无|活动标题
maxNum|Integer|是|无|活动最大允许人数
joinNum|Integer|是|无|报名人数
startTime|Long|是|无|活动开始时间
endTime|Long|是|无|活动结束时间
address|Object|是|无|活动地址
province|String|是|无|省
city|String|是|无|市
district|String|是|无|区
detail|String|是|无|详细地址
zipCode|String|是|无|邮编
favorCnt|Integer|否|无|收藏次数
commentCnt|Integer|否|无|评论次数
viewCnt|Integer|否|无|浏览次数
shareCnt|Integer|否|无|转发次数
voteCnt|Integer|否|无|点赞次数
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
posters|Object|是|无|海报
theme|String|否|无|活动主题
category|String|否|无|活动分类
tags|ArrayString|否|无|活动标签
visiable|Integer|是|0|活动是否可见，1表示不可见，2表示可见
desc|String|否|无|活动描述
applicantInfos|ArrayObject|否|无报名者信息列表
userId|Long|是|无|报名活动者用户id
phoneList|ArrayString|否|无|报名活动者座机列表
cellphoneList|ArrayString|否|无|报名活动者手机列表
qq|String|否|无|报名活动者qq号
weixin|String|否|无|报名活动者微信号
sina|String|否|无|报名活动者新浪微博号
fax|String|否|无|报名活动者传真号
email|String|否|无|报名活动者邮箱号
website|String|否|无|报名活动者网址
creator|Object|否|无|发布活动用户信息
userId|Long|是|无|发布活动者用户id
nickName|String|是|无|发布活动者用户昵称
avatar|Object|否|无|发布活动用户头像
ticketIds|ArrayString|否|无|门票的id列表
isFree|Boolean|是|无|是否免费
publishTime|Long|是|无|发布时间
isPhoneList|Boolean|是|无|报名者是否需要填写电话号
isCellphoneList|Boolean|是|无|报名者是否需要填写手机号
isQq|Boolean|是|无|报名者是否需要填写qq号
isWeixin|Boolean|是|无|报名者是否需要填写微信号
isSina|Boolean|是|无|报名者是否需要填写新浪微博号
isFax|Boolean|是|无|报名者是否需要填写传真号
isEmail|Boolean|是|无|报名者是否需要填写邮箱号
isWebsite|Boolean|是|无|报名者是否需要填写个人网址
tickets|ArrayObject|否|无|门票的信息列表
price|Double|是|无|门票价格
marketPrice|Double|否|无|门票价格
free|Boolean|是|无|是否免费
refundWay|Integer|否|无|退款方式，1表示退款到平台公共账号，2表示原路返回，3表示不接受退款
refundDesc|String|否|无|委托平台说明
desc|String|否|无|票种说明
maxNum|Integer|是|无|门票最大张数
title|String|是|无|门票标题
creatorId|Integer|是|无|门票创建者用户id

> 示例

	{
	    "timestamp": 1481707483849,
	    "code": 0,
	    "result": {
	        "activity": {
	            "id": "58510fb08edd1f21907f4410",
	            "title": "活动标题1",
	            "maxNum": 100,
	            "joinNum": 20,
	            "startTime": 1481707440201,
	            "endTime": 1481708880201,
	            "address": {
	                "province": "江西省",
	                "city": "南昌市",
	                "district": "昌北区",
	                "detail": "机场路16号",
	                "zipCode": "333133"
	            },
	            "favorCnt": 0,
	            "commentCnt": 0,
	            "viewCnt": 0,
	            "shareCnt": 0,
	            "voteCnt": 0,
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "posters": [
	                {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                }
	            ],
	            "theme": "音乐",
	            "category": "摇滚",
	            "tags": [
	                "摇滚",
	                "崔健"
	            ],
	            "visiable": 2,
	            "desc": "",
	            "applicantInfos": [
	                {
	                    "userId": 100001,
	                    "phoneList": [
	                        "010-62737359"
	                    ],
	                    "cellphoneList": [
	                        "13811111111"
	                    ],
	                    "qq": "1234213123",
	                    "weixin": "mofashi",
	                    "sina": "312413123",
	                    "fax": "010-62737358",
	                    "email": "3813231231@qq.com",
	                    "website": "www.bjlx.com"
	                }
	            ],
	            "creator": {
	                "id": "584e09778edd1f157cb87451",
	                "userId": 100001,
	                "nickName": "逍遥",
	                "avatar": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 0,
	                    "height": 0,
	                    "fmt": ""
	                }
	            },
	            "ticketIds": [
	                "58510fb08edd1f21907f440c"
	            ],
	            "isFree": false,
	            "publishTime": 1481707440317,
	            "isPhoneList": false,
	            "isCellphoneList": false,
	            "isQq": false,
	            "isWeixin": false,
	            "isSina": false,
	            "isFax": false,
	            "isEmail": false,
	            "isWebsite": false
	        },
	        "tickets": [
	            {
	                "id": "58510fb08edd1f21907f440c",
	                "price": 160.1,
	                "marketPrice": 150.1,
	                "free": false,
	                "refundWay": 1,
	                "refundDesc": "退款到平台公共账号",
	                "desc": "票种说明",
	                "maxNum": 100,
	                "title": "门票1",
	                "creatorId": 100001
	            }
	        ]
	    }
	}

错误码|描述|原因
--|--|--
103301|活动不存在|参数输入有误或者活动被非法删除

###取得用户活动列表1034
- Path:/app/users/{userId}/activities
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必含|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个开始取
limit|Integer|否|10|取多少个

- Request Body:无
- Response
> 返回字段说明

字段名|类型|必需|默认值|描述
--|--|--|--|--
publishActivities|ArrayObject|否|无|用户发布的活动列表
id|String|是|无|活动主键
title|String|是|无|活动标题
maxNum|Integer|是|无|活动最大允许人数
joinNum|Integer|是|无|报名人数
favorCnt|Integer|否|无|收藏次数
commentCnt|Integer|否|无|评论次数
viewCnt|Integer|否|无|浏览次数
shareCnt|Integer|否|无|转发次数
voteCnt|Integer|否|无|点赞次数
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
isFree|Boolean|是|无|是否免费
joinActivities|ArrayObject|否|无|用户参与的活动列表

> 示例

	{
	    "timestamp": 1481769419015,
	    "code": 0,
	    "result": {
	        "publishActivities": [
	            {
	                "id": "58510fb08edd1f21907f4410",
	                "title": "活动标题1",
	                "maxNum": 100,
	                "joinNum": 20,
	                "favorCnt": 0,
	                "commentCnt": 0,
	                "viewCnt": 0,
	                "shareCnt": 0,
	                "voteCnt": 0,
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "isFree": false
	            }
	        ],
	        "joinActivities": [
	            {
	                "id": "58510fb08edd1f21907f4411",
	                "title": "活动标题2",
	                "maxNum": 100,
	                "joinNum": 20,
	                "favorCnt": 0,
	                "commentCnt": 0,
	                "viewCnt": 0,
	                "shareCnt": 0,
	                "voteCnt": 0,
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "isFree": false
	            }
	        ]
	    }
	}

错误码|描述|原因
--|--|--
103401|用户未登录|用户未登录

###报名活动1035
- Path:/app/activities/{activityId}/join
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
phoneList|ArrayString|否|无|报名活动者座机列表
cellphoneList|ArrayString|否|无|报名活动者手机列表
qq|String|否|无|报名活动者qq号
weixin|String|否|无|报名活动者微信号
sina|String|否|无|报名活动者新浪微博号
fax|String|否|无|报名活动者传真号
email|String|否|无|报名活动者邮箱号
website|String|否|无|报名活动者网址

> 示例

	{
	    "phoneList": [
	        "010-62737359"
	    ],
	    "cellphoneList": [
	        "13811111111"
	    ],
	    "qq": "1234213123",
	    "weixin": "mofashi",
	    "sina": "312413123",
	    "fax": "010-62737358",
	    "email": "3813231231@qq.com",
	    "website": "www.bjlx.com"
	}

- Response

		{
		    "timestamp": 1481784295718,
		    "code": 0
		}

错误码|描述|原因
--|--|--
103501|用户未登录|用户未登录

###退出报名1036
- Path:/app/activities/{activityId}/quit
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
phoneList|ArrayString|否|无|报名活动者座机列表
cellphoneList|ArrayString|否|无|报名活动者手机列表
qq|String|否|无|报名活动者qq号
weixin|String|否|无|报名活动者微信号
sina|String|否|无|报名活动者新浪微博号
fax|String|否|无|报名活动者传真号
email|String|否|无|报名活动者邮箱号
website|String|否|无|报名活动者网址

> 示例

	{
	    "phoneList": [
	        "010-62737359"
	    ],
	    "cellphoneList": [
	        "13811111111"
	    ],
	    "qq": "1234213123",
	    "weixin": "mofashi",
	    "sina": "312413123",
	    "fax": "010-62737358",
	    "email": "3813231231@qq.com",
	    "website": "www.bjlx.com"
	}

- Response

		{
		    "timestamp": 1481784295718,
		    "code": 0
		}

错误码|描述|原因
--|--|--
103601|用户未登录|用户未登录

###更新活动1037
- Path:/app/activities/{activityId}
- Request Method:PUT
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
title|String|是|无|活动标题
maxNum|Integer|是|无|活动最大允许人数
startTime|Long|是|无|活动开始时间
endTime|Long|是|无|活动结束时间
address|Object|是|无|活动地址
province|String|是|无|省
city|String|是|无|市
district|String|是|无|区
detail|String|是|无|详细地址
zipCode|String|是|无|邮编
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
posters|Object|是|无|海报
theme|String|否|无|活动主题
category|String|否|无|活动分类
tags|ArrayString|否|无|活动标签
visiable|Integer|是|0|活动是否可见，1表示不可见，2表示可见
desc|String|否|无|活动描述
ticketIds|ArrayString|否|无|门票的id列表
isFree|Boolean|是|无|是否免费
isPhoneList|Boolean|是|无|报名者是否需要填写电话号
isCellphoneList|Boolean|是|无|报名者是否需要填写手机号
isQq|Boolean|是|无|报名者是否需要填写qq号
isWeixin|Boolean|是|无|报名者是否需要填写微信号
isSina|Boolean|是|无|报名者是否需要填写新浪微博号
isFax|Boolean|是|无|报名者是否需要填写传真号
isEmail|Boolean|是|无|报名者是否需要填写邮箱号
isWebsite|Boolean|是|无|报名者是否需要填写个人网址

> 示例

	{
		"title":"北京冰雪嘉年华",
		"maxNum":200,
		"startTime":1481704641779,
		"endTime":1481704641779,
		"address": {
            "province": "江西省",
            "city": "南昌市",
            "district": "昌北区",
            "detail": "机场路16号",
            "zipCode": "333133"
        },
		"cover": {
            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
            "width": 100,
            "height": 100,
            "fmt": "jpg"
        },
		"posters":[
			{
                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
                "width": 100,
                "height": 100,
                "fmt": "jpg"
            },
            {
                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
                "width": 100,
                "height": 100,
                "fmt": "jpg"
            }
		],
		"theme":"音乐",
		"category":"摇滚",
		"tags": [
            "摇滚",
            "崔健"
        ],
		"visiable":2,
		"desc":"",
		"ticketIds":[
			"58510fb08edd1f21907f440c"
		],
		"isFree":true,
		"isPhoneList": true,
		"isCellphoneList":"true",
	    "isQq": true,
	    "isWeixin": true,
	    "isSina": true,
	    "isFax": true,
	    "isEmail": true,
	    "isWebsite": true
	}

- Response

		{
		    "timestamp": 1481784295718,
		    "code": 0
		}

错误码|描述|原因
--|--|--
103701|用户未登录|用户未登录

###添加门票1038
- Path:/app/tickets
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
price|Double|是|无|门票价格
marketPrice|Double|否|无|门票市场价格
free|Boolean|是|无|是否免费
refundWay|Integer|否|无|退款方式，1表示退款到平台公共账号，2表示原路返回，3表示不接受退款
refundDesc|String|否|无|委托平台说明
desc|String|否|无|票种说明
maxNum|Integer|是|无|门票最大张数
title|String|是|无|门票标题

> 示例

	{
		"title":"门票标题4",
		"price":121,
		"marketPrice":171,
		"free":"false",
		"refundWay":2,
		"refundDesc":"委托说明4",
		"desc":"票种说明4",
		"maxNum":190
	}	

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
price|Double|是|无|门票价格
marketPrice|Double|否|无|门票市场价格
free|Boolean|是|无|是否免费
refundWay|Integer|否|无|退款方式，1表示退款到平台公共账号，2表示原路返回，3表示不接受退款
refundDesc|String|否|无|委托平台说明
desc|String|否|无|票种说明
maxNum|Integer|是|无|门票最大张数
title|String|是|无|门票标题
creatorId|Integer|是|无|门票创建者用户id

	{
	    "timestamp": 1481803456267,
	    "code": 0,
	    "result": {
	        "id": "585286c06f31a4194492a137",
	        "price": 121,
	        "marketPrice": 171,
	        "free": false,
	        "refundWay": 2,
	        "refundDesc": "委托说明4",
	        "desc": "票种说明4",
	        "maxNum": 190,
	        "title": "门票标题4",
	        "creatorId": 100001
	    }
	}

错误码|描述|原因
--|--|--
103801|用户未登录|用户未登录
103802|收费门票价格不可为空|不是免费的门票时，门票的价格参数缺失

###删除门票1039
- Path:/app/tickets/{ticketId}
- Request Method:DELETE
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":100001

- Query String:无
- Request Body:无
- Response

		{
		    "timestamp": 1481803456267,
		    "code": 0
		}

错误码|描述|原因
--|--|--
103901|用户未登录|用户未登录
103902|门票被使用，删除失败|此门票被某个活动使用，无法删除

###修改门票1040
- Path:/app/tickets/{ticketId}
- Request Method:DELETE
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
price|Double|否|无|门票价格
marketPrice|Double|否|无|门票市场价格
free|Boolean|否|无|是否免费
refundWay|Integer|否|无|退款方式，1表示退款到平台公共账号，2表示原路返回，3表示不接受退款
refundDesc|String|否|无|委托平台说明
desc|String|否|无|票种说明
maxNum|Integer|否|无|门票最大张数
title|String|否|无|门票标题

	{
		"title":"门票标题4",
		"price":121,
		"marketPrice":171,
		"free":"false",
		"refundWay":2,
		"refundDesc":"委托说明4",
		"desc":"票种说明4",
		"maxNum":190
	}

- Response

		{
		    "timestamp": 1481803456267,
		    "code": 0
		}

错误码|描述|原因
--|--|--
104001|用户未登录|用户未登录
104002|门票不存在|门票被非法删除


###取得门票详情1041
- Path:/app/tickets/{ticketId}
- Request Method:DELETE
- Request Headers
- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
price|Double|否|无|门票价格
marketPrice|Double|否|无|门票市场价格
free|Boolean|是|无|是否免费
refundWay|Integer|否|无|退款方式，1表示退款到平台公共账号，2表示原路返回，3表示不接受退款
refundDesc|String|否|无|委托平台说明
desc|String|否|无|票种说明
maxNum|Integer|是|无|门票最大张数
title|String|是|无|门票标题
creatorId|Long|是|无|门票创建者id

	{
	    "timestamp": 1481705859808,
	    "code": 0,
	    "result": {
	        "id": "585109158edd1f24e08e9001",
	        "price": 160.1,
	        "marketPrice": 150.1,
	        "free": false,
	        "refundWay": 1,
	        "refundDesc": "退款到平台公共账号",
	        "desc": "票种说明",
	        "maxNum": 100,
	        "title": "门票1",
	        "creatorId": 100001
	    }
	}

错误码|描述|原因
--|--|--
104101|门票不存在|门票被非法删除


###取得用户门票列表1042
- Path:/app/users/{userId}/tickets
- Request Method:GET
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
price|Double|否|无|门票价格
marketPrice|Double|否|无|门票市场价格
free|Boolean|是|无|是否免费
refundWay|Integer|否|无|退款方式，1表示退款到平台公共账号，2表示原路返回，3表示不接受退款
refundDesc|String|否|无|委托平台说明
desc|String|否|无|票种说明
maxNum|Integer|是|无|门票最大张数
title|String|是|无|门票标题
creatorId|Long|是|无|门票创建者id

	{
	    "timestamp": 1481804185133,
	    "code": 0,
	    "result": [
	        {
	            "id": "5852865c6f31a40db49aa773",
	            "price": 160,
	            "marketPrice": 180,
	            "free": false,
	            "refundWay": 2,
	            "refundDesc": "委托说明",
	            "desc": "票种说明",
	            "maxNum": 100,
	            "title": "门票标题2",
	            "creatorId": 100001
	        },
	        {
	            "id": "5852869a6f31a40db49aa774",
	            "price": 161,
	            "marketPrice": 181,
	            "free": false,
	            "refundWay": 2,
	            "refundDesc": "委托说明3",
	            "desc": "票种说明3",
	            "maxNum": 120,
	            "title": "门票标题3",
	            "creatorId": 100001
	        }
	    ]
	}

错误码|描述|原因
--|--|--
104201|用户未登录|用户未登录

***
#游记模块
###取得游记列表1043
- Path:/app/travelnotes
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必需|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几条开始取
limit|Integer|否|10|取多少条

- Request Body:无
- Response

> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"646f2da8b8ce0440eddb287f",
				"cover": {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rating":0.97,
				"hotness":0.98,
				"title":"",
				"viewCnt":100,
				"summary":""
			}
		]
	}

错误码|描述|原因
--|--|--

###发布游记1044
- Path:/app/travelnotes
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
	    "userId":100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|Object|否|无|游记图集
title|String|是|无|游记标题
travelTime|Long|是|无|旅行时间
contents|ArrayObject|否|无|游记内容
summary|String|否|无|游记摘要

	{
		"cover": {
			"width":400,
			"height":400,
			"url":"http://1.jpg",
			"fmt":"jpg"
		},
		"images":[
			{
				"width":400,
				"height":400,
				"url":"http://1.jpg",
				"fmt":"jpg"
			}
		],
		"title":"游记标题",
		"travelTime":1481704641779,
		"contents":[
			{
				"day1":"游记内容1"
			},
			{
				"day2":"游记内容2"
			}
		],
		"summary":"游记摘要"
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|游记id
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|Object|否|无|游记图集
rating|Double|否|无|评分
hotness|Double|否|无|热度
title|String|是|无|游记标题
publishTime|Long|是|无|创建时间
favorCnt|Integer|否|0|收藏数
voteCnt|Integer|否|0|点赞数
commentCnt|Integer|否|0|评论数
viewCnt|Integer|否|0|阅读数
shareCnt|Integer|否|0|转发数
travelTime|Long|是|无|旅行时间
contents|ArrayObject|否|无|游记内容
summary|String|否|无|游记摘要
source|String|否|无|游记来源
essence|Boolean|否|无|游记是否精华

	{
	    "timestamp": 1481858532336,
	    "code": 0,
	    "result": {
	        "id": "58535de28edd1f39cc53d2b3",
	        "cover": {
	            "url": "http://1.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "author": {
	            "id": "584e09778edd1f157cb87451",
	            "userId": 100001,
	            "nickName": "逍遥",
	            "avatar": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        },
	        "images": [
	            {
	                "url": "http://1.jpg",
	                "width": 400,
	                "height": 400,
	                "fmt": "jpg"
	            }
	        ],
	        "rating": 0,
	        "hotness": 0,
	        "title": "游记标题",
	        "publishTime": 1481858530121,
	        "favorCnt": 0,
	        "voteCnt": 0,
	        "commentCnt": 0,
	        "viewCnt": 0,
	        "shareCnt": 0,
	        "travelTime": 1481704641779,
	        "summary": "游记标题",
	        "contents": [
	            {
	                "day1": "游记内容1"
	            },
	            {
	                "day2": "游记内容2"
	            }
	        ],
	        "source": "bjlx",
	        "essence": false
	    }
	}

错误码|描述|原因
--|--|--
104401|用户未登录|用户未登录
104402|用户不存在|用户被非法删除

###修改游记1045
- Path:/app/travelnotes/{travelnoteId}
- Request Method:PUT
- Request Headers

		"key":"e38f80ccd586bf82155c13bdc5fb78f641a893790f9c0ceae5419b9a2c6d00d8"
	    "userId"：100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|Object|否|无|游记图集
title|String|是|无|游记标题
travelTime|Long|是|无|旅行时间
contents|ArrayObject|否|无|游记内容
summary|String|否|无|游记摘要

> 示例
	{
		"cover": {
			"width":400,
			"height":400,
			"url":"http://1.jpg",
			"fmt":"jpg"
		},
		"images":[
			{
				"width":400,
				"height":400,
				"url":"http://1.jpg",
				"fmt":"jpg"
			}
		],
		"title":"游记标题1",
		"travelTime":1481858530121,
		"contents":[
			{
				"day1":"游记内容1"
			},
			{
				"day2":"游记内容2"
			}
		],
		"summary":"游记摘要"
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|游记id
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|Object|否|无|游记图集
rating|Double|否|无|评分
hotness|Double|否|无|热度
title|String|是|无|游记标题
publishTime|Long|是|无|创建时间
favorCnt|Integer|否|0|收藏数
voteCnt|Integer|否|0|点赞数
commentCnt|Integer|否|0|评论数
viewCnt|Integer|否|0|阅读数
shareCnt|Integer|否|0|转发数
travelTime|Long|是|无|旅行时间
contents|ArrayObject|否|无|游记内容
summary|String|否|无|游记摘要
source|String|否|无|游记来源
essence|Boolean|否|无|游记是否精华

> 示例

	{
	    "timestamp": 1481859504070,
	    "code": 0,
	    "result": {
	        "id": "58535de28edd1f39cc53d2b3",
	        "cover": {
	            "url": "http://1.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "author": {
	            "id": "584e09778edd1f157cb87451",
	            "userId": 100001,
	            "nickName": "逍遥",
	            "avatar": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        },
	        "images": [
	            {
	                "url": "http://1.jpg",
	                "width": 400,
	                "height": 400,
	                "fmt": "jpg"
	            }
	        ],
	        "rating": 0,
	        "hotness": 0,
	        "title": "游记标题1",
	        "publishTime": 1481858530121,
	        "favorCnt": 0,
	        "voteCnt": 0,
	        "commentCnt": 0,
	        "viewCnt": 0,
	        "shareCnt": 0,
	        "travelTime": 1481858530121,
	        "summary": "游记标题",
	        "contents": [
	            {
	                "day1": "游记内容1"
	            },
	            {
	                "day2": "游记内容2"
	            }
	        ],
	        "source": "bjlx",
	        "essence": false
	    }
	}

错误码|描述|原因
--|--|--
104501|用户未登录|用户未登录
104502|游记不存在|游记被非法删除；更新者不是游记的作者，没有权限

###取得游记详情1046
- Path:/app/travelnotes/{travelnoteId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|游记id
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|Object|否|无|游记图集
rating|Double|否|无|评分
hotness|Double|否|无|热度
title|String|是|无|游记标题
publishTime|Long|是|无|创建时间
favorCnt|Integer|否|0|收藏数
voteCnt|Integer|否|0|点赞数
commentCnt|Integer|否|0|评论数
viewCnt|Integer|否|0|阅读数
shareCnt|Integer|否|0|转发数
travelTime|Long|是|无|旅行时间
contents|ArrayObject|否|无|游记内容
summary|String|否|无|游记摘要
source|String|否|无|游记来源
essence|Boolean|否|无|游记是否精华

> 示例

	{
	    "timestamp": 1481859979650,
	    "code": 0,
	    "result": {
	        "id": "58535de28edd1f39cc53d2b3",
	        "cover": {
	            "url": "http://1.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "author": {
	            "id": "584e09778edd1f157cb87451",
	            "userId": 100001,
	            "nickName": "逍遥",
	            "avatar": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        },
	        "images": [
	            {
	                "url": "http://1.jpg",
	                "width": 400,
	                "height": 400,
	                "fmt": "jpg"
	            }
	        ],
	        "rating": 0,
	        "hotness": 0,
	        "title": "游记标题1",
	        "publishTime": 1481858530121,
	        "favorCnt": 0,
	        "voteCnt": 0,
	        "commentCnt": 0,
	        "viewCnt": 0,
	        "shareCnt": 0,
	        "travelTime": 1481858530121,
	        "summary": "游记标题",
	        "contents": [
	            {
	                "day1": "游记内容1"
	            },
	            {
	                "day2": "游记内容2"
	            }
	        ],
	        "source": "bjlx",
	        "essence": false
	    }
	}

错误码|描述|原因
--|--|--
104601|游记不存在|游记被非法删除

###删除游记1047
- Path:/app/travelnotes/{travelnoteId}
- Request Method:DELETE
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
	    "userId":100001

- Query String:无
- Request Body:无
- Response

> 示例

	{
		"code":0,
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--
104701|用户未登录|用户未登录

###取得用户游记列表1048
- Path:/app/users/{userId}/travelnotes
- Request Method:GET
- Request Headers

		"key":"e38f80ccd586bf82155c13bdc5fb78f641a893790f9c0ceae5419b9a2c6d00d8"
	    "userId"：100001

- Query String

字段名|类型|必需|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几条开始取
limit|Integer|否|10|取多少条

- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|游记id
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
images|Object|否|无|游记图集
rating|Double|否|无|评分
hotness|Double|否|无|热度
title|String|是|无|游记标题
publishTime|Long|是|无|创建时间
favorCnt|Integer|否|0|收藏数
voteCnt|Integer|否|0|点赞数
commentCnt|Integer|否|0|评论数
viewCnt|Integer|否|0|阅读数
shareCnt|Integer|否|0|转发数
travelTime|Long|是|无|旅行时间
contents|ArrayObject|否|无|游记内容
summary|String|否|无|游记摘要
source|String|否|无|游记来源
essence|Boolean|否|无|游记是否精华

> 示例

	{
	    "timestamp": 1481860337156,
	    "code": 0,
	    "result": [
	        {
	            "id": "58535de28edd1f39cc53d2b3",
	            "cover": {
	                "url": "http://1.jpg",
	                "width": 400,
	                "height": 400,
	                "fmt": "jpg"
	            },
	            "author": {
	                "id": "584e09778edd1f157cb87451",
	                "userId": 100001,
	                "nickName": "逍遥",
	                "avatar": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                }
	            },
	            "title": "游记标题1",
	            "summary": "游记标题",
	            "essence": false
	        }
	    ]
	}

错误码|描述|原因
--|--|--
104801|用户未登录|用户未登录

***
#时间线模块
###查看朋友圈1049
- Path:/app/moments
- Request Method:GET
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String

字段名|类型|必需|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几条开始取
limit|Integer|否|10|取多少条
latestTime|Long|否|无|最晚时间，取最新消息时需要传此参数
earliestTime|Long|否|无|最早时间，取过去的消息时需要传此参数

- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
publishTime|Long|是|无|朋友圈发布时间
userId|Long|是|无|发布朋友圈的用户id
nickName|String|是|无|发布朋友圈的用户昵称
avatar|Object|是|无|发布朋友圈的用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
originId|String|否|无|转发的朋友圈的源朋友圈的主键
originUserId|Long|否|无|转发的朋友圈的源朋友圈的作者id
originNickName|String|否|无|转发的朋友圈的源朋友圈的作者昵称
originAvatar|Object|否|无|转发的朋友圈的源朋友圈的作者头像
text|String|否|无|发布的内容为文字
images|ArrayNode|否|无|发布的内容为图片列表
comment|String|否|无|发布时，自己的评论
card|Object|否|无|发布的内容为卡片
id|String|是|无|卡片主键
title|String|是|无|卡片的标题
summary|String|是|无|卡片的摘要
cover|Object|是|无|卡片的封面图
detailUrl|String|是|无|卡片的链接
favorCnt|Integer|是|0|收藏次数
voteCnt|Integer|是|0|点赞次数

> 示例

	{
	    "timestamp": 1481866277404,
	    "code": 0,
	    "result": [
	        {
	            "id": "585376f88edd1f35384eda23",
	            "publishTime": 1481864952672,
	            "userId": 100001,
	            "nickName": "逍遥",
	            "avatar": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "images": [
	                {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                }
	            ],
	            "comment": "庐山真美",
	            "card": {
	                "id": "58535de28edd1f39cc53d2b3",
	                "title": "游记标题1",
	                "summary": "游记摘要1",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "thumb": {},
	                "detailUrl": "http://xxx.html"
	            },
	            "favorCnt": 0,
	            "voteCnt": 0
	        }
	    ]
	}

错误码|描述|原因
--|--|--
104901|latestTime和earliestTime不可同时为空|要么含有latestTime，要么含有earliestTime
104901|用户未登录|用户未登录

###查看某个人的朋友圈1050
- Path:/app/users/{targetId}/moments
- Request Method:GET
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String

字段名|类型|必需|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几条开始取
limit|Integer|否|100|取多少条
latestTime|Long|否|无|最晚时间，用于拉取最新的朋友圈
earliestTime|Long|否|无|最早时间，用户拉取老的朋友圈

- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
publishTime|Long|是|无|朋友圈发布时间
userId|Long|是|无|发布朋友圈的用户id
nickName|String|是|无|发布朋友圈的用户昵称
avatar|Object|是|无|发布朋友圈的用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
originId|String|否|无|转发的朋友圈的源朋友圈的主键
originUserId|Long|否|无|转发的朋友圈的源朋友圈的作者id
originNickName|String|否|无|转发的朋友圈的源朋友圈的作者昵称
originAvatar|Object|否|无|转发的朋友圈的源朋友圈的作者头像
text|String|否|无|发布的内容为文字
images|ArrayNode|否|无|发布的内容为图片列表
comment|String|否|无|发布时，自己的评论
card|Object|否|无|发布的内容为卡片
title|String|是|无|卡片的标题
summary|String|是|无|卡片的摘要
cover|Object|是|无|卡片的封面图
detailUrl|String|是|无|卡片的链接
favorCnt|Integer|是|0|收藏次数
voteCnt|Integer|是|0|点赞次数

> 示例

	{
	    "timestamp": 1481867419894,
	    "code": 0,
	    "result": [
	        {
	            "id": "585376f88edd1f35384eda23",
	            "publishTime": 1481864952672,
	            "userId": 100001,
	            "nickName": "逍遥",
	            "avatar": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "images": [
	                {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                }
	            ],
	            "comment": "庐山真美",
	            "card": {
	                "id": "58535de28edd1f39cc53d2b3",
	                "title": "游记标题1",
	                "summary": "游记摘要1",
	                "cover": {
	                    "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                    "width": 100,
	                    "height": 100,
	                    "fmt": "jpg"
	                },
	                "thumb": {},
	                "detailUrl": "http://xxx.html"
	            },
	            "favorCnt": 0,
	            "voteCnt": 0
	        }
	    ]
	}

错误码|描述|原因
--|--|--
105001|latestTime和earliestTime不可同时为空|latestTime和earliestTime参数都没有传
105001|用户未登录|用户未登录

###发布朋友圈1051
- Path:/app/moments
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
originId|String|否|无|源朋友圈消息的id
originUserId|Long|否|无|源朋友圈消息的发布者用户id
originNickName|String|否|无|源朋友圈消息的发布者用户昵称
originAvatar|Object|否|无|源朋友圈消息的发布者用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
text|String|否|无|文字消息朋友圈
images|ArrayNode|否|无|图片列表朋友圈
comment|String|否|无|自己评论
card|Object|否|无|卡片朋友圈
id|String|是|无|卡片的主键
title|String|是|无|卡片的标题
summary|String|是|无|卡片的摘要
cover|Object|是|无|卡片的封面图
detailUrl|String|是|无|卡片的链接

> 示例

	{
		"originId":"646f2da8b8ce0440eddb287f",
		"originUserId":10002,
		"originNickName":"魔法屋",
		"originAvatar":{
			"width":400,
			"height":400,
			"url":"http://1.jpg"
		},
		"text":"南昌如此之美",
		"images" : [
			{
				"key" : "default_user_avatar.jpg",
				"bucket" : "qiniu-bujilvxing",
				"url" : "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
				"width" : 100,
				"height" : 100,
				"fmt" : "jpg"
			},
			{
				"key" : "default_group_avatar.jpg",
				"bucket" : "qiniu-bujilvxing",
				"url" : "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
				"width" : 100,
				"height" : 100,
				"fmt" : "jpg"
			}
		],
		"comment":"庐山真美",
		"card":	{
			"id":"58535de28edd1f39cc53d2b3",
			"title":"游记标题1",
			"summary":"游记摘要1",
			"cover":{
				"key" : "default_user_avatar.jpg",
				"bucket" : "qiniu-bujilvxing",
				"url" : "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
				"width" : 100,
				"height" : 100,
				"fmt" : "jpg"
			},
			"detailUrl":"http://xxx.html"
		}
	}

- Response
> 返回字段说明

	{
	    "timestamp": 1481864953711,
	    "code": 0,
	    "result": {
	        "id": "585376f88edd1f35384eda23",
	        "publishTime": 1481864952672,
	        "userId": 100001,
	        "nickName": "逍遥",
	        "avatar": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "images": [
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_group_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            }
	        ],
	        "comment": "庐山真美",
	        "card": {
	            "id": "58535de28edd1f39cc53d2b3",
	            "title": "游记标题1",
	            "summary": "游记摘要1",
	            "cover": {
	                "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	                "width": 100,
	                "height": 100,
	                "fmt": "jpg"
	            },
	            "thumb": {},
	            "detailUrl": "http://xxx.html"
	        },
	        "favorCnt": 0,
	        "voteCnt": 0
	    }
	}

错误码|描述|原因
--|--|--
105101|originUserId不可为空|originId不为空时，没有传originUserId字段
105102|originNickName不可为空|originId不为空时，没有传originNickName字段
105103|originAvatar不可为空|originId不为空时，没有传originAvatar字段
105104|用户未登录|用户未登录
105105|用户不存在|用户被非法删除

***
#足迹模块
###发布用户足迹1052
- Path:/app/traces
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
	    "userId":100001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

> 示例

	{
		"traceTime":1425225600000,
		"cover":{
			"width":400,
			"height":400,
			"url":"http://1.jpg"
		},
		"images":[
			{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			}
		],
		"audio":{
			"length":15,
			"fileName":"123as3212c",
			"url":"http://a.mp3",
			"key":"sa2313dasdq1"
		},
		"status":1,
		"desc":"XXX",
		"restaurant":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"煌上煌烤鸭店",
			"enName":"",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"tags":["",""],
			"openTime":"9:00~21:00"
			"saleVolume":100,
			"discount":0.65
		},
		"hotel":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"如家快捷酒店",
			"enName":"RuJia",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"tags":["",""],
			"saleVolume":100,
			"discount":0.65
		},
		"viewspot":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"八一广场",
			"enName":"",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"tags":["",""],
			"openTime":"9:00~21:00"
			"saleVolume":100,
			"discount":0.65
		},
		"activity": {
			"id":"646f2da8b8ce0440eddb287f",
			"title":"亲子游活动",
			"maxNum":200,
			"joinNum" : 106,
			"favorCnt":100001,
			"viewCnt":88888,
			"poster":{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			}
		},
		"shopping": {
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"优衣库",
			"enName":"Uniqlo",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"tags":["",""],
			"openTime":"9:00~21:00"
			"saleVolume":100,
			"discount":0.65
		},
		"originId":"646f2da8b8ce0440eddb287f",
		"originUserId":1001,
		"originNickName":"魔法师",
		"originAvatar":{
			"url":"http://1.jpg",
			"width":800,
			"height":600
		},
		"lat":78.23,
		"lng":97.42
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result": {
			"id":"646f2da8b8ce0440eddb287f",
			"userId":1002,
			"nickName":"魔法师的小黑屋",
			"avatar":"htpp://2.jpg",
			"createTime":1425225600000,
			"updateTime":1425225600000,
			"traceTime":1425225600000,
			"shareCnt":100,
			"favorCnt":1000,
			"commentCnt":200,
			"viewCnt":100000,
			"cover":{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"desc":"XXX",
			"images":[
				{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			],
			"audio":{
				"length":15,
				"fileName":"123as3212c",
				"url":"http://a.mp3",
				"key":"sa2313dasdq1"
			},
			"status":1,
			"restaurant":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"煌上煌烤鸭店",
				"enName":"",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"hotel":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"如家快捷酒店",
				"enName":"RuJia",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"saleVolume":100,
				"discount":0.65
			},
			"viewspot":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"八一广场",
				"enName":"",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"activity": {
				"id":"646f2da8b8ce0440eddb287f",
				"title":"亲子游活动",
				"maxNum":200,
				"joinNum" : 106,
				"favorCnt":100001,
				"viewCnt":88888,
				"poster":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			},
			"shopping": {
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"优衣库",
				"enName":"Uniqlo",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"originId":"646f2da8b8ce0440eddb287f",
			"originUserId":1001,
			"originNickName":"魔法师",
			"originAvatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"lat":78.23,
			"lng":97.42
		}
	}

错误码|描述|原因
--|--|--

###修改足迹1053
- Path:/app/traces/{traceId}
- Request Method:PUT
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

> 示例

	{
		"traceTime":1425225600000,
		"cover":{
			"width":400,
			"height":400,
			"url":"http://1.jpg"
		},
		"images":[
			{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			}
		],
		"audio":{
			"length":15,
			"fileName":"123as3212c",
			"url":"http://a.mp3",
			"key":"sa2313dasdq1"
		},
		"status":1,
		"desc":"XXX",
		"restaurant":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"煌上煌烤鸭店",
			"enName":"",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"tags":["",""],
			"openTime":"9:00~21:00"
			"saleVolume":100,
			"discount":0.65
		},
		"hotel":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"如家快捷酒店",
			"enName":"RuJia",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"tags":["",""],
			"saleVolume":100,
			"discount":0.65
		},
		"viewspot":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"八一广场",
			"enName":"",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"tags":["",""],
			"openTime":"9:00~21:00"
			"saleVolume":100,
			"discount":0.65
		},
		"activity": {
			"id":"646f2da8b8ce0440eddb287f",
			"title":"亲子游活动",
			"maxNum":200,
			"joinNum" : 106,
			"favorCnt":100001,
			"viewCnt":88888,
			"poster":{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			}
		},
		"shopping": {
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"优衣库",
			"enName":"Uniqlo",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"tags":["",""],
			"openTime":"9:00~21:00"
			"saleVolume":100,
			"discount":0.65
		},
		"originId":"646f2da8b8ce0440eddb287f",
		"originUserId":1001,
		"originNickName":"魔法师",
		"originAvatar":{
			"url":"http://1.jpg",
			"width":800,
			"height":600
		},
		"lat":78.23,
		"lng":97.42
	}
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result": {
			"id":"646f2da8b8ce0440eddb287f",
			"userId":1002,
			"nickName":"魔法师的小黑屋",
			"avatar":"htpp://2.jpg",
			"createTime":1425225600000,
			"updateTime":1425225600000,
			"traceTime":1425225600000,
			"shareCnt":100,
			"favorCnt":1000,
			"commentCnt":200,
			"viewCnt":100000,
			"cover":{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"desc":"XXX",
			"images":[
				{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			],
			"audio":{
				"length":15,
				"fileName":"123as3212c",
				"url":"http://a.mp3",
				"key":"sa2313dasdq1"
			},
			"status":1,
			"restaurant":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"煌上煌烤鸭店",
				"enName":"",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"hotel":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"如家快捷酒店",
				"enName":"RuJia",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"saleVolume":100,
				"discount":0.65
			},
			"viewspot":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"八一广场",
				"enName":"",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"activity": {
				"id":"646f2da8b8ce0440eddb287f",
				"title":"亲子游活动",
				"maxNum":200,
				"joinNum" : 106,
				"favorCnt":100001,
				"viewCnt":88888,
				"poster":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			},
			"shopping": {
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"优衣库",
				"enName":"Uniqlo",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"originId":"646f2da8b8ce0440eddb287f",
			"originUserId":1001,
			"originNickName":"魔法师",
			"originAvatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"lat":78.23,
			"lng":97.42
		}
	}

错误码|描述|原因
--|--|--

###删除用户足迹1054
- Path:/app/traces/{traceId}
- Request Method:DELETE
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body:无
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result": {
			"id":"646f2da8b8ce0440eddb287f",
			"userId":1002,
			"nickName":"魔法师的小黑屋",
			"avatar":"htpp://2.jpg",
			"createTime":1425225600000,
			"updateTime":1425225600000,
			"traceTime":1425225600000,
			"shareCnt":100,
			"favorCnt":1000,
			"commentCnt":200,
			"viewCnt":100000,
			"cover":{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"desc":"XXX",
			"images":[
				{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			],
			"audio":{
				"length":15,
				"fileName":"123as3212c",
				"url":"http://a.mp3",
				"key":"sa2313dasdq1"
			},
			"status":1,
			"restaurant":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"煌上煌烤鸭店",
				"enName":"",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"hotel":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"如家快捷酒店",
				"enName":"RuJia",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"saleVolume":100,
				"discount":0.65
			},
			"viewspot":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"八一广场",
				"enName":"",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"activity": {
				"id":"646f2da8b8ce0440eddb287f",
				"title":"亲子游活动",
				"maxNum":200,
				"joinNum" : 106,
				"favorCnt":100001,
				"viewCnt":88888,
				"poster":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			},
			"shopping": {
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"优衣库",
				"enName":"Uniqlo",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"originId":"646f2da8b8ce0440eddb287f",
			"originUserId":1001,
			"originNickName":"魔法师",
			"originAvatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"lat":78.23,
			"lng":97.42
		}
	}

错误码|描述|原因
--|--|--

###取得足迹列表1055
- Path:/app/users/{userId}/traces
- Request Method:GET
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String


- Request Body
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"646f2da8b8ce0440eddb287f",
				"userId":1002,
				"nickName":"魔法师的小黑屋",
				"avatar":"htpp://2.jpg",
				"createTime":1425225600000,
				"updateTime":1425225600000,
				"traceTime":1425225600000,
				"cover":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"desc":"XXX",
				"status":1,
				"originId":"646f2da8b8ce0440eddb287f",
				"originUserId":1001,
				"originNickName":"魔法师",
				"originAvatar":{
					"url":"http://1.jpg",
					"width":800,
					"height":600
				},
				"lat":78.23,
				"lng":97.42
			}
		]
	}

错误码|描述|原因
--|--|--

###取得足迹详情1056
- Path:/app/traces/{traceId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body：无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result": {
			"id":"646f2da8b8ce0440eddb287f",
			"userId":1002,
			"nickName":"魔法师的小黑屋",
			"avatar":"htpp://2.jpg",
			"createTime":1425225600000,
			"updateTime":1425225600000,
			"traceTime":1425225600000,
			"shareCnt":100,
			"favorCnt":1000,
			"commentCnt":200,
			"viewCnt":100000,
			"cover":{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"desc":"XXX",
			"images":[
				{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			],
			"audio":{
				"length":15,
				"fileName":"123as3212c",
				"url":"http://a.mp3",
				"key":"sa2313dasdq1"
			},
			"status":1,
			"restaurant":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"煌上煌烤鸭店",
				"enName":"",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"hotel":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"如家快捷酒店",
				"enName":"RuJia",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"saleVolume":100,
				"discount":0.65
			},
			"viewspot":{
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"八一广场",
				"enName":"",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"activity": {
				"id":"646f2da8b8ce0440eddb287f",
				"title":"亲子游活动",
				"maxNum":200,
				"joinNum" : 106,
				"favorCnt":100001,
				"viewCnt":88888,
				"poster":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			},
			"shopping": {
				"id":"646f2da8b8ce0440eddb287f",
				"lat":180.1,
				"lng":180.1,
				"cover" : {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"rank":3,
				"hotness":0.97,
				"rating":0.98,
				"zhName":"优衣库",
				"enName":"Uniqlo",
				"url":"http://XXX",
				"marketPrice":280.6,
				"price":180.3,
				"tags":["",""],
				"openTime":"9:00~21:00"
				"saleVolume":100,
				"discount":0.65
			},
			"originId":"646f2da8b8ce0440eddb287f",
			"originUserId":1001,
			"originNickName":"魔法师",
			"originAvatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"lat":78.23,
			"lng":97.42
		}
	}

错误码|描述|原因
--|--|--

***
#行程规划模块
###发布行程规划1057
- Path:/app/tripplans
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":10001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
title|String|是|无|行程规划标题
desc|String|是|无|行程规划描述
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
orginId|String|否|无|源行程规划id
originUserId|Long|否|无|形程规划原创用户id
originNickName|String|否|无|形程规划原创用户昵称
originAvatar|Object|否|无|形程规划原创用户头像
tripItems|ArrayObject|否|无|行程规划项
tripTime|Long|是|无|行程项时间
desc|String|是|无|行程项描述
restaurant|Object|否|无|美食信息
id|String|是|无|美食id
cover|Object|是|无|美食封面图
zhName|String|是|无|美食中文名
enName|String|是|无|美食英文名
url|String|是|无|美食链接
marketPrice|Double|是|无|美食市场价
price|Double|是|无|美食价格
hotel|Object|否|无|宾馆信息
viewspot|Object|否|无|景点信息
activity|Object|否|无|活动信息
theme|String|是|无|活动主题
category|String|是|无|活动分类
visiable|Integer|是|无|活动是否可见，1表示不可见，2表示可见
isFree|Boolean|是|无|是否免费
shopping|Object|否|无|购物信息

> 示例

	{
		"title":"标题",
		"desc":"描述",
		"cover": {
			"url":"http://1.jpg",
			"width":800,
			"height":600
		},
		"originId":"646f2da8b8ce0440eddb287f",
		"originUserId":1001,
		"originNickName":"魔法师",
		"originAvatar":{
			"url":"http://1.jpg",
			"width":800,
			"height":600
		},
		"tripItems":[
			{
				"tripTime":145000000000,
				"desc":"XXX",
				"restaurant":{
					"id":"646f2da8b8ce0440eddb287f",
					"cover" : {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"zhName":"煌上煌烤鸭店",
					"enName":"",
					"url":"http://XXX",
					"marketPrice":280.6,
					"price":180.3
				},
				"hotel":{
					"id":"646f2da8b8ce0440eddb287f",
					"cover" : {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"zhName":"如家快捷酒店",
					"enName":"RuJia",
					"url":"http://XXX",
					"marketPrice":280.6,
					"price":180.3
				},
				"viewspot":{
					"id":"646f2da8b8ce0440eddb287f",
					"cover" : {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"zhName":"八一广场",
					"enName":"",
					"url":"http://XXX",
					"marketPrice":280.6,
					"price":180.3
				},
				"activity": {
					"id":"646f2da8b8ce0440eddb287f",
					"title":"亲子游活动",
					"cover":{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"theme":"主题",
					"category":"活动分类",
					"visiable":1,
					"isFree":true
				},
				"shopping": {
					"id":"646f2da8b8ce0440eddb287f",
					"cover" : {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"zhName":"优衣库",
					"enName":"Uniqlo",
					"url":"http://XXX",
					"marketPrice":280.6,
					"price":180.3
				}
			}
		]
	}
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--


> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"userId":1001,
			"nickName":"魔法师",
			"avatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"createTime":1450000000,
			"updateTime":1450000000,
			"shareCnt":100,
			"favorCnt":1000,
			"commentCnt":200,
			"viewCnt":100000,
			"tripItems":[
				{
					"tripTime":145000000000,
					"desc":"XXX",
					"restaurant":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"煌上煌烤鸭店",
						"enName":"",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					},
					"hotel":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"如家快捷酒店",
						"enName":"RuJia",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"saleVolume":100,
						"discount":0.65
					},
					"viewspot":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"八一广场",
						"enName":"",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					},
					"activity": {
						"id":"646f2da8b8ce0440eddb287f",
						"title":"亲子游活动",
						"maxNum":200,
						"joinNum" : 106,
						"favorCnt":100001,
						"viewCnt":88888,
						"poster":{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					},
					"shopping": {
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"优衣库",
						"enName":"Uniqlo",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					}
				}
			],
			"originId":"646f2da8b8ce0440eddb287f",
			"originUserId":1001,
			"originNickName":"魔法师",
			"originAvatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"hotness":0.9
		}
	}

错误码|描述|原因
--|--|--
105701|用户未登录|用户未登录
105702|用户不存在|用户被非法删除
105703|源用户id不可为空|没有传originUserId字段
105704|源用户昵称不可为空|没有传originNickName字段
105705|源用户头像不可为空|没有传originAvatar字段

###复制行程规划1058
- Path:/app/tripplans/{tripPlanId}/fork
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"userId":1001,
			"nickName":"魔法师",
			"avatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"createTime":1450000000,
			"updateTime":1450000000,
			"shareCnt":100,
			"favorCnt":1000,
			"commentCnt":200,
			"viewCnt":100000,
			"tripItems":[
				{
					"tripTime":145000000000,
					"desc":"XXX",
					"restaurant":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"煌上煌烤鸭店",
						"enName":"",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					},
					"hotel":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"如家快捷酒店",
						"enName":"RuJia",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"saleVolume":100,
						"discount":0.65
					},
					"viewspot":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"八一广场",
						"enName":"",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					},
					"activity": {
						"id":"646f2da8b8ce0440eddb287f",
						"title":"亲子游活动",
						"maxNum":200,
						"joinNum" : 106,
						"favorCnt":100001,
						"viewCnt":88888,
						"poster":{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					},
					"shopping": {
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"优衣库",
						"enName":"Uniqlo",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					}
				}
			],
			"originId":"646f2da8b8ce0440eddb287f",
			"originUserId":1001,
			"originNickName":"魔法师",
			"originAvatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"hotness":0.9
		}
	}

错误码|描述|原因
--|--|--

###取得行程规划列表1059
- Path:/app/users/{userId}/tripplans
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"646f2da8b8ce0440eddb287f",
				"userId":1001,
				"nickName":"魔法师",
				"avatar":{
					"url":"http://1.jpg",
					"width":800,
					"height":600
				},
				"title":"标题",
				"cover":{
					"url":"http://1.jpg",
					"width":800,
					"height":600
				},
				"originId":"646f2da8b8ce0440eddb287f",
				"originUserId":1001,
				"originNickName":"魔法师",
				"originAvatar":{
					"url":"http://1.jpg",
					"width":800,
					"height":600
				}
			}
		]
	}

错误码|描述|原因
--|--|--
105901|用户未登录|用户未登录

###修改行程规划1060
- Path:/app/tripplans/{tripPlanId}
- Request Method:PUT
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":10001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
title|String|是|无|行程规划标题
desc|String|是|无|行程规划描述
cover|Object|是|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
tripItems|ArrayObject|否|无|行程规划项
tripTime|Long|是|无|行程项时间
desc|String|是|无|行程项描述
restaurant|Object|否|无|美食信息
id|String|是|无|美食id
cover|Object|是|无|美食封面图
zhName|String|是|无|美食中文名
enName|String|是|无|美食英文名
url|String|是|无|美食链接
marketPrice|Double|是|无|美食市场价
price|Double|是|无|美食价格
hotel|Object|否|无|宾馆信息
viewspot|Object|否|无|景点信息
activity|Object|否|无|活动信息
theme|String|是|无|活动主题
category|String|是|无|活动分类
visiable|Integer|是|无|活动是否可见，1表示不可见，2表示可见
isFree|Boolean|是|无|是否免费
shopping|Object|否|无|购物信息

> 示例

	{
		"title":"标题",
		"desc":"描述",
		"cover": {
			"url":"http://1.jpg",
			"width":800,
			"height":600
		},
		"tripItems":[
			{
				"tripTime":145000000000,
				"desc":"XXX",
				"restaurant":{
					"id":"646f2da8b8ce0440eddb287f",
					"cover" : {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"zhName":"煌上煌烤鸭店",
					"enName":"",
					"url":"http://XXX",
					"marketPrice":280.6,
					"price":180.3
				},
				"hotel":{
					"id":"646f2da8b8ce0440eddb287f",
					"cover" : {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"zhName":"如家快捷酒店",
					"enName":"RuJia",
					"url":"http://XXX",
					"marketPrice":280.6,
					"price":180.3
				},
				"viewspot":{
					"id":"646f2da8b8ce0440eddb287f",
					"cover" : {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"zhName":"八一广场",
					"enName":"",
					"url":"http://XXX",
					"marketPrice":280.6,
					"price":180.3
				},
				"activity": {
					"id":"646f2da8b8ce0440eddb287f",
					"title":"亲子游活动",
					"cover":{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"theme":"主题",
					"category":"活动分类",
					"visiable":1,
					"isFree":true
				},
				"shopping": {
					"id":"646f2da8b8ce0440eddb287f",
					"cover" : {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"zhName":"优衣库",
					"enName":"Uniqlo",
					"url":"http://XXX",
					"marketPrice":280.6,
					"price":180.3
				}
			}
		]
	}
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"userId":1001,
			"nickName":"魔法师",
			"avatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"createTime":1450000000,
			"updateTime":1450000000,
			"shareCnt":100,
			"favorCnt":1000,
			"commentCnt":200,
			"viewCnt":100000,
			"tripItems":[
				{
					"tripTime":145000000000,
					"desc":"XXX",
					"restaurant":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"煌上煌烤鸭店",
						"enName":"",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					},
					"hotel":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"如家快捷酒店",
						"enName":"RuJia",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"saleVolume":100,
						"discount":0.65
					},
					"viewspot":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"八一广场",
						"enName":"",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					},
					"activity": {
						"id":"646f2da8b8ce0440eddb287f",
						"title":"亲子游活动",
						"maxNum":200,
						"joinNum" : 106,
						"favorCnt":100001,
						"viewCnt":88888,
						"poster":{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					},
					"shopping": {
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"优衣库",
						"enName":"Uniqlo",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					}
				}
			],
			"originId":"646f2da8b8ce0440eddb287f",
			"originUserId":1001,
			"originNickName":"魔法师",
			"originAvatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"hotness":0.9
		}
	}

错误码|描述|原因
--|--|--

###取得行程规划详情1061
- Path:/app/tripplans/{tripPlanId}
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"userId":1001,
			"nickName":"魔法师",
			"avatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"createTime":1450000000,
			"updateTime":1450000000,
			"shareCnt":100,
			"favorCnt":1000,
			"commentCnt":200,
			"viewCnt":100000,
			"tripItems":[
				{
					"tripTime":145000000000,
					"desc":"XXX",
					"restaurant":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"煌上煌烤鸭店",
						"enName":"",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					},
					"hotel":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"如家快捷酒店",
						"enName":"RuJia",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"saleVolume":100,
						"discount":0.65
					},
					"viewspot":{
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"八一广场",
						"enName":"",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					},
					"activity": {
						"id":"646f2da8b8ce0440eddb287f",
						"title":"亲子游活动",
						"maxNum":200,
						"joinNum" : 106,
						"favorCnt":100001,
						"viewCnt":88888,
						"poster":{
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						}
					},
					"shopping": {
						"id":"646f2da8b8ce0440eddb287f",
						"lat":180.1,
						"lng":180.1,
						"cover" : {
							"width":400,
							"height":400,
							"url":"http://1.jpg"
						},
						"rank":3,
						"hotness":0.97,
						"rating":0.98,
						"zhName":"优衣库",
						"enName":"Uniqlo",
						"url":"http://XXX",
						"marketPrice":280.6,
						"price":180.3,
						"tags":["",""],
						"openTime":"9:00~21:00"
						"saleVolume":100,
						"discount":0.65
					}
				}
			],
			"originId":"646f2da8b8ce0440eddb287f",
			"originUserId":1001,
			"originNickName":"魔法师",
			"originAvatar":{
				"url":"http://1.jpg",
				"width":800,
				"height":600
			},
			"hotness":0.9
		}
	}

错误码|描述|原因
--|--|--
106101|行程规划不存在|行程规划被删除

###删除行程规划1062
- Path:/app/tripplans/{tripPlanId}
- Request Method:DELETE
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body:无
- Response

> 示例

	{
		"code":0,
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--
106201|用户未登录|用户未登录

---
#问题
###发布问题1063
- Path:/app/questions
- Request Method:POST
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":10001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
source|String|否|"不羁旅行"|问题来源
topics|List<String>|否|无|问题的主题
tags|List<String>|否|无|问题的标签
title|String|是|无|标题
contents|String|是|无|具体描述

> 示例

	{
		"source":"baidu",
		"topics":["湖边","摄影","婚纱"],
		"tags":["旅拍","XXX"],
		"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
		"content":"如题"
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"timestamp":1425225600000,
		"result": {
			
		}
	}

错误码|描述|原因
--|--|--
106301|标题不可为空|没有传title字段
106302|内容不可为空|没有传content字段
106303|用户未登录|用户未登录
106304|用户不存在|用户被非法删除了

###取得问题详情1064
- Path:/app/quoras/{quoraId}
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
question|Object|是|无|问题信息
id|String|是|无|问题主键
source|String|是|无|问题来源
topics|List<String>|否|无|问题主题
tags|List<String>|否|无|问题标签
viewCnt|Integer|否|无|问题的浏览数
favorCnt|Integer|否|无|问题的收藏数
voteCnt|Integer|否|无|问题的点赞数
answerCnt|Integer|是|0|问题的回答数
maxVoteCnt|Integer|是|无|问题答案的最大点赞数
publishTime|Long|是|无|问题发布时间
title|String|是|无|问题的标题
content|String|是|无|问题的内容
author|Object|是|无|问题的作者
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
answers|ArrayNode|否|无|回答
voteCnt|Integer|否|无|回答的点赞数
accepted|Boolean|否|无|回答是否被采纳
publishTime|Long|是|无|回答发布时间
title|String|是|无|回答的标题
content|String|是|无|回答的内容
questionId|String|是|无|问题主键

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"question": {
				"id":"646f2da8b8ce0440eddb287f",
				"source":"baidu",
				"topics":["湖边","摄影","婚纱"],
				"tags":["旅拍","XXX"],
				"viewCnt":100,
				"favorCnt":100,
				"voteCnt":100,
				"answerCnt":10,
				"maxVoteCnt":10000,
				"publishTime":145000000000,
				"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
				"content":"如题",
				"author": {
					"userId":10001,
					"nickName":"魔法师",
					"avatar": {
						"url":"http://1.jpg",
						"width":800,
						"height":600
					}
				}
			},
			"answers":[
				{
					"voteCnt":100,
					"accepted":true,
					"author":{
						"userId":10001,
						"nickName":"雷锋",
						"avatar":{
							"width":600,
							"height":600,
							"url":"http://1.jpg"
						}
					},
					"publishTime":1450000000000,
					"title":"",
					"contents":"XXX",
					"questionId":"646f2da8b8ce0440eddb287f"
				}
			]
		}
	}

错误码|描述|原因
--|--|--
106401|问题不存在|问题的id输入有误，或者问题被非法删除


###取得用户问题列表1065
- Path:/app/users/{targetId}/questions
- Request Method:GET
- Request Headers

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":10001

- Query String

字段名|类型|必需|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个问题开始取
limit|Integer|否|10|取多少个问题

- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
question|Object|是|无|问题信息
id|String|是|无|问题主键
viewCnt|Integer|否|无|问题的浏览数
favorCnt|Integer|否|无|问题的收藏数
voteCnt|Integer|否|无|问题的点赞数
answerCnt|Integer|是|0|问题的回答数
maxVoteCnt|Integer|是|无|问题答案的最大点赞数
publishTime|Long|是|无|问题发布时间
title|String|是|无|问题的标题
content|String|是|无|问题的内容
author|Object|是|无|问题的作者
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"646f2da8b8ce0440eddb287f",
				"viewCnt":100,
				"favorCnt":100,
				"voteCnt":100,
				"answerCnt":10,
				"maxVoteCnt":10000,
				"publishTime":145000000000,
				"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
				"content":"如题",
				"author": {
					"userId":10001,
					"nickName":"魔法师",
					"avatar": {
						"url":"http://1.jpg",
						"width":800,
						"height":600
					}
				}
			}
		]
	}

错误码|描述|原因
--|--|--
106501|用户未登录|用户未登录

###取得问题列表1066
- Path:/app/questions
- Request Method:GET
- Request Headers:无
- Query String

字段名|类型|必需|默认值|描述
--|--|--|--|--
offset|Integer|否|0|从第几个问题开始取
limit|Integer|否|10|取多少个问题

- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
question|Object|是|无|问题信息
id|String|是|无|问题主键
viewCnt|Integer|否|无|问题的浏览数
favorCnt|Integer|否|无|问题的收藏数
voteCnt|Integer|否|无|问题的点赞数
answerCnt|Integer|是|0|问题的回答数
maxVoteCnt|Integer|是|无|问题答案的最大点赞数
publishTime|Long|是|无|问题发布时间
title|String|是|无|问题的标题
content|String|是|无|问题的内容
author|Object|是|无|问题的作者
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"646f2da8b8ce0440eddb287f",
				"viewCnt":100,
				"favorCnt":100,
				"voteCnt":100,
				"answerCnt":10,
				"maxVoteCnt":10000,
				"publishTime":145000000000,
				"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
				"content":"如题",
				"author": {
					"userId":10001,
					"nickName":"魔法师",
					"avatar": {
						"url":"http://1.jpg",
						"width":800,
						"height":600
					}
				}
			}
		]
	}

###添加回答1067
- Path:/app/questions/{questionId}/answers
- Request Method:POST
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
title|String|是|无|回答标题
content|String|是|无|回答的内容

> 示例

	{
		"title":"",
		"content":"XXX"
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
author|Object|是|无|问题的作者
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
voteCnt|Integer|否|无|回答的点赞数
accepted|Boolean|否|无|回答是否被采纳
publishTime|Long|是|无|回答发布时间
title|String|是|无|回答的标题
content|String|是|无|回答的内容
questionId|String|是|无|问题主键

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"voteCnt":100,
			"accepted":true,
			"author":{
				"userId":10001,
				"nickName":"雷锋",
				"avatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				}
			},
			"publishTime":1450000000000,
			"title":"",
			"contents":"XXX"
		}
	}

错误码|描述|原因
--|--|--
106701|标题不可为空|没有传title字段
106702|内容不可为空|没有传content字段
106703|用户未登录|用户未登录
106704|用户不存在|用户被非法删除了

###删除问题1068
- Path:/app/questions/{questionId}
- Request Method:DELETE
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body:无
- Response

	{
		"code":0,
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--
106801|用户未登录|用户未登录
106802|没有权限|不是问题的作者或者管理员

###删除问题1069
- Path:/app/questions/{questionId}/answers/{answerId}
- Request Method:DELETE
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body:无
- Response

	{
		"code":0,
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--
106901|用户未登录|用户未登录
106902|没有权限|不是问题的作者或者管理员

###编辑问题1070
- Path:/app/questions/{questionId}
- Request Method:PUT
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
source|String|否|"不羁旅行"|问题来源
topics|List<String>|否|无|问题的主题
tags|List<String>|否|无|问题的标签
title|String|是|无|标题
contents|String|是|无|具体描述

> 示例

	{
		"source":"baidu",
		"topics":["湖边","摄影","婚纱"],
		"tags":["旅拍","XXX"],
		"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
		"content":"如题"
	}

- Response

	{
		"code":0,
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--
107001|用户未登录|用户未登录

###编辑问题1071
- Path:/app/questions/{questionId}/answers/{answerId}
- Request Method:PUT
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
title|String|是|无|标题
contents|String|是|无|具体描述

> 示例

	{
		"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
		"content":"如题"
	}

- Response

	{
		"code":0,
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--
107101|用户未登录|用户未登录

---
#消息和社交
###关注用户1072
- Path:/app/users/{userId}/followings
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
followingId|Long|是|无|待关注用户id

	{
		"followingId":10001
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
107201|参数followingId不可为空|没有传followingId参数
107202|用户未登录|用户未登录
107203|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###取消关注用户1073
- Path:/app/users/{userId}/followings
- Request Method:DELETE
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
followingId|Long|是|无|待取消关注用户id

	{
		"followingId":10001
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
107301|参数followingId不可为空|没有传followingId参数
107302|用户未登录|用户未登录

###用户的好友列表1074
- Path:/app/users/{userId}/contacts
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:?offset=0&limit=100

参数名|类型|必需|默认值|参数描述
--|--|--|--|--
offset|Integer|否|0|从第offset个文档开始取
limit|Integer|否|200|取limit个文档

- Request Body:无
- Response

参数名|类型|必含|默认值|参数描述
--|--|--|--|--
id|String|是|无|主键
userId|Long|是|无|用户id
nickName|String|是|无|用户昵称
avatar|Object|是|无|用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
memo|String|否|无|备注

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"ca121da221313131cbd",
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"memo":"备注信息"
			}
		]
	}

错误码|描述|原因
--|--|--
107401|用户未登录|用户未登录

###获取好友(关注人)的详细信息1075
- Path:/app/users/{userId}/contacts/{contactId}
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是||用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
gender|Integer|是|1|1、未选择，2、男 3、女
promotionCode|String|是|无|默认6位的数字或者大写字母，可以自定义长度
loginStatus|Boolean|是|false|登录状态
loginTime|Long|否|0|登录时间
logoutTime|Long|否|0|登出时间
version|Integer|否|0|登录设备版本
roles|Array[Integer]|是|[]|角色
qq|Object|是|{}|第三方平台的信息
provider|String|是|无|第三方平台的名称
oauthId|String|是|无|第三方平台的用户id
nickName|String|否|不羁+userId|第三方平台的用户昵称
avatar|String|否|默认用户头像|第三方平台的用户头像
token|String|是|无|第三方平台的用户令牌
level|Integer|是|1|用户等级
soundNotify|Boolean|是|true|是否声音提醒
vibrateNotify|Boolean|是|true|是否振动提醒
backGround|Object|是||用户背景图片
createTime|Long|是|0|用户创建时间
updateTime|Long|是|0|用户更新时间
key|String|是|无|授权码
memo|String|否|无|备注

	{
	    "timestamp": 1478761623320,
	    "code": 0,
	    "result": {
	        "id": "581c52918edd1f0f94b5b1b9",
	        "email": "381364134@qq.com",
	        "userId": 1,
	        "nickName": "逍遥",
	        "avatar": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "UG4LV8V9",
	        "loginStatus": true,
	        "loginTime": 1478758833216,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [],
	        "memo": "魔法师",
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 0,
	        "updateTime": 0
	    }
	}

错误码|描述|原因
--|--|--
107501|用户未登录|用户未登录
107502|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###修改好友备注1076
- Path:/app/users/{userId}/contacts/{contactId}/memos
- Request Method:PUT
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
memo|String|是|无|备注

	{
		"memo":"备注"
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是||用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
gender|Integer|是|1|1、未选择，2、男 3、女
promotionCode|String|是|无|默认6位的数字或者大写字母，可以自定义长度
loginStatus|Boolean|是|false|登录状态
loginTime|Long|否|0|登录时间
logoutTime|Long|否|0|登出时间
version|Integer|否|0|登录设备版本
roles|Array[Integer]|是|[]|角色
qq|Object|是|{}|第三方平台的信息
provider|String|是|无|第三方平台的名称
oauthId|String|是|无|第三方平台的用户id
nickName|String|否|不羁+userId|第三方平台的用户昵称
avatar|String|否|默认用户头像|第三方平台的用户头像
token|String|是|无|第三方平台的用户令牌
level|Integer|是|1|用户等级
soundNotify|Boolean|是|true|是否声音提醒
vibrateNotify|Boolean|是|true|是否振动提醒
backGround|Object|是||用户背景图片
createTime|Long|是|0|用户创建时间
updateTime|Long|是|0|用户更新时间
key|String|是|无|授权码
memo|String|否|无|备注

	{
	    "timestamp": 1478761623320,
	    "code": 0,
	    "result": {
	        "id": "581c52918edd1f0f94b5b1b9",
	        "email": "381364134@qq.com",
	        "userId": 1,
	        "nickName": "逍遥",
	        "avatar": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "UG4LV8V9",
	        "loginStatus": true,
	        "loginTime": 1478758833216,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [],
	        "memo": "魔法师",
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 0,
	        "updateTime": 0
	    }
	}

错误码|描述|原因
--|--|--
107601|备注不可为空或者或字符串|没有传memo参数或者参数memo的值为空字符串
107602|用户未登录|用户未登录
107603|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###将用户加入黑名单1077
- Path:/app/users/{userId}/blacklist
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
blockId|Long|是|无|待屏蔽用户id

	{
		"blockId":10001
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
107701|屏蔽用户id不可为空|没有传blockId参数
107702|用户未登录|用户未登录
107703|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###将用户移除黑名单1078
- Path:/app/users/{userId}/blacklist/{blockId}
- Request Method:DELETE
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
107801|用户未登录|用户未登录
107802|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###用户的关注列表1079
- Path:/app/users/{userId}/followings
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String

字段名|类型|必需|默认值|描述
--|--|--|--|--
offset|Integer|否|0|第几个开始
limit|Integer|否|200|最多多少个

	offset:0
	limit:100

- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是|无|用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
memo|String|否|无|备注

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"memo":"备注"
			}
		]
	}

错误码|描述|原因
--|--|--
107901|用户未登录|用户未登录

###用户的粉丝列表1080
- Path:/app/users/{userId}/follows
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String

字段名|类型|必需|默认值|描述
--|--|--|--|--
offset|Integer|否|0|第几个开始
limit|Integer|否|200|最多多少个

	offset:0
	limit:100

- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|系统生成的主键
userId|Long|是|无|用户id
nickName|String|是|不羁+userId|用户昵称
avatar|Object|是|无|用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
memo|String|否|无|备注

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"memo":"备注"
			}
		]
	}

错误码|描述|原因
--|--|--
108001|用户未登录|用户未登录

###发送消息1081
- Path:/app/messages
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
receiverId|Long|是|无|接收消息的用户或者群组id
contents|Object|是|无|消息内容，为下面的其中一种，文字、图片、位置、表情或者语音等
text|String|否|无|文本
thumb|Object|否|无|缩略图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
full|Object|否|无|完整图
origin|Object|否|无|原图
audio|Object|否|无|音频
length|Integer|否|无|长度
fileName|String|否|无|文件名
url|String|否|无|语音链接
key|Integer|否|无|语音的七牛key
position|Object|否|无|位置
name|String|否|无|位置名称
lat|Double|否|无|经度
lng|Double|否|无|纬度
desc|String|否|无|位置描述
emoticon|Object|否|无|表情
group|String|否|无|表情组
url|String|否|无|表情链接
convId|String|否|无|会话id

	{
		"receiverId":10002,
		"content": {
			"text":"您好！",
			"thumb": {
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"full": {
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"origin": {
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"audio": {
				"length":20,
				"fileName":"",
				"url":"",
				"key":""
			},
			"position":{
				"name":"位置名称",
				"lat":160.11,
				"lng":150.44,
				"desc":"描述"
			},
			"emoticon": {
				"group":"表情组",
				"url":"http://1.jpg"
			}
		},
		"msgType":1,
		"chatType":1,
		"convId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
msgId|Long|是|无|消息id
senderId|Long|是|无|发送者用户id
senderNickName|String|是|无|发送者用户昵称
senderAvatar|Object|是|无|发送者用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
receiverId|Long|是|无|接收者用户(群组)id
full|Object|否|无|完整图
origin|Object|否|无|原图
audio|Object|否|无|音频
length|Integer|否|无|长度
fileName|String|否|无|文件名
url|String|否|无|语音链接
key|Integer|否|无|语音的七牛key
position|Object|否|无|位置
name|String|否|无|位置名称
lat|Double|否|无|经度
lng|Double|否|无|纬度
desc|String|否|无|位置描述
emoticon|Object|否|无|表情
group|String|否|无|表情组
url|String|否|无|表情链接
msgType|Integer|是|无|消息类型
chatType|Integer|是|无|1表示单聊，2表示群聊
timestamp|Long|是|无|消息创建时间
convId|String|否|无|会话id

消息类型|值
--|--
文本消息|1
图片消息|2
音频消息|3
位置消息|4
攻略消息|5
游记消息|6
景点消息|7
美食消息|8
购物消息|9
旅馆消息|10
商品消息|11
订单消息|12
名片消息|13
关注消息|14
提示消息|15
通知消息|16

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
			"msgId":10,
			"senderId":10001,
			"senderNickName":"魔法师",
			"senderAvatar":{
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"receiverId":10002,
			"content": {
				"text":"您好！",
				"thumb": {
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"full": {
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"origin": {
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"audio": {
					"length":20,
					"fileName":"",
					"url":"",
					"key":""
				},
				"position":{
					"name":"位置名称",
					"lat":160.11,
					"lng":150.44,
					"desc":"描述"
				},
				"emoticon": {
					"group":"表情组",
					"url":"http://1.jpg"
				}
			},
			"msgType":1,
			"chatType":1,
			"timestamp":145000000000,
			"convId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
		}
	}

错误码|描述|原因
--|--|--
108101|接收者id不可为空|没有传receiverId参数
108102|消息内容不可为空|没有传content参数
108103|消息类型不可为空|没有传msgType参数
108104|聊天类型不可为空|没有传chatType参数
108105|缩略图不可为空|如果发的图片消息，没有传thumb参数
108106|完整图不可为空|如果发的图片消息，没有传full参数
108107|原图不可为空|如果发的图片消息，没有传origin参数
108108|链接不可为空|如果发的富文本消息，没有传url参数
108109|音频不可为空|如果发的是音频消息，没有传audio参数
108110|长度不可为空|如果发的语音消息，没有传length参数
108111|位置不可为空|如果发的位置消息，没有传position参数
108112|经度不可为空|如果发的位置消息，没有传lat参数
108113|纬度不可为空|如果发的位置消息，没有传lng参数
108114|消息类型不合法|消息的类型的值不在合法范围内
108115|聊天类型不合法|聊天类型的值不在合法范围内
108116|用户不存在|接收者用户id输入有误或者用户被非法删除
108117|聊天组不存在|接收者是聊天组，但是聊天组id输入有误或者群组被非法删除
108118|回话id不合法|会话的id不合法
108119|用户未登录|用户未登录
108120|消息id不合法|重发旧消息时，旧消息被非法删除

###拉取消息1082
- Path:/app/users/{userId}/messages
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
purgeBefore|Long|否|0|客户端收到的最后一条消息的时间

	{
		"purgeBefore":145000000000
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
msgId|Long|是|无|消息id
senderId|Long|是|无|发送者用户id
senderNickName|String|是|无|发送者用户昵称
senderAvatar|Object|是|无|发送者用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
receiverId|Long|是|无|接收者用户(群组)id
full|Object|否|无|完整图
origin|Object|否|无|原图
audio|Object|否|无|音频
length|Integer|否|无|长度
fileName|String|否|无|文件名
url|String|否|无|语音链接
key|Integer|否|无|语音的七牛key
position|Object|否|无|位置
name|String|否|无|位置名称
lat|Double|否|无|经度
lng|Double|否|无|纬度
desc|String|否|无|位置描述
emoticon|Object|否|无|表情
group|String|否|无|表情组
url|String|否|无|表情链接
msgType|Integer|是|无|消息类型
chatType|Integer|是|无|1表示单聊，2表示群聊
timestamp|Long|是|无|消息创建时间
convId|String|否|无|会话id

消息类型|值
--|--
文本消息|1
图片消息|2
音频消息|3
位置消息|4
攻略消息|5
游记消息|6
景点消息|7
美食消息|8
购物消息|9
旅馆消息|10
商品消息|11
订单消息|12
名片消息|13
关注消息|14
提示消息|15
通知消息|16

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
				"msgId":10,
				"senderId":10001,
				"senderNickName":"魔法师",
				"senderAvatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"receiverId":10002,
				"content": {
					"text":"您好！",
					"thumb": {
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"full": {
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"origin": {
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"audio": {
						"length":20,
						"fileName":"",
						"url":"",
						"key":""
					},
					"position":{
						"name":"位置名称",
						"lat":160.11,
						"lng":150.44,
						"desc":"描述"
					},
					"emoticon": {
						"group":"表情组",
						"url":"http://1.jpg"
					}
				},
				"msgType":1,
				"chatType":1,
				"timestamp":145000000000,
				"convId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
			}
		]
	}

错误码|描述|原因
--|--|--
108201|用户未登录|用户未登录

###修改会话属性1083
- Path:/app/users/{userId}/conversations/{id}
- Request Method:PATCH
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
mute|Boolean|是|无|消息免打扰，true添加，false表示移除

	{
		"mute":true
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
108301|mute不可为空|没有传mute参数
108302|用户未登录|用户未登录
108303|会话不存在|会话id输入有误或者会话被非法删除

###根据会话id列表取得会话属性1084
- Path:/app/users/{userId}/conversations
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|会话id
createTime|Long|是|0|会话创建时间
updateTime|Long|是|0|会话更新时间
lastMsgContent|Long|是|无|会话最后一条消息的摘要
muted|Boolean|是|无|是否消息免打扰
pinned|Boolean|是|无|是否会话置顶

	{
		"code":0,
		"timestamp":1425225600000,
		"result":[
			{
				"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
				"createTime":145000000000,
				"updateTime":145000000000,
				"lastMsgContent":"哈哈",
				"muted":1,
				"pinned":1
			}
		]
	}

错误码|描述|原因
--|--|--
108401|会话id列表参数不可为空|没有传ids参数
108402|会话id不合法|id不是[0-9, a-f]的24位字符串
108403|用户未登录|用户未登录

---
#群组
###创建群组1085
- Path:/app/chatgroups
- Request Method:POST
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"name":"旅拍大咖",
		"groupDesc":"旅行摄影群组，欢迎各位参与",
		"avatar": {
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"tags":["旅拍","影楼","摄影工作室"],
		"creator":1001,
		"participants":[1002,1003,1004],
		"maxUsers":50,
		"visible":true
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
			"chatGroupId":100001,
			"name":"旅拍大咖",
			"groupDesc":"旅行摄影群组，欢迎各位参与",
			"avatar": {
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"tags":["旅拍","影楼","摄影工作室"],
			"creator":1001,
			"participants":[1002,1003,1004],
			"maxUsers":50,
			"visible":true
		}
	}

错误码|描述|原因
--|--|--

###修改群组信息1086
- Path:/app/chatgroups/{chatgroupId}
- Request Method:PUT
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"name":"旅拍大咖",
		"groupDesc":"旅行摄影群组，欢迎各位参与",
		"avatar": {
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"tags":["旅拍","影楼","摄影工作室"],
		"maxUsers":50,
		"visible":true
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
			"chatGroupId":100001,
			"name":"旅拍大咖",
			"groupDesc":"旅行摄影群组，欢迎各位参与",
			"avatar": {
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"tags":["旅拍","影楼","摄影工作室"],
			"creator":1001,
			"participants":[1002,1003,1004],
			"maxUsers":50,
			"visible":true
		}
	}

错误码|描述|原因
--|--|--

###取得群组信息1087
- Path:/app/chatgroups/{chatgroupId}
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
			"chatGroupId":100001,
			"name":"旅拍大咖",
			"groupDesc":"旅行摄影群组，欢迎各位参与",
			"avatar": {
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"tags":["旅拍","影楼","摄影工作室"],
			"creator":1001,
			"participants":[1002,1003,1004],
			"maxUsers":50,
			"visible":true
		}
	}

错误码|描述|原因
--|--|--

###取得群组成员信息列表1088
- Path:/app/chatgroups/{chatgroupId}/members
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				}
			}
		]
	}

错误码|描述|原因
--|--|--

###添加群成员1089
- Path:/app/chatgroups/{chatgroupId}/members
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"memberId":10002
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"timestamp":1425225600000
	}

###删除群成员1090
- Path:/app/chatgroups/{chatgroupId}/members/{memberId}
- Request Method:DETELE
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取得用户群组列表1091
- Path:/app/user/{userId}/chatgroups
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
				"chatGroupId":100001,
				"name":"旅拍大咖",
				"groupDesc":"旅行摄影群组，欢迎各位参与",
				"avatar": {
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"tags":["旅拍","影楼","摄影工作室"],
				"creator":1001,
				"participants":[1002,1003,1004],
				"maxUsers":50,
				"visible":true
			}
		]
	}

错误码|描述|原因
--|--|--

###发布帖子1092
- Path:/app/chatgroups/{chatgroupId}/posts
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"title":"周游20国姑娘被亿万富豪持枪逼婚",
		"cover":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"images":[
			{
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			}
		],
		"summary":"还有这种事，求逼婚",
		"content":"还有这种事，求逼婚",
		"authorId":1001, // 非必须
		"authorNickName":"魔法师", // 非必须
		"authorAvatar":{ // 非必须
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		}
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"title":"周游20国姑娘被亿万富豪持枪逼婚",
			"cover":{
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"images":[
				{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				}
			],
			"publishTime":1450000000000,
			"updateTime":1450000000000,
			"favorCnt":0,
			"commentCnt":0,
			"viewCnt":0,
			"shareCnt":0,
			"summary":"还有这种事，求逼婚",
			"content":"还有这种事，求逼婚",
			"rank":100,
			"hotness":0.96,
			"rating":0.98,
			"authorId":1001, // 非必须
			"authorNickName":"魔法师", // 非必须
			"authorAvatar":{ // 非必须
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			}
		}
	}

错误码|描述|原因
--|--|--

###取得群帖子列表1093
按照最新评论时间倒序
- Path:/app/chatgroups/{chatgroupId}/posts
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"title":"周游20国姑娘被亿万富豪持枪逼婚",
				"cover":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"publishTime":1450000000000,
				"updateTime":1450000000000,
				"favorCnt":0,
				"commentCnt":0,
				"viewCnt":0,
				"shareCnt":0,
				"summary":"还有这种事，求逼婚",
				"authorId":1001, // 非必须
				"authorNickName":"魔法师", // 非必须
				"authorAvatar":{ // 非必须
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				}
			}
		]
	}

错误码|描述|原因
--|--|--

###更新聊天组帖子1094
###删除聊天组帖子1095
###取得聊天组帖子详情1096
###取得用户帖子列表1097

---
#收藏
###添加收藏1098
- Path:/app/favorites
- Request Method:POST
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
favoriteType|Integer|是|无|收藏类型
itemId|String|是|无|收藏对象id
authorId|Long|否|无|作者id
authorNickName|String|否|无|作者昵称
authorAvatar|Object|否|无|作者头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
cover|Object|否|无|封面图
title|String|是|无|标题

	{
		"favoriteType":1,
		"itemId":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"title":"周游20国姑娘被亿万富豪持枪逼婚",
		"cover":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"authorId":10001,
		"authorNickName":"魔法师",
		"authorAvatar"：{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		}
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
109801|收藏类型不可为空|没有传favoriteType参数
109802|收藏对象id不可为空|没有传itemId参数
109803|收藏标题不可为空|没有传title参数
109804|用户未登录|用户未登录
109805|favoriteType不合法|favoriteType的输入有误
109806|itemId不合法|itemId的输入有误

###取消收藏1099
- Path:/app/favorites/{itemId}
- Request Method:DELETE
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body:无
- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
109901|收藏类型不可为空|没有传favoriteType参数
109902|favoriteType不合法|favoriteType的输入有误
109903|用户未登录|用户未登录

###取得用户收藏列表1100
- Path:/app/favorites
- Request Method:GET
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
userId|Long|是|无|用户id
favoriteType|Integer|是|无|收藏类型
itemId|String|是|无|收藏对象id
authorId|Long|否|无|作者用户id
authorNickName|String|否|无|作者昵称
authorAvatar|Object|否|无|作者头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
cover|Object|否|无|封面图
title|String|是|无|标题
favoriteTime|Long|是|无|收藏时间

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"",
				"userId":1001,
				"favoriteType":1,
				"itemId":"9c91a6deec8f42c9acfb0d1bd89dee9e",
				"authorId":1002,
				"authorNickName":"魔法师",
				"authorAvatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"cover":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"title":"被持枪逼婚",
				"favoriteTime":145000000000
			}
		]
	}

错误码|描述|原因
--|--|--
110001|用户未登录|用户未登录

---
#点赞
###点赞1101
- Path:/app/votes
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
	"userId":1001

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--
voteType|Integer|是|无|点赞类型
itemId|String|是|无|点赞对象id

	{
		"voteType":1,
		"itemId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
110101|点赞类型不可为空|没有传voteType参数
110102|点赞对象id不可为空|没有传itemId参数
110103|用户未登录|用户未登录
110104|voteType不合法|voteType输入有误
110105|itemId不合法|itemId输入有误

###取消点赞1102
- Path:/app/votes/{itemId}
- Request Method:DELETE
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body:无
- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--
110201|点赞类型不可为空|没有传voteType参数
110202|voteType不合法|voteType输入有误
110203|用户未登录|用户未登录

###取得点赞列表1103
- Path:/app/votes
- Request Method:GET
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:无
- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
userId|Long|是|无|用户id
voteType|Integer|是|无|点赞类型
itemId|String|是|无|点赞对象id
voteTime|Long|是|无|点赞时间

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"8c91a6deec8f42c9acfb0d1bd89dee9a"
				"userId":10001,
				"voteType":1,
				"itemId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
				"voteTime":145000000000000
			}
		]
	}

错误码|描述|原因
--|--|--
110301|用户未登录|用户未登录

###添加新评论1104
- Path:/app/comments
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"rating":0.96,
		"contents":"评论内容",
		"commentType":1,
		"itemId":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"images":[
			{
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			}
		]
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"id":"8c91a6deec8f42c9acfb0d1bd89dee9a",
			"rating":0.96,
			"userId":10001,
			"name":"魔法师",
			"avatar":{
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"contents":"评论内容",
			"commentType":1,
			"publishTime":145000000000,
			"itemId":"9c91a6deec8f42c9acfb0d1bd89dee9e",
			"images":[
				{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				}
			]
		}
	}

错误码|描述|原因
--|--|--

###删除评论1105
- Path:/app/comments/{commentId}
- Request Method:DELETE
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取得评论列表1106
- Path:/app/comments
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:

字段名|类型|必需|默认值|描述
--|--|--|--|--

	commentType=1
	itemId=9c91a6deec8f42c9acfb0d1bd89dee9e

- Request Body:无
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"8c91a6deec8f42c9acfb0d1bd89dee9a",
				"rating":0.96,
				"userId":10001,
				"name":"魔法师",
				"avatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				},
				"contents":"评论内容",
				"commentType":1,
				"publishTime":145000000000,
				"itemId":"9c91a6deec8f42c9acfb0d1bd89dee9e",
				"images":[
					{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					}
				]
			}
		]
	}

错误码|描述|原因
--|--|--


***
#搜索

###用户搜索1107
- Path:/app/users
- Request Method:GET
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:?query=13811111111
- Request Body
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
userId|Long|是|无|用户id
nickName|String|是|无|用户昵称
avatar|Object|否|无|用户头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result": {
			"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
			"userId":1001,
			"nickName":"魔法师",
			"avatar":{
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			}
		}
	}

错误码|描述|原因
--|--|--
110701|query不可为空|没有传query参数
110702|用户未登录|用户未登录
110703|query不合法|query仅支持 邮箱号，手机号，userId
110704|用户不存在|未找到要搜索的用户信息

###聊天组搜索1108
- Path:/app/chatgroups
- Request Method:GET
- Request Headers:

		"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
		"userId":1001

- Query String:?query=10001
- Request Body
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--
id|String|是|无|主键
chatgroupId|Long|是|无|聊天组id
name|String|是|无|聊天组名称
avatar|Object|否|无|聊天组头像
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
				"chatgroupId":1001,
				"name":"魔法师群",
				"avatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				}
			}
		]
	}

错误码|描述|原因
--|--|--
110801|query不可为空|没有传query参数
110802|用户未登录|用户未登录

###全站搜索1109
- Path:/app/search
- Request Method:GET
- Request Headers:无
- Query String:?query=青春下一站all=false&viewspot=true&trace=true&tripplan=true&quora=true&activity=true&user=ture&post=true&travelNote=true&restaurant=true&hotel=true

字段名|类型|必需|默认值|描述
--|--|--|--|--
query|String|是|无|搜索关键字
all|Boolean|否|false|是否全站搜索
momemt|Boolean|否|false|是否搜索朋友圈
commodity|Boolean|否|false|是否搜索商品
guide|Boolean|否|false|是否搜索攻略
viewspot|Boolean|否|false|是否搜索景点
trace|Boolean|否|false|是否搜索足迹
tripPlan|Boolean|否|false|是否搜索形成规划
quora|Boolean|否|false|是否搜索问答
activity|Boolean|否|false|是否搜索活动
travelNote|Boolean|否|false|是否搜索游记
restaurant|Boolean|否|false|是否搜索美食
hotel|Boolean|否|false|是否搜索宾馆
shopping|Boolean|否|false|是否搜索购物

- Request Body
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--     


	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"moments":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"userId":1001,
					"nickName":"魔法师",
					"avatar":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"text":"文本",
					"card":{
						"id":"9c91a6deec8f42c9acfb0d1bd89dee9e"
						"title":"标题",
						"summary":"摘要",
						"cover":{
							"width":600,
							"height":600,
							"url":"http://1.jpg"
						},
						"thumb":{
							"width":600,
							"height":600,
							"url":"http://1.jpg"
						},
						"detailUrl":"http:XXX"
					}
				}
			],
			"commodities":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"firstCategory":"一级分类",
					"secondCategory":"二级分类",
					"thirdCategory":"三级分类",
					"title":"标题",
					"desc":"描述",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"price":78.1,
					"marketPrice":120.24,
					"status":1,
					"salesVolume":100,
					"createTime":145000000000,
					"updateTime":145000000000,
					"rating":0.9,
					"commodityType":"specical"
				}
			],
			"guides":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					}
					"updateTime":14500000000000,
					"title":"标题",
					"desc":"描述",
					"summary":"摘要",
					"detailUrl":"http:XXX",
					"viewCnt":10000,
					"shareCnt":900
				}
			],
			"viewspots":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"不羁客栈",
					"enName":"Bu Ji Hotel",
					"url":"http:XXX",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"marketPrice":108.9,
					"price":12.1,
					"saleVolume":1000,
					"discount":0.75,
					"contact":{
						"phoneList":["21312","2423424"],
						"cellphoneList":["13811111111"],
						"qq":"234231231",
						"weixin":"bujilvxing",
						"sina":"23131",
						"fax":"26432423",
						"email":"aaa@qq.com",
						"website":"http://www.xxx.com"
					}
				}
			],
			"traces":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"userId":1001,
					"nickName":"魔法师",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"avatar":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"desc":"我在天安门广场,升旗仪式很威武",
					"audio":{
						"length":35,
						"createTime":1450000000000,
						"fileName":"文件名",
						"url":"http:XXX",
						"key":""
					},
					"originId":"",
					"originUserId":1002,
					"originNickName":"昵称",
					"originAvatar":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
				}
			],
			"tripPlans":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"userId":1001,
					"nickName":"魔法师",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"avatar":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"title":"我在天安门广场,升旗仪式很威武",
					"originId":"",
					"originUserId":1002,
					"originNickName":"昵称",
					"originAvatar":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
				}
			],
			"activities":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"潜水沙龙",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"maxNum":100,
					"joinNum":34,
					"favorCnt":54,
					"commentCnt":34,
					"viewCnt":54,
					"shareCnt":53,
					"isFree":true
				}
			],
			"quoras":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"现在的南昌冷吗？",
					"viewCnt":133,
					"answerCnt":125,
					"maxVoteCnt":111,
					"author":{
						"id":"",
						"userId":10009,
						"nickName":"魔法师",
						"avatar":{
							"width":600,
							"height":600,
							"url":"http://1.jpg"
						}
					}
					"publishTime":1450000000000，
					"title":"标题",
					"contents":"内容"
				}
			],
			"restaurants":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"煌上煌",
					"enName":"huang shang huang",
					"url":"http:XXX",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"marketPrice":108.9,
					"price":12.1,
					"saleVolume":1000,
					"contact":{
						"phoneList":["21312","2423424"],
						"cellphoneList":["13811111111"],
						"qq":"234231231",
						"weixin":"bujilvxing",
						"sina":"23131",
						"fax":"26432423",
						"email":"aaa@qq.com",
						"website":"http://www.xxx.com"
					}
				}
			],
			"hotels":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"煌上煌",
					"enName":"huang shang huang",
					"url":"http:XXX",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"marketPrice":108.9,
					"price":12.1,
					"saleVolume":1000,
					"discount":0.75
					"contact":{
						"phoneList":["21312","2423424"],
						"cellphoneList":["13811111111"],
						"qq":"234231231",
						"weixin":"bujilvxing",
						"sina":"23131",
						"fax":"26432423",
						"email":"aaa@qq.com",
						"website":"http://www.xxx.com"
					},
					"tags":["",""],
					"address":{
						"province":"江西",
						"city":"南昌",
						"district":"昌北",
						"detail":"详细地址",
						"zipCode":"212312"
					}
				}
			],
			"travelNotes":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"title":"井冈山一日游",
					"summary":"灰常漂亮",
					"essence":true
				}
			],
			"shoppings":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"煌上煌",
					"enName":"huang shang huang",
					"url":"http:XXX",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"marketPrice":108.9,
					"price":12.1,
					"saleVolume":1000,
					"discount":0.75
					"contact":{
						"phoneList":["21312","2423424"],
						"cellphoneList":["13811111111"],
						"qq":"234231231",
						"weixin":"bujilvxing",
						"sina":"23131",
						"fax":"26432423",
						"email":"aaa@qq.com",
						"website":"http://www.xxx.com"
					}
				}
			]
		}
	}

错误码|描述|原因
--|--|--
110901|query不可为空|没有传query参数

