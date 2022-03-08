# 接入支付宝 APP 支付-证书版



&emsp;&emsp;最近项目中需要集成支付宝支付，以完善用户账户充值时的支付通道。由于项目比较依赖微信平台，所以在项目初期就已经集成了微信支付通道，但是由于各种原因迟迟未集成支付宝支付。最近项目趋于稳定拓展中，所以开始对支付通道进行完善，首选的就是要集成支付宝支付。

&emsp;&emsp;前期准备工作已由其他同事完成，所以我不需要从头开始去注册公司账户，创建申请应用等等的一些操作了。我直接从配置证书开始到集成SDK，再到开发APP支付相关业务处理。

### 设置接口加密方式

> 前往支付宝开发平台，点击“密钥管理” --> “开放平台密钥”即可见到如下界面：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1592145467074.png)

> 这里，咋们可以在目标应用那一栏，点击“**设置/查看**”进行密钥及证书上传等相关操作。具体的可参考官方文档 [**生成密钥**](https://opendocs.alipay.com/open/291/105971/)。

> 根据本机系统环境，下载对应的密钥生成工具：
> 
> [WINDOWS](https://ideservice.alipay.com/ide/getPluginUrl.htm?clientType=assistant&platform=win&channelType=WEB)（**Windows版本工具请不要安装在含有空格的目录路径下，否则会导致公私钥乱码的问题**）
> [MAC_OSX](https://ideservice.alipay.com/ide/getPluginUrl.htm?clientType=assistant&platform=mac&channelType=WEB)

> 官方给出的配置证书的指导文档地址：https://opendocs.alipay.com/open/291/twngcd

### 集成服务器端SDK

> 这里的服务器端语言是 JAVA ，运用的 Gradle 对 Jar 包进行统一管理。我们可以先找到支付宝SDK，如下图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1592145970023.png)

> 地址：https://opendocs.alipay.com/open/54/106370
> 这里我们可以找到服务端SDK及其相关的示例 demo 可以参考。

> 注意：官方声明，当前仅 JAVA 版 SDK 4.4.5.ALL 及以上版本支持 App 支付场景的证书签名功能。想了解更多证书签名内容，请参考开放平台证书升级指南。
> 我这里使用的就是证书版本的 APP 支付功能，所以默认使用最新版 SDK 即可。

> 点击 JAVA 版资源下载“Maven项目依赖”进入如下界面：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1592146176902.png)

> 我们选择最新版本，点击对应的版本号即可进入下一个界面：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1592146249637.png)

> 在这个页面我们就可以看到我们需要的 Gradle 引入 Jar 对应的语句了，点击旁边的“复制按钮”(黄绿色圆圈中)即可复制到项目中进行使用了，如果是 Maven 项目可以用 Apache Maven 下边的框中的内容，也可以点击旁边的“复制按钮”进行复制使用。
> 也可以前往 Maven 官方仓库进行搜索，搜索词“alipay”即可搜出：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1592146543336.png)

> 搜出来后使用方式和上面差不多，只是展示的界面效果不同。

### 开发APP支付及回调处理

> 在上一步中我们集成好了 SDK，现在我们便可以直接使用 SDK 进行预支付处理了。

> 接下来将展示部分 Java 代码进行说明，代码中引入了日志工具类(LogHelper)及异常统一处理(MfException)，代码中还含有部分数据持久化操作用的 Mapper 及对应的实体类 Model，如果复制如下代码需要将其替换成你们自己的处理类，否则会报错哦。

```java
import com.alibaba.fastjson.JSON;
import com.alipay.api.AlipayApiException;
import com.alipay.api.AlipayClient;
import com.alipay.api.CertAlipayRequest;
import com.alipay.api.DefaultAlipayClient;
import com.alipay.api.domain.AlipayTradeAppPayModel;
import com.alipay.api.internal.util.AlipaySignature;
import com.alipay.api.request.AlipayTradeAppPayRequest;
import com.alipay.api.response.AlipayTradeAppPayResponse;
import mf.base.LogHelper;
import mf.base.MfException;
import mf.dao.AliPayOrderMapper;
import mf.model.AliPayOrder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.util.Map;
import java.util.Objects;

/**
 * @author: hy
 * @date: 2020/6/8 10:21 上午
 * @Description: 支付宝支付服务
 * @version: v_1.0.0
 */
@Service
public class AliPayService {
    @Value("${ALI_PAY_CHARSET}")
    private String ALI_PAY_CHARSET;
    @Value("${ALI_PAY_SIGN_TYPE}")
    private String ALI_PAY_SIGN_TYPE;
    @Value("${ALI_PAY_FORMAT}")
    private String ALI_PAY_FORMAT;
    @Value("${ALI_PAY_APP_PRIVATE_KEY}")
    private String ALI_PAY_APP_PRIVATE_KEY;
    @Value("${ALI_PAY_PAY_TYPE_APP}")
    private String ALI_PAY_PAY_TYPE_APP;
    @Value("${ALI_PAY_APP_ID}")
    private String ALI_PAY_APP_ID;
    @Value("${ALI_PAY_TIME_OUT_EXPRESS}")
    private String ALI_PAY_TIME_OUT_EXPRESS;
    @Value("${ALI_PAY_NOTIFY_URL}")
    private String ALI_PAY_NOTIFY_URL;
    @Value("${ALI_PAY_SERVER_URL}")
    private String ALI_PAY_SERVER_URL;
    @Value("${ALI_PAY_PRIVATE_KEY}")
    private String ALI_PAY_PRIVATE_KEY;
    @Value("${ALI_PAY_PUBLIC_KEY}")
    private String ALI_PAY_PUBLIC_KEY;
    @Value("${ALI_PAY_CERT_PATH}")
    private String ALI_PAY_CERT_PATH;
    @Value("${ALI_PAY_PUBLIC_CERT_PATH}")
    private String ALI_PAY_PUBLIC_CERT_PATH;
    @Value("${ALI_PAY_ROOT_CERT_PATH}")
    private String ALI_PAY_ROOT_CERT_PATH;
    @Autowired
    private RechargeService rechargeService;
    @Autowired
    private AliPayOrderMapper aliPayOrderMapper;

    /**
     * 保存支付宝订单
     *
     * @param tradeNo        商户订单号
     * @param uid            用户ID
     * @param body           对一笔交易的具体描述信息
     * @param subject        商品的标题/交易标题/订单标题/订单关键字等
     * @param totalAmount    订单总金额，单位为元，精确到小数点后两位，取值范围[0.01,100000000]
     * @param terminalIp     终端IP
     * @param timeoutExpress 该笔订单允许的最晚付款时间，逾期将关闭交易
     * @param productCode    销售产品码，商家和支付宝签约的产品码
     */
    private void saveAliPayOrder(String tradeNo, Long uid, String body, String subject, String totalAmount, String terminalIp, String timeoutExpress, String productCode) {
        AliPayOrder aliPayOrder = new AliPayOrder(tradeNo, uid, ALI_PAY_APP_ID, body, subject, totalAmount, terminalIp, timeoutExpress, productCode);
        this.aliPayOrderMapper.insertSelective(aliPayOrder);
    }

    /**
     * 获取Resource中文件的位置（证书文件放置在 resources 目录下，此处统一处理路径）
     *
     * @param resourcePath resource文件位置
     * @return 问文件位置
     */
    private String getPath(String resourcePath) {
        return Objects.requireNonNull(Thread.currentThread().getContextClassLoader().getResource(resourcePath)).getPath();
    }

    /**
     * 获取支付宝支付请求对象
     *
     * @param body       对一笔交易的具体描述信息。
     * @param title      商品的标题/交易标题/订单标题/订单关键字等
     * @param amount     订单总金额，单位为元，精确到小数点后两位，取值范围[0.01,100000000]
     * @param outTradeNo 商户网站唯一订单号
     * @return request
     */
    private AlipayTradeAppPayRequest getAliPayRequest(String body, String title, String amount, String outTradeNo) {
        //实例化具体API对应的request类,类名称和接口名称对应,当前调用接口名称：alipay.trade.app.pay
        AlipayTradeAppPayRequest request = new AlipayTradeAppPayRequest();
        //SDK已经封装掉了公共参数，这里只需要传入业务参数。以下方法为sdk的model入参方式(model和biz_content同时存在的情况下取biz_content)。
        AlipayTradeAppPayModel model = new AlipayTradeAppPayModel();
        model.setBody(body);
        model.setSubject(title);
        model.setOutTradeNo(outTradeNo);
        model.setTimeoutExpress(ALI_PAY_TIME_OUT_EXPRESS);
        model.setTotalAmount(amount);
        model.setProductCode(ALI_PAY_PAY_TYPE_APP);
        request.setBizModel(model);
        request.setNotifyUrl(ALI_PAY_NOTIFY_URL);
        return request;
    }

    /**
     * 创建APP支付订单(密钥模式--保留-暂不使用)
     *
     * @param uid        用户ID
     * @param terminalIp 终端IP
     * @param body       对一笔交易的具体描述信息。
     * @param title      商品的标题/交易标题/订单标题/订单关键字等
     * @param amount     订单总金额，单位为元，精确到小数点后两位，取值范围[0.01,100000000]
     * @param outTradeNo 商户网站唯一订单号
     * @return 执行结果
     */
    public String createAppPayByKey(Long uid, String terminalIp, String body, String title, String amount, String outTradeNo) {
        //实例化客户端
        AlipayClient alipayClient = new DefaultAlipayClient(ALI_PAY_SERVER_URL, ALI_PAY_APP_ID, ALI_PAY_PRIVATE_KEY, ALI_PAY_FORMAT, ALI_PAY_CHARSET, ALI_PAY_PUBLIC_KEY, ALI_PAY_SIGN_TYPE);
        //实例化具体API对应的request类,类名称和接口名称对应,当前调用接口名称：alipay.trade.app.pay
        AlipayTradeAppPayRequest request = this.getAliPayRequest(body, title, amount, outTradeNo);
        try {
            //创建支付宝支付订单记录
            this.saveAliPayOrder(outTradeNo, uid, body, title, amount, terminalIp, ALI_PAY_TIME_OUT_EXPRESS, ALI_PAY_PAY_TYPE_APP);
            //这里和普通的接口调用不同，使用的是sdkExecute
            AlipayTradeAppPayResponse response = alipayClient.sdkExecute(request);
            System.out.println(response.getBody());//就是orderString 可以直接给客户端请求，无需再做处理。
            return response.getBody();
        } catch (AlipayApiException e) {
            LogHelper.error("AliPay创建APP支付订单出现异常", e);
            return "";
        }
    }

    /**
     * 创建APP支付订单(证书模式)
     *
     * @param uid        用户ID
     * @param terminalIp 终端IP
     * @param body       对一笔交易的具体描述信息。
     * @param title      商品的标题/交易标题/订单标题/订单关键字等
     * @param amount     订单总金额，单位为元，精确到小数点后两位，取值范围[0.01,100000000]
     * @param outTradeNo 商户网站唯一订单号
     * @return 执行结果
     * @throws AlipayApiException 执行异常
     */
    public String createAppPayByCrt(Long uid, String terminalIp, String body, String title, String amount, String outTradeNo) throws AlipayApiException {
        //构造client
        CertAlipayRequest certAlipayRequest = new CertAlipayRequest();
        //设置网关地址
        certAlipayRequest.setServerUrl(ALI_PAY_SERVER_URL);
        //设置应用Id
        certAlipayRequest.setAppId(ALI_PAY_APP_ID);
        //设置应用PKCS8格式的应用私钥
        certAlipayRequest.setPrivateKey(ALI_PAY_APP_PRIVATE_KEY);
        //设置请求格式，固定值json
        certAlipayRequest.setFormat(ALI_PAY_FORMAT);
        //设置字符集
        certAlipayRequest.setCharset(ALI_PAY_CHARSET);
        //设置签名类型
        certAlipayRequest.setSignType(ALI_PAY_SIGN_TYPE);
        //设置应用公钥证书路径
        certAlipayRequest.setCertPath(this.getPath(ALI_PAY_CERT_PATH));
        //设置支付宝公钥证书路径
        certAlipayRequest.setAlipayPublicCertPath(this.getPath(ALI_PAY_PUBLIC_CERT_PATH));
        //设置支付宝根证书路径
        certAlipayRequest.setRootCertPath(this.getPath(ALI_PAY_ROOT_CERT_PATH));
        //构造client
        AlipayClient alipayClient = new DefaultAlipayClient(certAlipayRequest);

        //实例化具体API对应的request类,类名称和接口名称对应,当前调用接口名称：alipay.trade.app.pay
        AlipayTradeAppPayRequest request = this.getAliPayRequest(body, title, amount, outTradeNo);
        try {
            //创建支付宝支付订单记录
            this.saveAliPayOrder(outTradeNo, uid, body, title, amount, terminalIp, ALI_PAY_TIME_OUT_EXPRESS, ALI_PAY_PAY_TYPE_APP);
            //这里和普通的接口调用不同，使用的是sdkExecute
            AlipayTradeAppPayResponse response = alipayClient.sdkExecute(request);
            System.out.println(JSON.toJSONString(response));//就是orderString 可以直接给客户端请求，无需再做处理。
            return response.getBody();
        } catch (AlipayApiException e) {
            LogHelper.error("AliPay创建APP支付订单出现异常", e);
            return "";
        }
    }

    /**
     * 支付宝异步回调处理
     *
     * @param params 支付宝回调传入参数
     */
    public Boolean doNotify(Map<String, String> params) throws AlipayApiException {
        //切记alipaypublickey是支付宝的公钥，请去open.alipay.com对应应用下查看。
        //boolean AlipaySignature.rsaCertCheckV1(Map<String, String> params, String publicKeyCertPath, String charset,String signType)
        boolean flag = AlipaySignature.rsaCertCheckV1(params, this.getPath(ALI_PAY_PUBLIC_CERT_PATH), ALI_PAY_CHARSET, ALI_PAY_SIGN_TYPE);
        if (flag) {
            //支付宝回调数据验签通过--业务处理
            try {
        //支付宝回调后的业务处理--此处支付宝支付用于充值，所以后续将进行充值支付成功后的业务处理
                this.rechargeService.aliPayNotify(params);
                return true;
            } catch (MfException e) {
                return false;
            }
        } else {
            //支付宝回调数据验签不通过--返回失败
            return false;
        }
    }
}
```

> 上面代码用到的配置文件信息如下：

```txt
#AliPay 支付宝支付-配置信息
ALI_PAY_CHARSET = utf-8
ALI_PAY_SIGN_TYPE = RSA2
ALI_PAY_FORMAT = json
ALI_PAY_TIME_OUT_EXPRESS = 30m
ALI_PAY_APP_PRIVATE_KEY = MIIEvgIBADANBgkqh...e3/dN5IgQ6y
ALI_PAY_PAY_TYPE_APP = QUICK_MSECURITY_PAY
ALI_PAY_APP_ID = 202xxxxxxxxxx202
ALI_PAY_NOTIFY_URL = https://www.xxx.com/aliPay/async/notify.do
ALI_PAY_SERVER_URL = https://openapi.alipay.com/gateway.do
ALI_PAY_PRIVATE_KEY =
ALI_PAY_PUBLIC_KEY =
ALI_PAY_CERT_PATH = ali_pay_cert/appCertPublicKey_20xxxxxx02.crt
ALI_PAY_PUBLIC_CERT_PATH = ali_pay_cert/alipayCertPublicKey_RSA2.crt
ALI_PAY_ROOT_CERT_PATH = ali_pay_cert/alipayRootCert.crt
```

> 回调处理的接口控制器示例，如下：

```java
import mf.base.LogHelper;
import mf.dubbo.service.web.PayService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

/**
 * @author: hy
 * @date: 2020/6/1 3:27 下午
 * @Description: 支付宝业务处理控制器
 * @version: v_1.0.0
 */
@Controller
@RequestMapping(value = "/aliPay/")
public class AliPayController {
    @Autowired
    private AliPayService aliPayService;

    /**
     * 微信支付回调接口
     *
     * @param request  请求对象
     * @param response 响应对象
     */
    @RequestMapping("async/notify.do")
    @ResponseBody
    public void notify(HttpServletRequest request, HttpServletResponse response) {
        LogHelper.print("支付宝回调接收参数1：", request.getParameterMap());
        //获取支付宝POST过来反馈信息
        Map<String, String> params = new HashMap<String, String>();
        Map requestParams = request.getParameterMap();
        for (Iterator iter = requestParams.keySet().iterator(); iter.hasNext(); ) {
            String name = (String) iter.next();
            String[] values = (String[]) requestParams.get(name);
            String valueStr = "";
            for (int i = 0; i < values.length; i++) {
                valueStr = (i == values.length - 1) ? valueStr + values[i] : valueStr + values[i] + ",";
            }
            //乱码解决，这段代码在出现乱码时使用。
            //valueStr = new String(valueStr.getBytes("ISO-8859-1"), "utf-8");
            params.put(name, valueStr);
        }

        try {
            LogHelper.print("支付宝回调参数Params", params);
            Boolean isSuccess = this.aliPayService.doNotify(params);
            if (isSuccess) {
                response.getWriter().print("SUCCESS");
            } else {
                response.getWriter().print("FAIL");
            }
        } catch (Exception e) {
            e.printStackTrace();
            LogHelper.error("支付宝回调处理异常", e);
        }
    }
}
```

> 针对上面的代码示例进行说明：

> - 首先需要说明一下的是回调处理的验签部分，使用**支付宝公钥**，也就是上文配置文件的 ALI_PAY_PUBLIC_CERT_PATH 这项配置的证书，切记不要用错了。很多人会在此处踩坑！
> - 其次，回调处理的时候，由于咋们约定的字符编码为 utf-8 ，所以不需要再处理乱码，即使是中文也不会乱码，如果将解决乱码的代码放开，将导致其他问题。有兴趣的可以尝试放开看看效果！
> - 其余需要注意字符编码为 utf-8 ，签名类型是 RSA2 ，签约的产品码(productCode)是 QUICK_MSECURITY_PAY 这些基本上都是固定的。

以上便是集成支付宝支付完成预支付订单部分的处理，处理成功后将会返回预下单结果，如下所示：

```
alipay_root_cert_sn=687b59193f3f462dd5336e5abf83c5d8_02941eef3187dddf3d3b83462e1dfcf6&alipay_sdk=alipay-sdk-java-dynamicVersionNo&app_cert_sn=94bc4bff3807a953f058f9535e3af0eb&app_id=2021001136635202&biz_content=%7B%22body%22%3A%22%E7%82%B9%E9%87%91%E5%85%AC%E7%A4%BE%22%2C%22out_trade_no%22%3A%2220061617391613567%22%2C%22product_code%22%3A%22QUICK_MSECURITY_PAY%22%2C%22subject%22%3A%22APP%E5%85%85%E5%80%BC%22%2C%22timeout_express%22%3A%2230m%22%2C%22total_amount%22%3A%220.01%22%7D&charset=utf-8&format=json&method=alipay.trade.app.pay&notify_url=https%3A%2F%2Fwww.hyblogs.com%2FaliPay%2Fasync%2Fnotify.do&sign=QbnKgbrwazSwFjRLzYME5yUK1vBPrunFmq4%2FYL7KORueM9OFEZbaJeM0YFt7Kox11znvGrd%2BQgZGRVGYHTFbIAq2Jt%2B4FOxictKqXlw%2FZEKg7tgc8VfUpoLSmO96S0Ic%2FF4y6QyGJJCGCD7hFj0n6gv85EXD77VW%2FE29JpP19z61xPHQW0y91tkxbYW7Y0pYrvj8Fm54Hyy1ypBQ9f8M1XVsDBNKg8TVzv9F3M%2B98wLDLt4UV3w1a7N55Yjzl5sFzTY19LS%2F5mDH7ysQh8YZwvYtjc1WZmAByaxGKSaSYM2ijf%2FgTanUsuN5aktUoYjMi6Hr6UU3xn9%2F64gidzRwRg%3D%3D&sign_type=RSA2&timestamp=2020-06-16+17%3A39%3A16&version=1.0
```

返回的结果可以直接返回给 APP 端发起实际支付处理。这里推荐一个后端自测 APP(安卓APK) --> [download:客户端调试.zip](https://alipaybbs.oss-cn-hangzhou.aliyuncs.com/1807/thread/60_191_eb31b639a0caf31.zip) &emsp;下载安装后便可以将刚才预下单结果粘贴进行发起支付实测了，实测通过了，那么 APP 端集成 APP-SDK 后获取接口数据就能正常实际支付了，这样就可以减少前后端联调的时间啦。此工具具体的使用说明请参考官方：[客户端调试工具使用教程](https://openclub.alipay.com/club/history/read/7695)

### 总结

&emsp;&emsp;到此，后端(JAVA)集成支付宝完成 APP 支付部分的预下单处理就完成了。

&emsp;&emsp;上面第一步中的密钥配置没有完全展示所有步骤，可以参考官方文档一步一步的去完成即可，比较简单。

&emsp;&emsp;后面第二步主要是选择合适的 SDK 进行集成，我选择的是 Alipay SDK ，此 SDK 可以获得所有开放能力的完整支持。如果你不需要这么复杂，也可以选择集成 Alipay Easy SDK 来处理常见的一些场景。亦或者将两者都集成到项目中。当然，Alipay Easy SDK 和 Alipay SDK 并不冲突，有需要的开发者也完全可以同时使用这两套 SDK 产品。

&emsp;&emsp;最后第三步主要是对整个 APP 支付预下单流程的代码梳理展示，切记有些地方需要特别注意，避免踩坑浪费时间。上面有对应的说明，不妨看看，也许你遇不到，但是看看也没有啥坏处。

&emsp;&emsp;好了，今天的分享记录就到这里了，如有疑问可评论区一起讨论～
