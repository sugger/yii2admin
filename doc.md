1.通用规则

>1.1 本接口为RESTful风格的接口文档，基本功能和接口访问的格式如下例子：
- GET /users: 逐页列出所有游戏
- POST /users: 创建一个新用户
- GET /users/123: 返回用户 123 的详细信息
- PATCH /users/123 and PUT /users/123: 更新用户123
- DELETE /users/123: 删除用户123


>1.2 通用响应头包含了接口的分页数据可在url上加入参数，用page控制页数，用per-page控制每页显示的数据个数。响应后信息中的分页信息分别表示为：
- X-Pagination-Total-Count: 资源所有数量;
- X-Pagination-Page-Count: 页数;
- X-Pagination-Current-Page: 当前页(从1开始);
- X-Pagination-Per-Page: 每页资源数量;
- Link: 允许客户端一页一页遍历资源的导航链接集合.

>1.3 REST框架的HTTP状态代码:
- 200: OK。一切正常。
- 201: 响应 POST 请求时成功创建一个资源。Location header 包含的URL指向新创建的资源。
- 204: 该请求被成功处理，响应不包含正文内容 (类似 DELETE 请求)。
- 304: 资源没有被修改。可以使用缓存的版本。
- 400: 错误的请求。可能通过用户方面的多种原因引起的，例如在请求体内有无效的JSON 数据，无效的操作参数，等等。
- 401: 验证失败。
- 403: 已经经过身份验证的用户不允许访问指定的 API 末端。
- 404: 所请求的资源不存在。
- 405: 不被允许的方法。 请检查 Allow header 允许的HTTP方法。
- 415: 不支持的媒体类型。 所请求的内容类型或版本号是无效的。
- 422: 数据验证失败 (例如，响应一个 POST 请求)。 请检查响应体内详细的错误消息。
- 429: 请求过多。 由于限速请求被拒绝。
- 500: 内部服务器错误。 这可能是由于内部程序错误引起的。

>1.4 用户权限验证

采用BearerAuth验证，在信息中加入请求头
Authorization:Bearer XXXX
其中 XXXX为创建Token接口响应的BearerToken（access_token）
如果access_token正确会正确的处理请求，如不正确则会返回401的状态码，并响应JSON格式的错误信息
{
    "name": "Unauthorized",
    "message": "Your request was made with invalid credentials.",
    "code": -1,
    "status": 401
}
其中code对应的错误信息状态码含义分别为
0:用户不存在或已被冻结
-1:无效的AccessToken 
-2:过期的AccessToken 
过期的AccessToken需要通过user-token接口更新

>1.5 资源获取规则

通过Url添加参数的方式
例如通过Get方法访问http://{接口域名}/user接口
响应默认的数据为
```
{
    "uid": "2",
    "username": "user002",
    "nickname": "点爬",
    "is_18": 0,
    "total_money": 0,
    "reg_time": "1501814354",
    "last_login_time": "1514340649",
    "update_time": "2017-12-27 10:10:49",
    "image": "",
    "score": "100",
    "score_all": "100",
    "regIp": "127.0.0.1",
    "lastLoginIp": "127.0.0.1"
}
```
如果只想要lastLoginIp和nickname则url可加fields参数，url地址写为http://{接口域名}/user?fields=lastLoginIp,nickname则响应的数据变为
```
{
    "nickname": "点爬",
    "lastLoginIp": "127.0.0.1"
}
```
另外一些接口提供拓展数据，这些拓展数据一般情况是不输出的，只有在用expand参数指定拓展数据的字段名，例如http://{接口域名}/user?fields=lastLoginIp,nickname&expand=money,email则响应数据为
```
{
    "nickname": "点爬",
    "lastLoginIp": "127.0.0.1",
    "email": "user002@mail.com",
    "money": 0
}
```
***
 2.接口及参数说明
>2.1 Token接口

- Url:   http://{接口域名}/user-token
- 注:更新密钥的ID为access_token,注销密钥的ID为refresh_token
拓展参数：

参数|说明
:---:|:---:
user|用户信息的JSON对象
createTime|Token创建的时间
ip|创建Token时的ip地址
os|创建Token时的客户端操作系统
client|创建Token时的客户端浏览器
2.1.1 创建Token （POST）
requestBody参数:

参数|示例|说明
:---:|:---:|:---:
account|user002|账号
password|q12355|密码

responseBody:
```
{
    "refresh_token": "ZTBjMWQ0ODcwY2NlMmI3ZGIzNmY1ZWJh",/*申请新access_token及注销token所需密钥*/
    "access_token": "TV!N@I5a4339e07b311",/*BearerToken*/
    "uid": "2",//用户ID
    "expire": "1800"//过期时间
}
```
2.1.2 更新access_token(PUT,PATCH)
url:http://{接口域名}/user-token/TV!N@I5a4339e07b311
requestBody参数:

参数|示例|说明
:---:|:---:|:---:
randStr|DcwY2NlMmI3ZGIz|随机字符串（最大32位）
sign|37aa65a1237131ac41081192a3abba7e|密钥加密规则 MD5（refresh_token + randStr + access_token）
responseBody:
```
{
    "refresh_token": "ZTBjMWQ0ODcwY2NlMmI3ZGIzNmY1ZWJh",/*申请新access_token及注销token所需密钥*/
    "access_token": "WM&Mz=5a43182c27a5c",/*BearerToken*/
    "uid": "2",//用户ID
    "expire": "1800"//过期时间
}
```
2.1.3 注销Token（DELETE）
无参数无响应，状态码为204注销成功

> 2.2 游戏数据获取接口

2.2.1 游戏列表（所有游戏按权重排序）
url:http://{接口域名}/game
搜索的额外参数

键|描述|代表含义
:--:|:--:|:--:
type| 类型|（数值代表的含义未定）
theme| 题材|（数值代表的含义未定）
form| 形式|（数值代表的含义未定）
ucfirst| 首字母|（1:[a-e]  2:[f-k] 3:[l-p] 4:[q-u]  5:[v-z]）

responseBody:
```
[
    {
        "game_players": 4,
        "game_hit": 0,
        "id": 96,
        "name": "操戈天下",
        "gamestyle": 96,
        "pic": null,
        "game_web": "",
        "gametype": "页游"
    },
    {
        "game_players": 0,
        "game_hit": 0,
        "id": 102,
        "name": "传奇霸业",
        "gamestyle": 102,
        "pic": {
            "pic1": "",
            "pic2": "",
            "pic3": "",
            "pic4": "",
            "pic5": "",
            "pic6": "",
            "pic7": "",
            "pic8": "",
            "pic9": ""
        },
        "game_web": "cqby.kukewan.com",
        "gametype": "页游"
    },
···
]
```
2.2.2 推荐游戏列表
url:http://{接口域名}/game/recommend
响应数据同文档所述的上个接口
2.2.3 
url:http://{接口域名}/game/good
响应数据同文档所述的上个接口
```
[
    {
        "game_players": 4,
        "game_hit": 0,
        "id": 96,
        "name": "操戈天下",
        "gamestyle": "休闲益智",
        "pic": null,
        "game_web": "",
        "gametype": "页游"
    },
    {
        "game_players": 0,
        "game_hit": 0,
        "id": 102,
        "name": "传奇霸业",
        "gamestyle": "休闲益智",
        "pic": {
            "pic1": "",
            "pic2": "",
            "pic3": "",
            "pic4": "",
            "pic5": "",
            "pic6": "",
            "pic7": "",
            "pic8": "",
            "pic9": ""
        },
        "game_web": "cqby.kukewan.com",
        "gametype": "页游"
    },
···
]
```
2.2.4 登录游戏
2.2.5 查询游戏角色信息

> 2.3 获取区服数据


拓展参数：

参数|说明
:---:|:---:
player_num|玩家数量
stop_notice|停服通知
content|区服内容介绍
server_img|区服图片
server_sn|游戏的区服编号

2.3.1 获取去区服列表
url:http://{接口域名}/server
GET参数

参数|说明
:---:|:---:
gid|游戏ID
recommend|是否推荐（0 or 1）
openstatus|区服开启状态 (0为所有状态  1新服预告  2已开新服)

responseBody:
````
[
    {
        "sid": 3963,
        "gid": 104,
        "serverName": "双线18区",
        "gameName": "射雕英雄传",
        "type": "休闲益智",
        "serverSn": "18",
        "isRecommend": 1,
        "isStop": 0,
        "startTime": "2017-12-22 16:50:45"
    },
    {
        "sid": 3958,
        "gid": 105,
        "serverName": "双线6区",
        "gameName": "烈火永恒",
        "type": "休闲益智",
        "serverSn": "6",
        "isRecommend": 0,
        "isStop": 0,
        "startTime": "2017-12-14 13:00:00"
    },
····
]
````
2.3.2 获取单个区服信息
url:http://{接口域名}/server/{区服ID}
> 2.4 文章

url:http://{接口域名}/article
拓展参数：

参数|说明
:---:|:---:
tag|标签
content|文章内容
metaTitle|seo网页标题
keyWords|关键词
catLink|栏目链接

2.4.1获取文章列表

参数|说明
:---:|:---:
catid|栏目ID
tag|是否推荐（标签名称）

responseBody:
```
[
    {
        "id": "3",
        "category_id": "1",
        "title": "帆海汇介绍",
        "cover": " http://backed.yii.com:88/201610/1477567061651.png",
        "description": "帆海汇介绍",
        "extend": "",
        "link": "",//文章链接，有链接点击跳转到此链接，没有则通过接口获取数据
        "up": "0",//支持人数
        "down": "0",//反对人数
        "view": "0",//阅读人数
        "time": "1473606822",
        "catTitle": "关于我们"
    }
]
```
2.4.2 获取文章详情

url:http://{接口域名}/article/{文章ID}

> 2.6 网站通用接口

2.6.1 网站配置信息
2.6.2 检测（身份证号、真实姓名、邮箱、用户名检测、手机号检测）
2.6.3 手机验证码发送
2.6.4 获取图片验证码
> 2.7 用户信息

2.7.1 注册
2.7.2 修改信息
2.7.3 查询信息
>2.8 充值

2.8.1 获取支付渠道列表
2.8.2 下单
2.8.3 付款
2.8.4 查询订单

>2.9 商品

2.9.1 商品列表
2.9.2 兑换商品
2.9.3 商品详情









