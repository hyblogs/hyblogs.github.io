# 微信小程序用户登录&获取用户信息(UnionId)



&emsp;&emsp;前面记录了微信公众号里面的自定义分享是如何一步一步实现的，今天咋们来试试微信小程序里面的用户登录及获取用户信息是如何实现的。重点是如何获取用户信息中的UnionId，这估计是很多同学的共同疑问。

### 微信小程序用户登录

微信小程序的用户登录流程如下：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1590245240076.png)

> 如上流程图来自微信官方文档，地址：https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html

> **大概步骤**：

> 1、调用 wx.login() 获取 临时登录凭证code ，并回传到开发者服务器。
> 2、调用 auth.code2Session 接口，换取 用户唯一标识 OpenID 和 会话密钥 session_key。

> 其中步骤1是由前端调用 JsApi 完成的，步骤2是由后端调用微信服务端 API 完成的。

> 注：由于我们等会儿需要获取用户信息，所以要先获取用户的授权，前端可以通过调用 JsApi 接口 wx.authorize 实现用户授权，具体参见微信官方文档，地址：https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/authorize.html 其中 scope 至少需要传入 scope.userInfo 以便后面获取用户信息所需。

接下来咋们步入正题，实现微信用户登录。

如上面步骤所述，需要前端调用微信 JsApi 接口 wx.login(Object object)
调用接口获取登录凭证（code）。通过凭证进而换取用户登录态信息，包括用户的唯一标识（openid）及本次登录的会话密钥（session_key）等。用户数据的加解密通讯需要依赖会话密钥完成。更多使用方法详见微信官方文档：
https://developers.weixin.qq.com/miniprogram/dev/api/open-api/login/wx.login.html

后端需要写接口接受前端获取到的临时登录凭证 code 并去微信服务器换取用户登录态信息。其中，后端需要调用的API接口地址为：https://api.weixin.qq.com/sns/jscode2session?appid=%s&secret=%s&js_code=%s&grant_type=authorization_code
里面需要传入小程序的 APP_ID、APP_SECRET 以及前端获取并传入的临时 code 。

```JAVA
/**
 * 用临时登录凭证 code 换取微信登录态信息及session_key
 *
 * @param jsCode 临时登录凭证 code
 * @return ticket
 */
public void getSessionKey(String jsCode) throws MfException {
    Result<JSONObject> result = HttpUtil.get(String.format(jscode2sessionUrl, WX_APPLET_APPID, WX_APPLET_SECRET, jsCode));
    LogHelper.print("获取微信小程序SessionKey结果", result);
}
```

以上代码便是通过发送 Http-GET 请求获取 session_key 等信息，具体的 HTTP 请求工具可根据实际项目所需进行使用。

这里需要注意以下两点：
1、临时登录凭证 code 只能使用一次；
2、会话密钥 session_key 是对用户数据进行**加密签名**的密钥。为了应用自身的数据安全，开发者服务器**不应该把会话密钥下发到小程序，也不应该对外提供这个密钥。**

此处，建议大家将获取到的 session_key 等信息进行缓存处理，因为后面解密用户信息的时候是需要用到 session_key 的。例如：

> 将获取到的 session_key 等信息存入 Redis 缓存一定时间，然后将缓存的 key 值返回给前端，前端提交用户密文信息的时候顺带将此 key 一并传入，然后我们去 Redis 取出 session_key 对用户信息密文进行解密。

到此，微信用户登录小程序算是完成啦！

### 获取用户信息(UnionId)

&emsp;&emsp;只是获取到用户的登录态信息及会话密钥显然是不够的，很多时候我们业务上需要使用到用户的昵称、头像等信息，接下来咋们将实现对用户基本信息的获取。

&emsp;&emsp;由于前面已经提到，获取用户信息必须要有用户的授权。如果在用户未授权过的情况下调用 wx.getUserInfo(Object object) 接口，将不再出现授权弹窗，会直接进入 fail 回调。微信官方文档说明：https://developers.weixin.qq.com/miniprogram/dev/api/open-api/user-info/wx.getUserInfo.html

在调用 wx.getUserInfo(Object object) 接口后，建议大家将获取到的用户密文信息发送到后端进行解密后使用。其中解密出来的信息中将包含用户的昵称、性别、头像、地区信息等，重要的是还包含 OpenId 及 UnionId ，这两个数据很重要，我相信很多人或多或少都会用到它进行一些业务处理。

```JAVA
/**
 * 小程序解密用户数据
 *
 * @param sessionKey    会话密钥
 * @param encryptedData 包括敏感数据在内的完整用户信息的加密数据
 * @param iv            密算法的初始向量
 * @return 解密结果
 */
public static JSONObject decryptionUserInfo(String encryptedData, String sessionKey, String iv) {
    // 被加密的数据
    byte[] dataByte = Base64.decodeBase64(encryptedData);
    // 加密秘钥
    byte[] keyByte = Base64.decodeBase64(sessionKey);
    // 偏移量
    byte[] ivByte = Base64.decodeBase64(iv);

    try {
        // 如果密钥不足16位，那么就补足. 这个if 中的内容很重要
        int base = 16;
        if (keyByte.length % base != 0) {
            int groups = keyByte.length / base + (keyByte.length % base != 0 ? 1 : 0);
            byte[] temp = new byte[groups * base];
            Arrays.fill(temp, (byte) 0);
            System.arraycopy(keyByte, 0, temp, 0, keyByte.length);
            keyByte = temp;
        }
        // 初始化
        Security.addProvider(new BouncyCastleProvider());
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS7Padding", "BC");
        SecretKeySpec spec = new SecretKeySpec(keyByte, "AES");
        AlgorithmParameters parameters = AlgorithmParameters.getInstance("AES");
        parameters.init(new IvParameterSpec(ivByte));
        // 初始化
        cipher.init(Cipher.DECRYPT_MODE, spec, parameters);
        byte[] resultByte = cipher.doFinal(dataByte);
        if (null != resultByte && resultByte.length > 0) {
            String result = new String(resultByte, Constants.UTF_8);
            return JSONObject.parseObject(result);
        }
    } catch (Exception e) {
        LogHelper.error("微信小程序解密用户信息--执行异常", e);
    }
    return null;
}
```

如上，便是对于用户密文信息的解密过程。其中 Base64 解码工具包使用的是 commons-codec 包中的 Base64.decodeBase64(String str) 方法，也可以使用其他的类似方法。

在很多业务场景中都会使用到 OpenId 或者 UnionID 但是小程序不同于公众号，将小程序添加到微信公众平台并不能百分百获取到用户的 UnionID 信息。可以根据下面的说明进行判别获取不到 UnionID 的原因。

**UnionID 机制说明：**

> 如果开发者拥有多个移动应用、网站应用、和公众帐号（包括小程序），可通过 UnionID 来区分用户的唯一性，因为只要是同一个微信开放平台帐号下的移动应用、网站应用和公众帐号（包括小程序），用户的 UnionID 是唯一的。换句话说，同一用户，对同一个微信开放平台下的不同应用，UnionID是相同的。

**UnionID获取途径：**

> 绑定了开发者帐号的小程序，可以通过以下途径获取 UnionID。

> 调用接口 wx.getUserInfo，从解密数据中获取 UnionID。注意本接口需要用户授权，请开发者妥善处理用户拒绝授权后的情况。

> 如果开发者帐号下存在同主体的公众号，并且该用户已经关注了该公众号。开发者可以直接通过 wx.login + code2Session 获取到该用户 UnionID，无须用户再次授权。

> 如果开发者帐号下存在同主体的公众号或移动应用，并且该用户已经授权登录过该公众号或移动应用。开发者也可以直接通过 wx.login + code2Session 获取到该用户 UnionID ，无须用户再次授权。

> 用户在小程序（暂不支持小游戏）中支付完成后，开发者可以直接通过getPaidUnionId接口获取该用户的 UnionID，无需用户授权。注意：本接口仅在用户支付完成后的5分钟内有效，请开发者妥善处理。

> 小程序端调用云函数时，如果开发者帐号下存在同主体的公众号，并且该用户已经关注了该公众号，可在云函数中通过 cloud.getWXContext 获取 UnionID。

> 小程序端调用云函数时，如果开发者帐号下存在同主体的公众号或移动应用，并且该用户已经授权登录过该公众号或移动应用，也可在云函数中通过 cloud.getWXContext 获取 UnionID。

综上所述，通常通过让用户授权，然后获取用户信息密文进行解密即可获取到用户的 UnionID 信息，本人亲测可以成功获取到用户的 UnionId 信息，因此建议通过此方案获取用户的 UnionID 信息哦。

好了，今天的分享就到这里啦，在做微信小程序用户登录及用户信息处理的小伙伴可以留意下本篇文章哦，兴许对您有所帮助呢！
