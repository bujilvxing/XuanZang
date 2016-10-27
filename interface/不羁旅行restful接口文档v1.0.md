#不羁旅行restful接口文档v1.0
少量的接口(查询接口)，无需登录，游客用户即可操作。大部分接口(比如：与用户相关的接口)需要登录才能操作。
###header
HTTP请求需要设置以下Header:

- Accept:application/vnd.bjlx.v1+json。随着系统的开发，接口可能出现不同的版本，因此在这个header中携带接口版本信息，v1表示第一个版本。
- Accept-Encoding:gzip。可支持gzip压缩格式，减小数据传输。
- bjlxToken:不羁旅行自定义Header。若用户登录，则需要指定该Header，用于获取用户的信息以及部分接口的登录权限验证。

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
action|Integer|是|无|1表示新用户注册；2表示用户绑定手机号；3表示用户找回密码
> 示例

		{
			"account":"13811111111",   // 可以是手机号，也可以是邮箱号
			"action":1
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
			"result":{
				"account": "13811111111",
				"validationCode":"324512"
			}
		}

错误码|描述|原因
--|--|--
100101|手机号格式不正确|输入了错误的手机号
100102|邮箱格式不正确|输入了错误的邮箱号
100103|用户已存在|注册用户时，手机号已经注册了或者邮箱号已经注册过了
100104|请求次数过多|请求验证码的次数过多

###检验验证码1002
- Path:/app/tokens
- Request Method:POST
- Request Headers:无
- Query String:空
- Request Body

		{
			"account":"13811111111",   // 可以是手机号，也可以是邮箱号
			"action":1,
			"validationCode":"022321"
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
			"result":{
				"tel":{
					"number":"13811111111",
					"dialCode":86
				},// 该字段有可能是邮箱号，email
				"token":"bjlx::token::eddf6dce-4dbd-41b2-9893-d0d3a5b7bcfa"
			}
		}

错误码|描述|原因
--|--|--
100201|手机号格式不正确|输入了错误的手机号
100202|邮箱格式不正确|输入了错误的邮箱号
100203|校验失败|验证码错误

###注册1003
- Path:/app/users
- Request Method:POST
- Request Headers:无
- Query String:无
- Request Body

		{
			"token":"bjlx::token::eddf6dce-4dbd-41b2-9893-d0d3a5b7bcfa",
			"account":"13811111111",   // 可以是手机号，也可以是邮箱号
			"password":"312315"
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
			"result":{
				"id":"",
				"userId":10001,
				"nickName":"不羁客10001",
				"avatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"tel":{
					"number":"13811111111",
					"dialCode":86
				}
				"email":"",
				"level":1,
				"soundNotify":true,
				"vibrateNotify":true
			}
		}
错误码|描述|原因
--|--|--
100301|XXX|XXX
###登录1004
- Path:/app/users/login
- Request Method:POST
- Request Headers:无
- Query String:无
- Request Body

		{
			"loginName":"13811111111",   // 可以是手机号，也可以是邮箱号
			"password":"ABCabc123"
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
			"result":{
				"id":"557049120c2022abe1acf0a1",
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"gender":1, // 1表示未选择，2表示男，3表示女
				"signature":"前世的乡愁",
				"tel":{
					"number":"13811111111",
					"dialCode":86
				},
				"email":"",
				"promotionCode":"N2A2MV",
				"roles":[1, 2],
				"residence":"北京市海淀区闵庄路15号",
				"birthday":"1990-06-01",
				"oauthInfoList":[
					{
						"provider":"qq",
						"oauthId":"231da3213da",
						"nickName":"小呆",
						"avatar":"http://1.jpg",
						"token":""
					}
				],
				"level":1,
				"zodiac":1,
				"soundNotify":true,
				"vibrateNotify":true,
				"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
			}
		}
错误码|描述|原因
--|--|--

###第三方登录1005
- Path:/app/users/oauthlogin
- Request Method:POST
- Request Headers:无
- Query String:无
- Request Body

		{
			"provider":"qq",
			"oauthId":"231da3213da",
			"nickName":"小呆",
			"avatar":"http://1.jpg",
			"token":""
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
			"result":{
				"id":"557049120c2022abe1acf0a1",
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"gender":1, // 1表示未选择，2表示男，3表示女
				"signature":"前世的乡愁",
				"tel":{},
				"email":"",
				"promotionCode":"N2A2MV",
				"roles":[1, 2],
				"residence":"北京市海淀区闵庄路15号",
				"birthday":"1990-06-01",
				"oauthInfoList":[
					{
						"provider":"qq",
						"oauthId":"231da3213da",
						"nickName":"小呆",
						"avatar":"http://1.jpg",
						"token":""
					}
				],
				"level":1,
				"zodiac":1,
				"soundNotify":true,
				"vibrateNotify":true,
				"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
			}
		}

错误码|描述|原因
--|--|--

***
#用户模块
###重置密码1006
- Path:/app/users/password
- Request Method:PUT
- Request Headers:无
- Query String:无
- Request Body

		{
			"loginName":"13811111111",
			"newPassword":"123343",
			"token":"bjlx::token::9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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

错误码|描述|原因
--|--|--

###修改密码1007
- Path:/app/users/{userId}/password
- Request Method:PUT
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

		{
			"oldPassword":"das123",
			"newPassword":"123344"
		}
- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取得用户信息1008
- Path:/app/users/{userId}
- Request Method:GET
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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
				"id":"557049120c2022abe1acf0a1",
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"gender":1, // 1表示未选择，2表示男，3表示女
				"signature":"前世的乡愁",
				"tel":{},
				"email":"",
				"promotionCode":"N2A2MV",
				"roles":[1, 2],
				"residence":"北京市海淀区闵庄路15号",
				"birthday":"1990-06-01",
				"oauthInfoList":[
					{
						"provider":"qq",
						"oauthId":"231da3213da",
						"nickName":"小呆",
						"avatar":"http://1.jpg",
						"token":""
					}
				],
				"level":1,
				"zodiac":1,
				"soundNotify":true,
				"vibrateNotify":true,
				"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
			}
		}

错误码|描述|原因
--|--|--

###修改用户信息1009
- Path:/app/users/10001
- Request Method:PATCH
- Request Headers

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

		{
			"nickName":"魔法师",
			"gender":2,
			"birthday":"1990-06-01",
			"signature":"前世的乡愁",
			"residence":"北京市海淀区闵庄路15号",
			"backGround": {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
			},
			"avatar": {
				"width":400,
				"height":400,
				"url":"http://1.jpg"
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
			"timestamp":1425225600000,
			"result":{
				"id":"557049120c2022abe1acf0a1",
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"gender":1, // 1表示未选择，2表示男，3表示女
				"signature":"前世的乡愁",
				"tel":{},
				"email":"",
				"promotionCode":"N2A2MV",
				"roles":[1, 2],
				"residence":"北京市海淀区闵庄路15号",
				"birthday":"1990-06-01",
				"oauthInfoList":[
					{
						"provider":"qq",
						"oauthId":"231da3213da",
						"nickName":"小呆",
						"avatar":"http://1.jpg",
						"token":""
					}
				],
				"level":1,
				"zodiac":1,
				"soundNotify":true,
				"vibrateNotify":true,
				"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
			}
		}

错误码|描述|原因
--|--|--

###退出登录1010
- Path:/app/users/logout
- Request Method:POST
- Request Headers

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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

###绑定手机号1011
- Path:/app/users/10001/tel
- Request Method:PUT
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

		{
			"tel":"13811111111",
			"token":"bjlx::token::eddf6dce-4dbd-41b2-9893-d0d3a5b7bcfa"
		}
- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

***
#其他模块
###申请商家1012
- Path:/app/misc/sellers
- Request Method:POST
- Request Headers

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

		{
			"tel":"13811111111"
		}
- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###用户反馈1013
- Path:/app/misc/feedback
- Request Method:POST
- Request Headers

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

		{
			"content":""
		}
- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

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

> 示例

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000,
			"result":[{
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
			}]
		}

错误码|描述|原因
--|--|--

###取得首页1015
- Path:/app/misc/columns
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
			"result":[{
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
			}]
		}

错误码|描述|原因
--|--|--

###取得商品列表(特产等)1016
- Path:/app/marketplace/commodities
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
- Path:/app/guides
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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body
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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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

***
#时间线模块
###查看朋友圈1038
- Path:/app/moments
- Request Method:GET
- Request Headers

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String


- Request Body
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"
- Query String:无
- Request Body

	{
		"source":"baidu",
		"topics":["湖边","摄影","婚纱"],
		"tags":["旅拍","XXX"],
		"title":"上哪里找影楼去鄱阳湖拍婚纱照？",
		"contents":"如题"
	}
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

###关注用户1055
- Path:/app/users/{userId}/follows
- Request Method:POST
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

		{
			"concernedId":10001
		}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取消关注用户1056
- Path:/app/users/{userId}/follows
- Request Method:DELETE
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

		{
			"concernedId":10001
		}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###用户的好友列表1057
- Path:/app/users/{userId}/contacts
- Request Method:GET
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

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
					"memo":"备注信息"
				}
			]
		}

错误码|描述|原因
--|--|--

###获取好友详细信息1058
- Path:/app/users/{userId}/contacts/{contactId}
- Request Method:GET
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000,
			"result":{
				"id":"557049120c2022abe1acf0a1",
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"gender":1, // 1表示未选择，2表示男，3表示女
				"signature":"前世的乡愁",
				"residence":"北京市海淀区闵庄路15号",
				"birthday":"1990-06-01",
				"level":1,
				"zodiac":1,
				"memo":"备注信息"
			}
		}

错误码|描述|原因
--|--|--

###修改好友备注1059
- Path:/app/users/{userId}/contacts/{contactId}/memos
- Request Method:PUT
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

		{
			"memo":"备注"
		}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000,
			"result":{
				"id":"557049120c2022abe1acf0a1",
				"userId":10001,
				"nickName":"魔法师",
				"avatar":{
					"width":400,
					"height":400,
					"url":"http://1.jpg"
				},
				"gender":1, // 1表示未选择，2表示男，3表示女
				"signature":"前世的乡愁",
				"residence":"北京市海淀区闵庄路15号",
				"birthday":"1990-06-01",
				"level":1,
				"zodiac":1,
				"memo":"备注信息"
			}
		}

错误码|描述|原因
--|--|--

###将用户加入黑名单1060
- Path:/app/users/{userId}/blacklist
- Request Method:POST
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

	{
		"blockId":10001
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###将用户移除黑名单1061
- Path:/app/users/{userId}/blacklist/{blockId}
- Request Method:DELETE
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

###用户的关注列表1062
- Path:/app/users/{userId}/followings
- Request Method:GET
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String

		offset:0
		limit:100

- Request Body:无
- Response

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

###用户的粉丝列表1063
- Path:/app/users/{userId}/followers
- Request Method:GET
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String

		offset:0
		limit:100

- Request Body:无
- Response

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

###发送消息1064
- Path:/app/messages
- Request Method:POST
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

		{
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
			"conversionId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
		}

- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

		{
			"purgeBefore":145000000000
		}

- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

	{
		"mute":1
	}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

错误码|描述|原因
--|--|--

###取得会话属性1067
- Path:/app/users/{userId}/conversations/{conversationId}
- Request Method:GET
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

		{
			"memberId":10002
		}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000
		}

###删除群成员1073
- Path:/app/chatgroups/{chatgroupId}/members/{memberId}
- Request Method:DETELE
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

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

###收藏帖子1077
- Path:/app/users/{userId}/favorites/posts
- Request Method:POST
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body:无
- Response

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

###点赞1094
- Path:/app/votes
- Request Method:POST
- Request Headers:

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

		{
			"voteType":1,
			"targetId":"9c91a6deec8f42c9acfb0d1bd89dee9e"
		}

- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:

		voteType=1
		targetId=9c91a6deec8f42c9acfb0d1bd89dee9e

- Request Body:无
- Response

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:无
- Request Body

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

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

	"bjlxToken":"9c91a6de-ec8f-42c9-acfb-0d1bd89dee9e"

- Query String:

		commentType=1
		itemId=9c91a6deec8f42c9acfb0d1bd89dee9e

- Request Body:无
- Response

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
- Request Body
- Response

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
