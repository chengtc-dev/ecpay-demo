> [綠界金流 Java SDK 使用者文件](https://developers.ecpay.com.tw/)

到 [ECPayAIO_Java](https://github.com/ECPay/ECPayAIO_Java) 下載 SDK

將 SDK 加入專案，但是因為缺少 Apache Log4j Core 和 Java Servlet API 這兩個 dependencies 所以 AllInOne.java 出現錯誤，因此從 [Maven Repository](https://mvnrepository.com/) 加入 dependencies 到 pom.xml 中

```java
<!-- <https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api> -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.0.1</version>
    <scope>provided</scope>
</dependency>

<!-- <https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core> -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version>
</dependency>

```

將 SDK 中的 payment_conf.xml 加入至專案路徑 src\main\resources 底下，基本上無須修改內容

建立 controller 及 service 層，訂單內容請自行修改，可參考[範例](https://github.com/ECPay/ECPayAIO_Java/blob/master/example/ExampleAllInOne.java)

```java
package com.example.ECPayDemo.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import com.example.ECPayDemo.service.OrderService;

@RestController
public class OrderController {

	@Autowired
	OrderService orderService;

	@PostMapping("/ecpayCheckout")
	public String ecpayCheckout() {
		String aioCheckOutALLForm = orderService.ecpayCheckout();

		return aioCheckOutALLForm;
	}
}
```

```java
package com.example.ECPayDemo.service;

import org.springframework.stereotype.Service;

import ecpay.payment.integration.AllInOne;
import ecpay.payment.integration.domain.AioCheckOutALL;

@Service
public class OrderService {

	public String ecpayCheckout() {

		AllInOne all = new AllInOne("");

		AioCheckOutALL obj = new AioCheckOutALL();
		obj.setMerchantTradeNo("testCompany0004");
		obj.setMerchantTradeDate("2017/01/01 08:05:23");
		obj.setTotalAmount("50");
		obj.setTradeDesc("test Description");
		obj.setItemName("TestItem");
		// 交易結果回傳網址，只接受 https 開頭的網站，可以使用 ngrok
		obj.setReturnURL("<http://211.23.128.214:5000>");
		obj.setNeedExtraPaidInfo("N");
		// 商店轉跳網址 (Optional)
		obj.setClientBackURL("<http://192.168.1.37:8080/>");
		String form = all.aioCheckOut(obj, null);

		return form;
	}
}
```

使用 API Tester 呼叫 http://localhost:8080/ecpayCheckout ，**HTTP METHOD 要記得用 POST !**，呼叫成功(http 狀態碼為 200)後，應該會出現一段內容有表單的 html

將上述的 html 貼到空白的記事本並將副檔名由 .txt 修改為 .html，打開它後你會看到付款畫面，造成交易失敗的原因是訂單編號重覆了，修改廠商交易編號 MerchantTradeNo 即可，細節請參考[FAQ知識庫](https://www.ecpay.com.tw/CascadeFAQ/CascadeFAQ_Qa?nID=1454&_gl=1*crf39v*_gcl_aw*R0NMLjE2Nzk2NzIzNjEuQ2owS0NRandsUFdnQmhESEFSSXNBSDJ4ZE5mTUpZY1AzMEZhelpUMWtZYjlsNWhHY0dOdWR6NTdjX2RIZkg4LWxOZHJ0NFFsMm9peFNRb2FBcTRWRUFMd193Y0I.)

為了測試方便，所以使用 UUID 隨機產生 20 碼的亂數

```java
package com.example.ECPayDemo.service;

import java.util.UUID;

import org.springframework.stereotype.Service;

import ecpay.payment.integration.AllInOne;
import ecpay.payment.integration.domain.AioCheckOutALL;

@Service
public class OrderService {

	public String ecpayCheckout() {

		String uuId = UUID.randomUUID().toString().replaceAll("-", "").substring(0, 20);

		AllInOne all = new AllInOne("");

		AioCheckOutALL obj = new AioCheckOutALL();
		obj.setMerchantTradeNo(uuId);
		obj.setMerchantTradeDate("2017/01/01 08:05:23");
		obj.setTotalAmount("50");
		obj.setTradeDesc("test Description");
		obj.setItemName("TestItem");
		obj.setReturnURL("<http://211.23.128.214:5000>");
		obj.setNeedExtraPaidInfo("N");
		String form = all.aioCheckOut(obj, null);

		return form;
	}
}
```

若是成功會出現付款畫面，在下方輸入

- 手機號碼
- 一般信用卡測試卡號 : 4311-9522-2222-2222
- 安全碼 : 222
- 信用卡測試有效月/年：輸入的 MM/YYYY 值請大於現在當下時間的月年，例如在 2016/04/20 當天作測試，請設定 05/2016(含)之後的有效月年，否則回應刷卡失敗。
細節請參考[準備事項 / 測試介接資訊](https://developers.ecpay.com.tw/?p=2856)


最終會看到付款成功
