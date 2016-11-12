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
		"account":"13811111111",   // 可以是手机号，也可以是邮箱号
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
promotionCodeSize|Integer|否|6|邀请码长度

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
	    "timestamp": 1478098984477,
	    "code": 0,
	    "result": {
	        "id": "581a0028d903d71bb874d1df",
	        "email": "381364134@qq.com",
	        "userId": 2,
	        "nickName": "不羁2",
	        "avatar": {
	            "id": "5819fff1d903d71bb874d1d8",
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "56B992",
	        "loginStatus": false,
	        "loginTime": 0,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [],
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "id": "5819fff1d903d71bb874d1d9",
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
	    "timestamp": 1478251455371,
	    "code": 0,
	    "result": {
	        "id": "581c52918edd1f0f94b5b1b9",
	        "email": "381364134@qq.com",
	        "userId": 1,
	        "nickName": "不羁1",
	        "avatar": {
	            "id": "581c51c88edd1f0f94b5b1b1",
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "UG4LV8V9",
	        "loginStatus": true,
	        "loginTime": 1478251168995,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [],
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "id": "581c51c88edd1f0f94b5b1b2",
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 0,
	        "updateTime": 0,
	        "key": "75dd5365bfb3fd94620458bbff79cb27f139a3d39c95e9dfce2f912d26c7ff1e"
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
		"provider":"qq",
		"oauthId":"231da3213da",
		"nickName":"小呆",
		"avatar":"http://1.jpg",
		"token":"a23ca21354cad2321c231c",
		"clientId":"ad312c3123b323e32b2332a"
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
	    "timestamp": 1478533884105,
	    "code": 0,
	    "result": {
	        "id": "58209febd903d70e107cab77",
	        "userId": 4,
	        "nickName": "小呆",
	        "avatar": {
	            "id": "",
	            "url": "http://1.jpg",
	            "width": 0,
	            "height": 0,
	            "fmt": ""
	        },
	        "gender": 1,
	        "promotionCode": "9SXVI8WX",
	        "loginStatus": false,
	        "loginTime": 0,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [],
	        "qq": {
	            "provider": "qq",
	            "oauthId": "231da3213da",
	            "nickName": "小呆",
	            "avatar": "http://1.jpg",
	            "token": "a23ca21354cad2321c231c"
	        },
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "id": "58209febd903d70e107cab74",
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_background.jpg",
	            "width": 400,
	            "height": 400,
	            "fmt": "jpg"
	        },
	        "createTime": 1478533099579,
	        "updateTime": 1478533099579,
	        "key": "279e601b0fb6ee63b5dd1acdc3e20b21c9bf0296193a1ae426067b5aebea37c3"
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

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

	"userId":1001

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
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

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
	    "timestamp": 1478513549001,
	    "code": 0,
	    "result": {
	        "id": "581c52918edd1f0f94b5b1b9",
	        "email": "381364134@qq.com",
	        "userId": 1,
	        "nickName": "不羁1",
	        "avatar": {
	            "id": "581c51c88edd1f0f94b5b1b1",
	            "url": "http://oe7hx2tam.bkt.clouddn.com/default_user_avatar.jpg",
	            "width": 100,
	            "height": 100,
	            "fmt": "jpg"
	        },
	        "gender": 1,
	        "promotionCode": "UG4LV8V9",
	        "loginStatus": true,
	        "loginTime": 1478510849981,
	        "logoutTime": 0,
	        "version": 0,
	        "roles": [],
	        "level": 1,
	        "soundNotify": true,
	        "vibrateNotify": true,
	        "backGround": {
	            "id": "581c51c88edd1f0f94b5b1b2",
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

###修改用户信息1010
- Path:/app/users/{userId}
- Request Method:PATCH
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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
	    "timestamp": 1478584187234,
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
	        "loginTime": 1478583857333,
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
	        "createTime": 0,
	        "updateTime": 0
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

***
#其他模块
###申请商家1012
- Path:/app/misc/sellers
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
101201|参数tel为空|没有传tel参数
101202|手机号格式不正确|手机号输入有误
101203|用户未登录|用户未登录

###用户反馈1013
- Path:/app/misc/feedback
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
101301|参数content为空|没有传content参数
101302|用户未登录|用户未登录

***
#首页运营模块
###取得专栏1014
- Path:/app/misc/columns
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--
itemId|String|是|无|主键
rank|Integer|是|无|排名
itemType|String|是|无|所属模块分类
columnType|String|是|无|所属的专栏分类
title|String|是|无|标题
linkType|String|是|"app"|链接分类
link|String|是|无|链接
desc|String|否|无|描述
cover|Object|否|无|封面图
url|String|是|""|图片链接
width|Integer|是|0|图片宽度
height|Integer|是|0|图片高度
fmt|String|否|无|图片格式
status|String|是|无|专栏状态

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"itemId":"546f2da8b8ce0440eddb287e",
				"rank":1,
				"itemType":"hotel",
				"columnType": "special",
				"title":"如家豪华酒店",
				"linkType":"app",
				"link":"http://XXX",
				"desc":"超大双人床",
				"cover": {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"status":"review"
			}
		]
	}

错误码|描述|原因
--|--|--
101401|运营专栏数据为空|没有录入运营专栏的数据

###取得首页1015
- Path:/app/misc/banners
- Request Method:GET
- Request Headers:无
- Query String:无
- Request Body
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
				"itemId":"546f2da8b8ce0440eddb287e",
				"itemType":"hotel",
				"title":"如家豪华酒店",
				"linkType":"app",
				"linkUrl":"http://XXX",
				"desc":"超大双人床",
				"cover": {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				}
			}
		]
	}

错误码|描述|原因
--|--|--

###取得商品列表(特产等)1016
- Path:/app/marketplace/columncommodities
- Request Method:GET
- Request Headers:无
- Query String:category=speciality&offset=0&limit=100
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
				"id":"546f2da8b8ce0440eddb287e",
				"category":["specialty"],
				"title":"煌上煌烤鸭",
				"cover": {
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"price":33.3,
				"marketPrice":56.4,
				"status":1,
				"salesVolume":100,
				"rating":0.98
			}
		]
	}

###取得商品详情(特产等)1017
- Path:/app/marketplace/commodities/{commodityId}
- Request Method:GET
- Request Headers:无
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
		"result":{
			"id":"546f2da8b8ce0440eddb287e",
			"category":["specialty"],
			"title":"煌上煌烤鸭",
			"locality": {
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
			"desc":"太好吃了",
			"cover": {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"images":[{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			}],
			"price":33.3,
			"marketPrice":56.4,
			"status":1,
			"plans":[{
				"planId":"646f2da8b8ce0440eddb287f",
				"title":"买3只送1只",
				"desc":"",
				"pricing":[],
				"marketPrice":56.4,
				"price":33.3,
				"stockInfo":[{
					"status":"plenty",
					"quantity":100,
					"timeRange":[]##  ##
				}],
				"timeRequired":false
			}],
			"salesVolume":100,
			"createTime":1450000000000,
			"updateTime":1450000000000,
			"rating":0.98,
			"commodityType":"食品",
			"version":1
		}
	}

错误码|描述|原因
--|--|--

###取得攻略列表1018
- Path:/app/columnguides
- Request Method:GET
- Request Headers:无
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
		"result":[{
			"id":"646f2da8b8ce0440eddb287f",
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"title":"南昌文化",
			"summary":"革命根据地，风景如画...",
			"viewCnt":1000000
		}]
	}

错误码|描述|原因
--|--|--

###取得攻略详情1019
- Path:/app/guides/{guideId}
- Request Method:GET
- Request Headers:无
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
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"cover" : {
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
			"updateTime":1425225600000,
			"title":"南昌文化",
			"desc":"南昌是一个...",
			"bestTripTime":"5月1日~10月20日",
			"tips":"",
			"hotels":[
				{
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
				}
			],
			"shoppings":[
				{
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
			],
			"restaurants":[
				{
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
				}
			],
			"activities": [
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
			"viewspots":[
				{
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
				}
			],
			"tripPlans":[
				{
					"id":"646f2da8b8ce0440eddb287f",
					"userId":10001,
					"nickName":"魔法屋",
					"avatar": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"tripItems": [
						{
							"tripTime":1450000000000,
							"createTime":1450000000000,
							"desc":"",
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
					]
				}
			],
			"summary":"革命根据地，风景如画...",
			"viewCnt":1000000
		}
	}

错误码|描述|原因
--|--|--

***
#POI模块
###取得客栈列表1020
- Path:/app/poi/hotels
- Request Method:GET
- Request Headers:无
- Query String:offset=0&limit=100
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
			}
		]
	}

错误码|描述|原因
--|--|--

###取得客栈详情1021
- Path:/app/poi/hotels/{hotelId}
- Request Method:GET
- Request Headers:无
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
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
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
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"contact": {
				"phoneList":["010-86752341","010-86752342"],
				"cellphoneList":["13811111111", "13811111112"],
				"qq":"13231235432",
				"weixin":"pisa",
				"sina":"a123123",
				"fax":"010-23131231",
				"email":"bujilvxing@163.com",
				"website":"www.baidu.com"
			},
			"zhName":"如家快捷酒店",
			"enName":"RuJia",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"priceDesc":"",
			"description":{
				"desc":"XXX",
				"details":"XXX",
				"tips":"XXX",
				"traffic":"XXX"
			},
			"tags":["",""],
			"alias":["",""],
			"targets":["",""],
			"address":"XXX",
			"locList":[
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
			],
			"saleVolume":100,
			"discount":0.65,
			"rentCar": {
				"price":180,
				"pickupAddr":{
					"province":"江西省",
					"city":"南昌市",
					"district":"东湖区",
					"detail":"XXX",
					"zipCode":"100071"
				},
				"returnAddr":{
					"province":"江西省",
					"city":"南昌市",
					"district":"东湖区",
					"detail":"XXX",
					"zipCode":"100071"
				},
				"contact":{
					"phoneList":["010-86752341","010-86752342"],
					"cellphoneList":["13811111111", "13811111112"],
					"qq":"13231235432",
					"weixin":"pisa",
					"sina":"a123123",
					"fax":"010-23131231",
					"email":"bujilvxing@163.com",
					"website":"www.baidu.com"
				},
				"minRentDay":1,
				"car":{
					"id":"646f2da8b8ce0440eddb287f",
					"carId":"赣N7023",
					"transmission":1,
					"vehicleType":"豪华型",
					"brand":"丰田",
					"carOwner":{
						"surname":"王",
						"givenName":"力宏",
						"gender":1,
						"birthday":"1982-12-03",
						"identities":[
							{
								"idType":"身份证",
								"number":"137771198212037145"
							}
						],
						"tel":{
							"dialCode":86,
							"number":"13811111111"
						}
					},
					"displacement":3.0,
					"seatNum":5,
					"name":"丰田霸道",
					"fuelType":"汽油",
					"gasolineType":"93#"
					"actuationType":"前后驱动",
					"dormer":true,
					"gps":true,
					"seatType":"织物座椅",
					"airbagNum":3,
					"gearboxType":"AT",
					"airConditioner":true
				},
				"autoInsurance":true,
				"autoInsurancePrice":50,
				"pickup":true
			},
			"locality": {
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
			"availableDays":[
				{
					"bookTime":"2016-08-01",
					"available":true,
					"price":392.1
				}
			]
		}
	}

错误码|描述|原因
--|--|--

###取得目的地列表1022
- Path:/app/localities
- Request Method:GET
- Request Headers:无
- Query String:offset=0&limit=100
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

错误码|描述|原因
--|--|--

###取得目的地详情1023
- Path:/app/localities/{localityId}
- Request Method:GET
- Request Headers:无
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

###取得景点列表1024
- Path:/app/poi/viewspots
- Request Method:GET
- Request Headers:无
- Query String:offset=0&limit=100
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
			}
		]
	}

错误码|描述|原因
--|--|--

###取得景点详情1025
- Path:/app/poi/viewspots/{viewspotId}
- Request Method:GET
- Request Headers:无
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
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
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
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"zhName":"八一广场",
			"enName":"",
			"url":"http://XXX",
			"marketPrice":280.6,
			"price":180.3,
			"priceDesc":"",
			"tags":["",""],
			"openTime":"9:00~21:00",
			"description":{
				"desc":"XXX",
				"details":"XXX",
				"tips":"XXX",
				"traffic":"XXX"
			},
			"alias":["",""],
			"targets":["",""],
			"locList":[
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
			],
			"locality":{
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
		}
	}

错误码|描述|原因
--|--|--

###取得餐厅列表1026
- Path:/app/poi/restaurants
- Request Method:GET
- Request Headers:无
- Query String:offset=1&limit=100
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
			}
		]
	}

错误码|描述|原因
--|--|--

###取得餐厅详情1027
- Path:/app/poi/restaurants/{restaurantId}
- Request Method:GET
- Request Headers:无
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
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"images":[
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			],
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"contact":{
				"phoneList":["010-86752341","010-86752342"],
				"cellphoneList":["13811111111", "13811111112"],
				"qq":"13231235432",
				"weixin":"pisa",
				"sina":"a123123",
				"fax":"010-23131231",
				"email":"bujilvxing@163.com",
				"website":"www.baidu.com"
			},
			"zhName":"煌上煌烤鸭店",
			"enName":"",
			"url":"http://XXX",
			"priceDesc":"",
			"price":180.3,
			"tags":["",""],
			"description":{
				"desc":"XXX",
				"details":"XXX",
				"tips":"XXX",
				"traffic":"XXX"
			},
			"alias":["",""],
			"targets":["",""],
			"locList":[
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
			],
			"locality":{
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
			"openTime":"9:00~21:00"
			"saleVolume":100
		}
	}

错误码|描述|原因
--|--|--

###取得商场列表1028
- Path:/app/poi/shoppings
- Request Method:GET
- Request Headers:无
- Query String:offset=1&limit=100
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
			}
		]
	}

错误码|描述|原因
--|--|--

###取得商场详情1029
- Path:/app/poi/shoppings/{shoppingId}
- Request Method:GET
- Request Headers:无
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
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
			"lat":180.1,
			"lng":180.1,
			"cover" : {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"images":[
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			],
			"rank":3,
			"hotness":0.97,
			"rating":0.98,
			"contact":{
				"phoneList":["010-86752341","010-86752342"],
				"cellphoneList":["13811111111", "13811111112"],
				"qq":"13231235432",
				"weixin":"pisa",
				"sina":"a123123",
				"fax":"010-23131231",
				"email":"bujilvxing@163.com",
				"website":"www.baidu.com"
			},
			"zhName":"煌上煌烤鸭店",
			"enName":"",
			"url":"http://XXX",
			"priceDesc":"",
			"price":180.3,
			"tags":["",""],
			"description":{
				"desc":"XXX",
				"details":"XXX",
				"tips":"XXX",
				"traffic":"XXX"
			},
			"alias":["",""],
			"targets":["",""],
			"locList":[
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
			],
			"locality":{
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
			"openTime":"9:00~21:00"
		}
	}

错误码|描述|原因
--|--|--

***
#活动模块
###发布活动1030
- Path:/app/activities
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"title":"北京冰雪嘉年华",
		"maxNum":200,
		"joinNum":100,
		"startTime":14500000000,
		"endTime":14500000000,
		"address":{
			"province":"江西省",
			"city":"南昌市",
			"district":"东湖区",
			"detail":"XXX",
			"zipCode":"100071"
		},
		"posters":[
			{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			}
		],
		"theme":"音乐",
		"category":"摇滚",
		"tags":["",""],
		"visiable":1,
		"desc":"",
		"tickets":[
			{
				"price":100.1,
				"free":false;
				"refundWay":1,
				"refundDesc":"",
				"desc":"",
				"maxNum":100
			}
		]
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--

###取得活动列表1031
- Path:/app/activities
- Request Method:GET
- Request Headers:无
- Query String:offset=1&limit=100
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
				"title":"北京冰雪嘉年华",
				"maxNum":200,
				"joinNum":100,
				"startTime":14500000000,
				"endTime":14500000000,
				"address":{
					"province":"江西省",
					"city":"南昌市",
					"district":"东湖区",
					"detail":"XXX",
					"zipCode":"100071"
				},
				"favorCnt":1000,
				"viewCnt":31231,
				"posters":[
					{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					}
				],
				"tags":["",""],
				"tickets":[
					{
						"id":"646f2da8b8ce0440eddb287f",
						"price":100.1,
						"free":false;
						"refundWay":1,
						"refundDesc":"",
						"desc":"",
						"maxNum":100
			
					}
				]
			}
		]
	}

错误码|描述|原因
--|--|--

###取得活动详情1032
- Path:/app/activities/{activityId}
- Request Method:GET
- Request Headers:无
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
		"result":{
				"id":"646f2da8b8ce0440eddb287f",
				"title":"北京冰雪嘉年华",
				"maxNum":200,
				"joinNum":100,
				"startTime":14500000000,
				"endTime":14500000000,
				"address":{
					"province":"江西省",
					"city":"南昌市",
					"district":"东湖区",
					"detail":"XXX",
					"zipCode":"100071"
				},
				"favorCnt":1000,
				"commentCnt":3123,
				"viewCnt":31231,
				"shareCnt":312,
				"posters":[
					{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					}
				],
				"theme":"音乐",
				"category":"摇滚",
				"tags":["",""],
				"visiable":1,
				"desc":"",
				"applicantInfos":[
					{
						"phoneList":["010-86752341","010-86752342"],
						"cellphoneList":["13811111111", "13811111112"],
						"qq":"13231235432",
						"weixin":"pisa",
						"sina":"a123123",
						"fax":"010-23131231",
						"email":"bujilvxing@163.com",
						"website":"www.baidu.com"
					}
				],
				"tickets":[
					{
						"id":"646f2da8b8ce0440eddb287f",
						"price":100.1,
						"free":false;
						"refundWay":1,
						"refundDesc":"",
						"desc":"",
						"maxNum":100
					}
				]
			}
		}
	}

错误码|描述|原因
--|--|--

***
#搜索模块
###搜索全部以及按分类搜索1032
包含：用户，形成规划，足迹，游记，美食，客栈，景点，购物，特产，婚纱摄影

- Path:/app/search
- Request Method:GET
- Request Headers:无
- Query String:user=false&tripplan=false

	没有参数的，表示搜索所有的分类。设置为false的表示不需要搜索该分类
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
		"result":{
			"userInfos":[
				{
					"userId":10001,
					"nickName":"魔法师",
					"avatar": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"gender":1
				}
			],
			"tripPlans":[
				{
					"id":"",
					"userId":"",
					"nickName":"",
					"avatar":{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"tripItems":[
						"id":"646f2da8b8ce0440eddb287f",
					"userId":10001,
					"nickName":"魔法屋",
					"avatar": {
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"tripItems": [
						{
							"tripTime":1450000000000,
							"createTime":1450000000000,
							"desc":"",
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
					]
				}
			],
			"traces":[],
			"restaurants":[
				{
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
				}
			],
			"hotels":[
				{
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
				}
			],
			"viewspots":[
				{
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
				}
			],
			"shoppings":[
				{
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
					"openTime":"9:00~21:00",
					"discount":0.65
				}
			],
			"specialities":[],
			"photographies":[]
		}
	}

错误码|描述|原因
--|--|--

***
#游记模块
###取得游记列表1033
- Path:/app/travelnotes
- Request Method:GET
- Request Headers:无
- Query String:offset=1&limit=100
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

###发布游记1034
- Path:/app/travelnotes
- Request Method:POST
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body


	{
		"id":"646f2da8b8ce0440eddb287f",
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
		"travelTime":14500000000,
		"contents":[
			{
				"day1":"XXX",
				"day2":"XXX"
			}
		],
		"summary":""
	}

- Response

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--

###修改游记1035
- Path:/app/travelnotes/{travelnoteId}
- Request Method:PUT
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

> 示例

	{
		"id":"646f2da8b8ce0440eddb287f",
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
		"travelTime":14500000000,
		"contents":[
			{
				"day1":"XXX",
				"day2":"XXX"
			}
		],
		"summary":""
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--

###取得游记详情1036
- Path:/app/travelnotes/{travelnoteId}
- Request Method:GET
- Request Headers:无
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
		"result":{
			"id":"646f2da8b8ce0440eddb287f",
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
			"rating":0.98,
			"hotness":0.98,
			"title":"",
			"publishTime":14500000000,
			"favorCnt":100,
			"commentCnt":10002,
			"viewCnt":1231,
			"shareCnt":123,
			"travelTime":14500000000,
			"contents":[
				{
					"day1":"XXX",
					"day2":"XXX"
				}
			],
			"summary":"",
			"source":"baidu",
			"essence":true
		}
	}

错误码|描述|原因
--|--|--

###删除游记1037
- Path:/app/travelnotes/{travelnoteId}
- Request Method:DELETE
- Request Headers

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
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--

***
#时间线模块
###查看朋友圈1038
- Path:/app/moments
- Request Method:GET
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:offset=1&limit=100
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
				"publishTime":14500000000,
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"originId":"646f2da8b8ce0440eddb287f",
				"originUserId":10002,
				"originNickName":"魔法屋",
				"originAvatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"text":"南昌如此之美",
				"images":[
					{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					}
				],
				"comment":"自己评论",
				"card":	{
					"id":"646f2da8b8ce0440eddb287f",
					"title":"",
					"summary":"",
					"cover":{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"thumb":{
						"width":400,
						"height":400,
						"url":"http://1.jpg"
					},
					"detailUrl":"http://xxx.html"
				}
			}
		]
	}

错误码|描述|原因
--|--|--

###发布朋友圈1039
- Path:/app/traces
- Request Method:POST
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

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
		"images":[
			{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			}
		],
		"comment":"自己评论",
		"card":	{
			"id":"646f2da8b8ce0440eddb287f",
			"title":"",
			"summary":"",
			"cover":{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"thumb":{
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"detailUrl":"http://xxx.html"
		}
	}
- Response
> 返回字段说明

字段名|类型|必含|默认值|描述
--|--|--|--|--

> 示例

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000
	}

***
#足迹模块
###发布用户足迹1040
- Path:/app/users/{userId}/traces/traces
- Request Method:POST
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

###修改足迹1041
- Path:/app/users/{userId}/traces/{traceId}
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

###删除用户足迹1042
- Path:/app/users/{userId}/traces/{traceId}
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

###取得足迹列表1043
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

###取得足迹详情1044
- Path:/app/users/{userId}/traces/{traceId}
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
#形成规划模块
###发布行程规划1045
- Path:/app/users/{userId}/tripplans
- Request Method:POST
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

> 示例

	{
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

###复制行程规划1046
- Path:/app/users/{userId}/tripplans/{tripPlanId}/copy
- Request Method:POST
- Request Headers

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

###取得行程规划列表1047
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
				"createTime":1450000000,
				"updateTime":1450000000,
				"shareCnt":100,
				"favorCnt":1000,
				"commentCnt":200,
				"viewCnt":100000,
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
		]
	}

错误码|描述|原因
--|--|--

###修改行程规划1048
- Path:/app/users/{userId}/tripplans/{tripPlanId}
- Request Method:PUT
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

> 示例

	{
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

###取得行程规划详情1049
- Path:/app/users/{userId}/tripplans/{tripPlanId}
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

###删除行程规划1050
- Path:/app/users/{userId}/tripplans/{tripPlanId}
- Request Method:DELETE
- Request Headers

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

---
#问题
###用户发布问题1051
- Path:/app/users/{userId}/quoras
- Request Method:POST
- Request Headers

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

> 示例

	{
		"source":"baidu",
		"topics":["湖边","摄影","婚纱"],
		"tags":["旅拍","XXX"],
		"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
		"contents":"如题"
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
			"source":"baidu",
			"topics":["湖边","摄影","婚纱"],
			"tags":["旅拍","XXX"],
			"viewCnt":100,
			"answerCnt":10,
			"maxVoteCnt":10000,
			"publishTime":145000000000,
			"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
			"contents":"如题",
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
	}

错误码|描述|原因
--|--|--

###取得问题详情1052
- Path:/app/quoras/{quoraId}
- Request Method:GET
- Request Headers:无
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
			"source":"baidu",
			"topics":["湖边","摄影","婚纱"],
			"tags":["旅拍","XXX"],
			"viewCnt":100,
			"answerCnt":10,
			"maxVoteCnt":10000,
			"publishTime":145000000000,
			"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
			"contents":"如题",
			"author": {
				"userId":10001,
				"nickName":"魔法师",
				"avatar": {
					"url":"http://1.jpg",
					"width":800,
					"height":600
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
					"contents":"XXX"
				}
			]
		}
	}

错误码|描述|原因
--|--|--

###取得用户问题列表1052
- Path:/app/users/{userId}/quoras
- Request Method:GET
- Request Headers:无
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
				"source":"baidu",
				"topics":["湖边","摄影","婚纱"],
				"tags":["旅拍","XXX"],
				"viewCnt":100,
				"answerCnt":10,
				"maxVoteCnt":10000,
				"publishTime":145000000000,
				"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
				"contents":"如题",
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

###取得问题列表1053
- Path:/app/quoras
- Request Method:GET
- Request Headers:无
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
				"source":"baidu",
				"topics":["湖边","摄影","婚纱"],
				"tags":["旅拍","XXX"],
				"viewCnt":100,
				"answerCnt":10,
				"maxVoteCnt":10000,
				"publishTime":145000000000,
				"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
				"contents":"如题",
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

###添加回答1054
- Path:/app/quoras/{quoraId}/answers
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

> 示例

	{
		"author":{
			"userId":10001,
			"nickName":"雷锋",
			"avatar":{
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			}
		},
		"title":"",
		"contents":"XXX"
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
			"source":"baidu",
			"topics":["湖边","摄影","婚纱"],
			"tags":["旅拍","XXX"],
			"viewCnt":100,
			"answerCnt":10,
			"maxVoteCnt":10000,
			"publishTime":145000000000,
			"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
			"contents":"如题",
			"author": {
				"userId":10001,
				"nickName":"魔法师",
				"avatar": {
					"url":"http://1.jpg",
					"width":800,
					"height":600
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
					"contents":"XXX"
				}
			]
		}
	}

错误码|描述|原因
--|--|--

---
#消息和社交
###关注用户1055
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
105501|参数followingId不可为空|没有传followingId参数
105502|用户未登录|用户未登录
105503|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###取消关注用户1056
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
105601|参数followingId不可为空|没有传followingId参数
105602|用户未登录|用户未登录

###用户的好友列表1057
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
105701|用户未登录|用户未登录

###获取好友(关注人)的详细信息1058
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
105801|用户未登录|用户未登录
105802|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###修改好友备注1059
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
105901|备注不可为空或者或字符串|没有传memo参数或者参数memo的值为空字符串
105902|用户未登录|用户未登录
105903|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###将用户加入黑名单1060
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
106001|屏蔽用户id不可为空|没有传blockId参数
106002|用户未登录|用户未登录
106003|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###将用户移除黑名单1061
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
106101|用户未登录|用户未登录
106102|用户不存在|1、输入有误，2、被关注的用户已经注销， 3、被关注的用户被非法删除

###用户的关注列表1062
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
106201|用户未登录|用户未登录

###用户的粉丝列表1063
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
106301|用户未登录|用户未登录

###发送消息1064
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

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"msgId":10,
			"senderId":10001,
			"senderNickName":"魔法师",
			"senderAvatar":{
				"width":600,
				"height":600,
				"url":"http://1.jpg"
			},
			"receiverId":10002,
			"contents":"您好！",
			"msgType":1,
			"chatType":1,
			"timestamp":145000000000,
			"conversionId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
		}
	}

错误码|描述|原因
--|--|--

###拉取消息1065
- Path:/app/users/{userId}/messages
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"purgeBefore":145000000000
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"msgId":10,
				"senderId":10001,
				"senderNickName":"魔法师",
				"senderAvatar":{
					c
				},
				"receiverId":10002,
				"contents":"您好！",
				"msgType":1,
				"chatType":1,
				"timestamp":145000000000,
				"conversionId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
			}
		]
	}

错误码|描述|原因
--|--|--

###修改会话属性1066
- Path:/app/users/{userId}/conversations/{conversationId}
- Request Method:PATCH
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"mute":1
	}

- Response

		{
			"code":0,
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

---
#群组
###取得会话属性1067
- Path:/app/users/{userId}/conversations/{conversationId}
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
			"muted":1,
			"pinned":1,
			"targetId":10001
		}
	}

错误码|描述|原因
--|--|--

###创建群组1068
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

###修改群组信息1069
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

###取得群组信息1070
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

###取得群组成员信息列表1071
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

###添加群成员1072
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
		"msg":"success",
		"timestamp":1425225600000
	}

###删除群成员1073
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

###取得用户群组列表1074
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

###发布帖子1075
- Path:/app/posts
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

###取得群帖子列表1076
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

---
#收藏
###收藏帖子1077
- Path:/app/users/{userId}/favorites/posts
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"title":"周游20国姑娘被亿万富豪持枪逼婚",
		"cover":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"summary":"还有这种事，求逼婚"
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取消收藏帖子1078
- Path:/app/users/{userId}/favorites/posts/{postId}
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

###收藏活动1079
- Path:/app/users/{userId}/favorites/activities
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"title":"潜水沙龙",
		"poster":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"desc":"潜水交流"
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取消收藏活动1080
- Path:/app/users/{userId}/favorites/activities/{activityId}
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

###收藏足迹1081
- Path:/app/users/{userId}/favorites/traces
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"cover":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"desc":"我在天安门广场,升旗仪式很威武"
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取消收藏足迹1082
- Path:/app/users/{userId}/favorites/traces/{traceId}
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

###收藏行程规划1083
- Path:/app/users/{userId}/favorites/tripplans
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"title":"我在天安门广场",
		"cover":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"desc":"升旗仪式很威武"
	}

- Response

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000
	}

错误码|描述|原因
--|--|--

###取消收藏行程规划1084
- Path:/app/users/{userId}/favorites/tripplans/{tripplanId}
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

###收藏问答1085
- Path:/app/users/{userId}/favorites/quoras
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"title":"现在的南昌冷吗？",
		"contents":"需要带什么衣服过去穿？"
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取消收藏问答1086
- Path:/app/users/{userId}/favorites/quoras/{quoraId}
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

###收藏美食1087
- Path:/app/users/{userId}/favorites/restaurants
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"zhName":"煌上煌烤鸭店",
		"openTime":"8：00~21：00",
		"cover":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		}
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取消收藏美食1088
- Path:/app/users/{userId}/favorites/restaurants/{restaurantId}
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

###收藏客栈1089
- Path:/app/users/{userId}/favorites/hotels
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"zhName":"不羁客栈",
		"openTime":"8：00~23：00",
		"cover":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		}
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取消收藏客栈1090
- Path:/app/users/{userId}/favorites/hotels/hotelId
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

###收藏游记1091
- Path:/app/users/{userId}/favorites/travelnotes
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
		"cover":{
			"width":600,
			"height":600,
			"url":"http://1.jpg"
		},
		"title":"井冈山一日游",
		"summary":"灰常漂亮"
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取消收藏游记1092
- Path:/app/users/{userId}/favorites/travelnotes/{travelnoteId}
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

###取得用户收藏列表1093
- Path:/app/users/{userId}/favorites
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
			"id":"",
			"userId":1001,
			"posts":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"周游20国姑娘被亿万富豪持枪逼婚",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"summary":"还有这种事，求逼婚"
				}
			],
			"traces":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"desc":"我在天安门广场,升旗仪式很威武"
				}
			],
			"tripPlans":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"我在天安门广场",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"desc":"升旗仪式很威武"
				}
			],
			"activities":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"潜水沙龙",
					"poster":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"desc":"潜水交流"
				}
			],
			"quoras":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"现在的南昌冷吗？",
					"contents":"需要带什么衣服过去穿？"
				}
			],
			"restaurants":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"煌上煌烤鸭店",
					"openTime":"8：00~21：00",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					}
				}
			],
			"hotels":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"不羁客栈",
					"openTime":"8：00~23：00",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
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
					"summary":"灰常漂亮"
				}
			]
		}
	}

错误码|描述|原因
--|--|--

---
#点赞
###点赞1094
- Path:/app/votes
- Request Method:POST
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

字段名|类型|必需|默认值|描述
--|--|--|--|--

	{
		"voteType":1,
		"targetId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
	}

- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result": {
			"id":"8c91a6deec8f42c9acfb0d1bd89dee9a"
			"userId":10001,
			"voteType":1,
			"targetId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
			"voteTime":145000000000000
		}
	}

错误码|描述|原因
--|--|--

###取消点赞1095
- Path:/app/votes/{voteId}
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

###取得点赞列表1096
- Path:/app/votes
- Request Method:GET
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:

字段名|类型|必需|默认值|描述
--|--|--|--|--

	voteType=1
	targetId=9c91a6deec8f42c9acfb0d1bd89dee9e

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
				"id":"8c91a6deec8f42c9acfb0d1bd89dee9a"
				"userId":10001,
				"voteType":1,
				"targetId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
				"voteTime":145000000000000
			}
		]
	}

错误码|描述|原因
--|--|--

###添加新评论1097
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

###删除评论1098
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

###取得评论列表1099
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

###用户搜索1100
- Path:/app/users
- Request Method:GET
- Request Headers:无
- Query String:?query=13811111111
- Request Body
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
				"userId":1001,
				"nickName":"魔法师",
				"avatar":{
					"width":600,
					"height":600,
					"url":"http://1.jpg"
				}
			}
		]
	}

###群组搜索1101
- Path:/app/chatgroups
- Request Method:GET
- Request Headers:无
- Query String:?query=10001
- Request Body
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":[
			{
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

###全站搜索1102
- Path:/app/search
- Request Method:GET
- Request Headers:无
- Query String:?query=青春下一站&viewspot=true&trace=true&tripplan=true&quora=true&activity=true&user=ture&post=true&travelNote=true&restaurant=true&hotel=true&sortby=publishTime&sort=asc

字段名|类型|必需|默认值|描述
--|--|--|--|--

- Request Body
- Response

字段名|类型|必含|默认值|描述
--|--|--|--|--

	{
		"code":0,
		"msg":"success",
		"timestamp":1425225600000,
		"result":{
			"viewspots":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"不羁客栈",
					"openTime":"8：00~23：00",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					}
				}
			],
			
			"users":[
				{
					"userId":1001,
					"nickName:"",
					"avatar":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					}
				}
			],
			"posts":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"周游20国姑娘被亿万富豪持枪逼婚",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"summary":"还有这种事，求逼婚"
				}
			],
			"traces":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"desc":"我在天安门广场,升旗仪式很威武"
				}
			],
			"tripPlans":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"我在天安门广场",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"desc":"升旗仪式很威武"
				}
			],
			"activities":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"潜水沙龙",
					"poster":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					},
					"desc":"潜水交流"
				}
			],
			"quoras":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"title":"现在的南昌冷吗？",
					"contents":"需要带什么衣服过去穿？"
				}
			],
			"restaurants":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"煌上煌烤鸭店",
					"openTime":"8：00~21：00",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
					}
				}
			],
			"hotels":[
				{
					"id":"9c91a6deec8f42c9acfb0d1bd89dee9e",
					"zhName":"不羁客栈",
					"openTime":"8：00~23：00",
					"cover":{
						"width":600,
						"height":600,
						"url":"http://1.jpg"
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
					"summary":"灰常漂亮"
				}
			]
		}
	}

###绑定邮箱1103
- Path:/app/users/{userId}/email
- Request Method:PUT
- Request Headers:

	"key":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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
110301|参数email为空|没有传email字段
110302|参数token为空|没有传token字段
110303|邮箱号格式不正确|邮箱号输入有误
110304|用户未登录|用户未登录
110305|邮箱号已存在|邮箱号已注册其他账号或者绑定其他账号
110306|令牌不合法|令牌不合法