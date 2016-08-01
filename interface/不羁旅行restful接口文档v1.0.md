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

>删除接口时，无需真正删除，只需更新资源状态即可，例如删除帖子，实际上是将帖子状态标记为删除状态。

###发送验证码
- Path:/app/validationcodes
- Request Method:POST
- Request Headers:无
- Query String:空
- Request Body

		{
			"tel":"13811111111",   // 可选，如果有tel，那么没有email字段
			"email":"381364134@qq.com", //可选，如果有email，那么没有tel字段
			"action":1
		}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000,
			"result":{
				"tel": {
					"number":"13811111111",
					"dialCode":86
				}
				"validationCode":"324512"
			}
		}

错误码|描述|原因
--|--|--
100101|手机号格式不正确|输入了错误的手机号
100102|邮箱格式不正确|输入了错误的邮箱号
100103|用户已存在|注册用户时，手机号已经注册了或者邮箱号已经注册过了
100104|请求次数过多|请求验证码的次数过多

###检验验证码
- Path:/app/tokens
- Request Method:POST
- Request Headers:无
- Query String:空
- Request Body

		{
			"tel":"13811111111",   // 可选，如果有tel，那么没有email字段
			"email":"381364134@qq.com", //可选，如果有email，那么没有tel字段
			"action":1,
			"validationCode":"022321"
		}

- Response

		{
			"code":0,
			"msg":"success",
			"timestamp":1425225600000,
			"result":{
				"tel":{
					"number":"13811111111",
					"dialCode":86
				},
				"token":"bjlx::token::eddf6dce-4dbd-41b2-9893-d0d3a5b7bcfa"
			}
		}

错误码|描述|原因
--|--|--
100201|手机号格式不正确|输入了错误的手机号
100202|邮箱格式不正确|输入了错误的邮箱号
100203|校验失败|验证码错误

###注册
- Path:/app/users
- Request Method:POST
- Request Headers:无
- Query String:无
- Request Body

		{
			"token":"bjlx::token::eddf6dce-4dbd-41b2-9893-d0d3a5b7bcfa",
			"tel":"13811111111",   // 可选，如果有tel，那么没有email字段
			"email":"381364134@qq.com", //可选，如果有email，那么没有tel字段
			"password":"312315"
		}

- Response

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
###登录
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###第三方登录
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###重置密码
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###修改密码
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得用户信息
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###修改用户信息
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###退出登录
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###绑定手机号
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###申请商家
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###用户反馈
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###用户定制
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得专栏
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###特产搜索
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得特产详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得攻略详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###查询客栈
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得客栈详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得目的地列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得目的地详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得景点列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得景点详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得餐厅列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得餐厅详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得商场列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得商场详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###发布活动
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得活动列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得活动详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###搜索全部
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###查询游记
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###查看朋友圈
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###发布足迹
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###修改足迹
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###删除用户足迹
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得足迹列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得足迹详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###发布行程规划
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###复制行程规划
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得行程规划列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###修改行程规划
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得行程规划详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###删除行程规划
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###发布游记
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###修改游记
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得游记详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###删除游记
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###发布问题
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###搜索问题
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得问题详情
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###添加回答
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###关注用户
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消关注用户
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###是否好友关系
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###用户的好友列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###获取好友详细信息
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###修改好友备注
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###将用户加入/移除黑名单
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###用户的关注列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###发送消息
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###拉取消息
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###修改会话属性
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得会话属性
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###创建群组
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###修改群组信息
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得群组信息
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得群组成员信息列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###操作群成员
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得用户群组列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###发布帖子
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得群帖子列表
按照最新评论时间倒序
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###收藏帖子
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消收藏帖子
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得收藏帖子列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###收藏足迹
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消收藏足迹
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得收藏足迹列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###收藏行程规划
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消收藏行程规划
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得收藏行程规划列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###收藏问答
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消收藏问答
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得收藏问答列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###收藏美食
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消收藏美食
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得收藏美食列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###收藏客栈
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消收藏客栈
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得收藏客栈列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###收藏游记
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消收藏游记
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得收藏游记列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###点赞帖子
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消点赞帖子
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得帖子点赞列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###点赞行程规划
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消点赞行程规划
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得行程规划点赞列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###点赞足迹
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消点赞足迹
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得足迹点赞列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###点赞活动
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消点赞活动
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得活动点赞列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###点赞游记
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取消点赞游记
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得游记点赞列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###投票问题
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###投票回答
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###添加新评论
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###删除评论
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--

###取得评论列表
- Path:/app/
- Request Method:
- Request Headers:
- Query String:
- Request Body
- Response

错误码|描述|原因
--|--|--
