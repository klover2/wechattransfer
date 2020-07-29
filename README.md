# wechattransfer
配置回调函数
我们在微信客户端访问第三方网页（即我们自己的网页）的时候，我们可以通过微信网页授权机制，我们不仅要有前面获取到的appid和appsecret还需要有当用户授权之后，回调的域名设置，即用户授权后，页面会跳转到哪里。具体的配置如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729150845785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzExMDYwOQ==,size_16,color_FFFFFF,t_70)



请求授权页面的构造方式：
>https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect


|参数|必须|说明|
|--|--|--|
|appid|是|公众号的唯一标识|
|redirect_uri|是|授权后重定向的回调链接地址|
|response_type|是|返回类型，请填写code|
|scope|是|应用授权作用域，snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），snsapi_userinfo （弹出授权页面，可通过openid拿到昵称、性别、所在地。并且，即使在未关注的情况下，只要用户授权，也能获取其信息）|
|state|否|重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节，该值会被微信原样返回，我们可以将其进行比对，防止别人的攻击。|
|#wechat_redirect|否|直接在微信打开链接，可以不填此参数。做页面302重定向时候，必须带此参数|

```bash
应用授权作用域：由于snsapi_base只能获取到openid，意义不大，所以我们使用snsapi_userinfo。

参考链接如下：（域名是你配置的回调函数域名）
>https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx3d0b47a2e903d8e7&redirect_uri=https://域名/wechattransfer.html&response_type=code&scope=snsapi_userinfo&state=https://域名/index.html#wechat_redirect

`https://域名/wechattransfer.html`是授权后会跳转的地址,在这个页面中会解析路径参入从而进入`https://域名/index.html`（[wechattransfer下载链接](https://github.com/wjc49420645/wechattransfer)）

下载wechattransfer.html之后配置在你的服务器上，在微信授权下的域名能够访问到
`https://域名/index.html`是要进入的页面



使用微信开发工具访问上面参考链接，最后会进入下面链接，同时带上code
https://域名/index.html?code=0014rK302cHWcV0w7K502IhK3024rK3u
```


把当前code发送给服务器
```bash
const got = require('got');
(async()=>{
    let appid= ''; // 公众号appid
    let appsecret='';// 公众号appsecret

let WxLoginInfo = (await got.get(`https://api.weixin.qq.com/sns/oauth2/access_token?appid=${appid}&secret=${appsecret}&code=${code}&grant_type=authorization_code`, { 'json': true })).body;
        // 获取微信中的用户信息
let { access_token, openid } = WxLoginInfo || {};
        // 通过微信获取用户信息
let wxUserInfo = (await got.get(`https://api.weixin.qq.com/sns/userinfo?access_token=${access_token}&openid=${openid}`, { 'json': true })).body;
})()

```


注意：
使用微信调试工具调试必须得有当前公众号开发者权限[如何添加](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Web_Developer_Tools.html)


UnionID机制的作用说明：如果开发者拥有多个移动应用、网站应用和公众帐号，可通过获取用户基本信息中的unionid来区分用户的唯一性，因为同一用户，对同一个微信开放平台下的不同应用（移动应用、网站应用和公众帐号），unionid是相同的。
