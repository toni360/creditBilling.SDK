
#信用支付开发指南

@(发布.蜂鸟)[开发指南|信用支付|SDK]


##第一步：SDK一键支付接口
###1.接口说明

**creditPay接口：**
只要调用SDK中的接口,就可以实现一键支付，让用户毫无中断游戏感觉。
    
    Pay pay = PayFactory.getInstance(context);
    string appId = "bingxueqiyuan";
    string appKey= "222dwdddiuw344d8dfr088";
    string productName = "商品名称";
    int sum = 300;  //精确到分，表示300分，即3元
    string alias = "360market";
    string sellerUserId = "uc-zhangsan";
    
    pay.creditPay(appId, appKey, productName, sum, alias, sellerUserId,new ResponseCallback() {
	    Parameters: code 支付状态码
	    @Override
    	public void responseStateCode(int code) {
		..............//开发者的处理代码
	}
    });
    

参数|必须|说明
---|-------|----  
appId|是|信用支付平台分配给游戏应用的应用标识
appKey|是|信用支付平台分配给游戏应用的接入秘钥，不能对外公开  
productName|是|商品名称，可以是汉字，支持28个字节的字母和数字，或支持14个汉字
sum |是|交易金额，分为单位，最大到千元，即长度为6位(1-100000分)
alias|是|开发者自定义串 ,可以是渠道标识，长度不能超过为100个字符(只能是字符或数字)
sellerUserId|是|商户用户ID，28个字符，如果商户传入值大于28，则截取28个字符(只能是字符或数字)
responseCallback|是|回调接口,通过此接口通知开发者支付状态

code: 

	// 支付成功: 0;

	// 支付失败: 404;

	// productName(商品名称)不能等于null:401;

	// sum交易金额因在(1-100000分): 402;

	// alias不合法 :403;

	// sellerUserId不能等于null :403;

	// sellerUserId不能有汉字 :405;

**isBalanceDue接口**
isBalanceDue(appId, appKey, BalanceDueCallback balanceDueCallback)	

该接口用于游戏在启动后调用，用于判断该用户是否有欠款到期需要偿还。
通过回调notifyBalanceDueMomey(int money)通知开发者欠款金额
如果有到期欠款的，money等于欠款金额，如果无到期欠款，money等于0。

>1)如果该用户未登录，money等于0，表示这个无法识别的用户没有欠款。	

    //初始化准备环境
    Pay pay = PayFactory.getInstance(context);

    pay.isBalanceDue(appId, appKey, new BalanceDueCallback() {
	    /*
	    *@param momey : 欠款金额(无到期欠款或没有登录momey==0)
	    */
        @Override
        public void notifyBalanceDueMomey(int money) {
        ..............//开发者的处理代码
    }
    };
	

参数|必须|说明
----|----|----
appId|是|信用支付平台分配给游戏应用的应用标识
appKey|是|信用支付平台分配给游戏应用的接入秘钥，不能对外公开  BalanceDueCallback|是|如果isBalanceDue(null)没有任何意义


###2.导入信用支付SDK到开发工程

导入信用支付SDK开发包（以下简称SDK）到你开发的应用程序项目

1）paysdk是一个Library,你需要把你的paysdk引入到工作空间中

2）把该library与你自己的项目关联(eclispe工具关联步骤,在你自己的工程右键->propreties->android->add...->选择paysdk->ok->ok)做完这两部操作你就可以进行paysdk开发了


###3.配置AndroidManifest.xml

####权限配置：

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.SEND_SMS" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
####Activity配置：
    <activity
            android:name="com.wi360.pay.sdk.WebViewRechargeActivity">

如下面所示:

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.wi360.pay.sdk"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.SEND_SMS" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    
    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="17" />


    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >

        <activity
            android:name="com.wi360.pay.sdk.WebViewRechargeActivity"
            android:launchMode="singleTask"
            android:screenOrientation="portrait" >
        </activity>

    </application>

    </manifest>



##第二步：服务端计费成功通知接口


当信用支付成功后，信用支付平台将通知业务平台平台订单计费成功。便于应用平台进行下一步逻辑的处理。

1）接口调用请求说明

http请求方式：POST

    http://NOTIFY_URL/notify

POST数据格式：JSON

POST数据例子：

    {"orderId":"1234567890123456789","productName":"钻石道具","sum":300,"payTime":"2014-04-05 12:22:30","status":"支付成功","buyerId":"13912345678","appId":"12345678","alias":"商户自定义字段","sellerUserId":"对用的商户的用户ID"}

参数|是否必须|说明
---|-------|----
alias|是|开发者自定义串，可以是渠道标识，长度不能超过为100个字符
orderId|是|成功计费的订单流水号
productName|是|商品名称
sum|是|交易金额
payTime|是|成功计费时间
status|是|支付状态
sellerUserId|是|商户定义的用户ID
NOTIFY_URL|是|业务平台定义的通知URL地址

2）返回说明

正常时的返回JSON数据包示例：

    {"errcode":0,"errmsg":"接收到订单计费通知信息"}

参数|是否必须|说明
---|-------|----
errcode|是|错误代码
errmsg|是|错误代码信息描述

##SDK下载地址

http://cl.ly/0s2U3q0q3l1x

##DEMO下载地址

http://cl.ly/2D24003P1f09/download/paysdk_demo.rar



