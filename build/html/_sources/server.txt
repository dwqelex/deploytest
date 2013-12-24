
概述
----

在用户支付完成之后，支付平台会将支付结果通知到应用提供的回调地址，应用接收到回调后，应该根据结果给用户提供相应的商品，并给支付平台返回一个事先约定好的结果。至此一笔交易在支付平台才算完成。

当应用没有返回要求的结果时，支付平台最多会回调3次，3次不成功之后，当前交易会被设置为失败交易，之后会有客服介入，手动处理失败交易。

为了保证应用方收到的数据是由支付平台发送而非第三方伪造，支付平台提供了二次验证的方式，应用服务端在收到回调通知之后，可以拿收到的数据到支付平台提供的接口进行验证，若验证通过，则说明该通知真实可信。

回调环节
--------

应用需要提供一个回调地址来接收回调参数

支付平台会使用post方式发送以下参数：

* trans_id：交易流水号，该交易在支付平台生成的唯一id
* amount：道具数量，该值由前端传递而来
* user_id：完成支付的玩家id，该值由前端传递而来
* timestamp：时间戳
* gross：交易金额
* currency：货币类型代码，比如USD(美元)、CNY(人民币)、BRL(巴西雷亚尔)
* channel：渠道名称，比如alipay、paypal、mycard
* item：商品描述，该值由前端传递而来
* custom_data：应用方的自定义字段，由前端传递而来
* product_id：应用方定义的商品id，由前端传递而来
* pay_type：交易类型，移动端该值为mobile
* role_id：角色id

支付平台需要应用服务端返回以下规则的字符串：

* 3,null：表示游戏方处理失败
* 3,94a0acb127ef8ee8c925e3944941ce5e：表示游戏方不认识这个玩家id
* 3,user_id：表示处理成功，比如回调时，user_id为123456，那么游戏方的返回值应该为3,123456

如果3次回调后应用服务端未按规则返回，则该交易失败，等待客服处理。


安全验证
--------

*安全验证机制可以保证应用服务端收到的回调确实是由支付平台发起而非第三方伪造。*

验证交易信息
------------

**IP验证**

需要对请求作IP验证，只有IP地址：174.37.255.60，173.192.195.130发送过来的请求才能通过

**验证参数**

当接收到支付系统的回调之后，应请求地址 VERIFY URL 验证交易信息以保证交易信息的正确性。如果交易信息正确则智明星通支付系统将返回 'OK'。::

	[VERIFY URL]?trans_id=[TRANS_ID]&amount=[AMOUNT]&user_id=[USER_ID]&timestamp=[TIMESTAMP]&gross=[GROSS]&currency=[CURRENCY]&channel=[CHANNEL]

* VERIFY URL: https://pay.337.com/payelex/api/callback/verify.php	
* REQUEST TYPE: GET/POST

**验证返回**

* OK：交易信息正确
* 否则交易信息不可信，不能发游戏币

**样例代码**

注意：php版本的代码需要 `下载证书文件`_  ，请将这个文件放到和回调php文件同样的目录。
如果想要了解更多有关https的资料，请参阅文档：`HTTPS学习笔记`_
 
.. _下载证书文件: http://elexpublish.googlecode.com/files/verisign_ca.crt
.. _HTTPS学习笔记: http://code.google.com/p/elexpublish/wiki/https_notes

php ::

	<?php
	$trans_id = $_REQUEST ["trans_id"];
	$user_id = $_REQUEST ["user_id"];
	$amount = $_REQUEST['amount'];
	$gross = $_REQUEST['gross'];
	$currency = $_REQUEST['currency'];
	$channel = $_REQUEST['channel'];
	$timestamp = $_REQUEST['timestamp'];

	ob_clean();
	//To check if the transaction exists in db, if it does. means the transactions has been successfully processed. Just return OK status
	$exist = is_trans_exist($trans_id);
	if($exist) {
			echo '3,'.$user_id;
			return;
	}

	//to verify the transaction towards payelex server.
	$res = check_payelex_transaction($trans_id, $user_id, $amount, $gross, $currency, $channel);
	if(!$res) {
			echo "3,null";
			return;
	}

	//retrieve the user from db.
	$user = find_user_from_db();
	if ($user == null) {
			echo '3,94a0acb127ef8ee8c925e3944941ce5e';
			return;
	}

	//recharge the user with the deserved game coins.
	if(add_coins($_REQUEST)) {
			echo '3,'.$user_id;
			return;
	}
	echo "3,null";
	function check_payelex_transaction($trans_id, $user_id, $amount, $gross, $currency, $channel) {
			$ch = curl_init();
		curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
		curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 1);
		//verisign_ca.crt is the public certificate from VeriSign(It is the biggest Certificate Authority which issue ELEX client certificate)
		//verisign_ca.crt must be located at the same directory as this PHP code are.
		curl_setopt($ch, CURLOPT_CAINFO, 'verisign_ca.crt'); 
		curl_setopt($ch, CURLOPT_HTTPHEADER, array("Content-Type: application/x-www-form-urlencoded"));
		curl_setopt($ch, CURLOPT_URL, 'https://pay.337.com/payelex/api/callback/verify.php');
		curl_setopt($ch, CURLOPT_POST, true);  
			curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
			$params = array(
					'trans_id'=>$trans_id,
					'user_id'=>$user_id,
					'amount'=>$amount,
					'gross'=>$gross,
					'currency'=>$currency,
					'channel'=>$channel,
					'timestamp'=>$timestamp
			);  
			curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($params));  
		$result = curl_exec($ch);
		curl_close($ch);
		$result = trim($result);
		if ($result === 'OK') return true;
		return false;
	}
	
java ::

	import java.io.ByteArrayInputStream;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.InputStreamReader;
	import java.io.StringWriter;
	import java.net.HttpURLConnection;
	import java.net.URL;
	import java.net.URLEncoder;
	import java.security.InvalidKeyException;
	import java.security.NoSuchAlgorithmException;
	import java.security.NoSuchProviderException;
	import java.security.PublicKey;
	import java.security.SignatureException;
	import java.security.cert.Certificate;
	import java.security.cert.CertificateException;
	import java.security.cert.CertificateFactory;
	import java.security.cert.X509Certificate;

	import javax.net.ssl.HttpsURLConnection;
	import javax.net.ssl.SSLContext;
	import javax.net.ssl.TrustManager;
	import javax.net.ssl.X509TrustManager;

	import javax.servlet.ServletException;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;


	public class PayelexTransactionServlet extends HttpServlet {

			private static final long serialVersionUID = -2108375440169533437L;
			private static final String VERIFY_URL = "https://pay.337.com/payelex/api/callback/verify.php";
			private static final String VERISIGN_CA = 
					"-----BEGIN CERTIFICATE-----\n"+
					"MIIE0zCCA7ugAwIBAgIQGNrRniZ96LtKIVjNzGs7SjANBgkqhkiG9w0BAQUFADCByjELMAkGA1UE\n"+
					"BhMCVVMxFzAVBgNVBAoTDlZlcmlTaWduLCBJbmMuMR8wHQYDVQQLExZWZXJpU2lnbiBUcnVzdCBO\n"+
					"ZXR3b3JrMTowOAYDVQQLEzEoYykgMjAwNiBWZXJpU2lnbiwgSW5jLiAtIEZvciBhdXRob3JpemVk\n"+
					"IHVzZSBvbmx5MUUwQwYDVQQDEzxWZXJpU2lnbiBDbGFzcyAzIFB1YmxpYyBQcmltYXJ5IENlcnRp\n"+
					"ZmljYXRpb24gQXV0aG9yaXR5IC0gRzUwHhcNMDYxMTA4MDAwMDAwWhcNMzYwNzE2MjM1OTU5WjCB\n"+
					"yjELMAkGA1UEBhMCVVMxFzAVBgNVBAoTDlZlcmlTaWduLCBJbmMuMR8wHQYDVQQLExZWZXJpU2ln\n"+
					"biBUcnVzdCBOZXR3b3JrMTowOAYDVQQLEzEoYykgMjAwNiBWZXJpU2lnbiwgSW5jLiAtIEZvciBh\n"+
					"dXRob3JpemVkIHVzZSBvbmx5MUUwQwYDVQQDEzxWZXJpU2lnbiBDbGFzcyAzIFB1YmxpYyBQcmlt\n"+
					"YXJ5IENlcnRpZmljYXRpb24gQXV0aG9yaXR5IC0gRzUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAw\n"+
					"ggEKAoIBAQCvJAgIKXo1nmAMqudLO07cfLw8RRy7K+D+KQL5VwijZIUVJ/XxrcgxiV0i6CqqpkKz\n"+
					"j/i5Vbext0uz/o9+B1fs70PbZmIVYc9gDaTY3vjgw2IIPVQT60nKWVSFJuUrjxuf6/WhkcIzSdhD\n"+
					"Y2pSS9KP6HBRTdGJaXvHcPaz3BJ023tdS1bTlr8Vd6Gw9KIl8q8ckmcY5fQGBO+QueQA5N06tRn/\n"+
					"Arr0PO7gi+s3i+z016zy9vA9r911kTMZHRxAy3QkGSGT2RT+rCpSx4/VBEnkjWNHiDxpg8v+R70r\n"+
					"fk/Fla4OndTRQ8Bnc+MUCH7lP59zuDMKz10/NIeWiu5T6CUVAgMBAAGjgbIwga8wDwYDVR0TAQH/\n"+
					"BAUwAwEB/zAOBgNVHQ8BAf8EBAMCAQYwbQYIKwYBBQUHAQwEYTBfoV2gWzBZMFcwVRYJaW1hZ2Uv\n"+
					"Z2lmMCEwHzAHBgUrDgMCGgQUj+XTGoasjY5rw8+AatRIGCx7GS4wJRYjaHR0cDovL2xvZ28udmVy\n"+
					"aXNpZ24uY29tL3ZzbG9nby5naWYwHQYDVR0OBBYEFH/TZafC3ey78DAJ80M5+gKvMzEzMA0GCSqG\n"+
					"SIb3DQEBBQUAA4IBAQCTJEowX2LP2BqYLz3q3JktvXf2pXkiOOzEp6B4Eq1iDkVwZMXnl2YtmAl+\n"+
					"X6/WzChl8gGqCBpH3vn5fJJaCGkgDdk+bW48DW7Y5gaRQBi5+MHt39tBquCWIMnNZBU4gcmU7qKE\n"+
					"KQsTb47bDN0lAtukixlE0kF6BWlKWE9gyn6CagsCqiUXObXbf+eEZSqVir2G3l6BFoMtEMze/aiC\n"+
					"Km0oHw0LxOXnGiYZ4fQRbxC1lfznQgUy286dUV4otp6F01vvpX1FQHKOtw5rDgb7MzVIcbidJ4vE\n"+
					"ZV8NhnacRHr2lVz2XTIIM6RUthg/aFzyQkqFOFSDX9HoLPKsEdao7WNq\n"+
					"-----END CERTIFICATE-----";
			
			@Override
		protected void doGet(HttpServletRequest request, HttpServletResponse response)
					throws ServletException, IOException {
					doPost(request, response);
			}
			
			protected void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
					String transId=request.getParameter("trans_id");
					String userId=request.getParameter("user_id");
					String amount=request.getParameter("amount");
					String gross=request.getParameter("gross");
					String currency=request.getParameter("currency");
					String channel=request.getParameter("channel");
					String timestamp=request.getParameter("timestamp");
					boolean flag=check(transId,userId,amount,gross,currency,channel,timestamp);
					response.setContentType("application/json; charset=UTF-8");
					response.setStatus(HttpServletResponse.SC_OK);
					
					if(flag==true){
							//TODO:检查该uid在游戏中是否真实存在，如果不存在的话返回3,94a0acb127ef8ee8c925e3944941ce5e
							boolean isUserExists=checkUserIdExists(userId);
							if(isUserExists==false){
									response.getWriter().write("3,94a0acb127ef8ee8c925e3944941ce5e");
							}else{
									response.getWriter().write("3,"+userId);
							}
					}else{
							response.getWriter().write("3,null");
					}
			}
			public boolean checkUserIdExists(String userId){
					//TODO:判断该玩家是否真实存在,需要开发者自行扩展该方法
					return false;
			}
			public static final boolean check(String transId, String userId, String amount, String gross, String currency, String channel,String timestamp) {
					try {
							StringBuilder buffer = new StringBuilder();
							buffer.append("trans_id=").append(URLEncoder.encode(transId, "UTF-8")).append("&")
									.append("user_id=").append(URLEncoder.encode(userId, "UTF-8")).append("&")
									.append("amount=").append(URLEncoder.encode(amount, "UTF-8")).append("&")
									.append("gross=").append(URLEncoder.encode(gross, "UTF-8")).append("&")
									.append("currency=").append(URLEncoder.encode(currency, "UTF-8")).append("&")
									.append("channel=").append(URLEncoder.encode(channel, "UTF-8")).append("&")
									.append("timestamp").append(URLEncoder.encode(timestamp, "UTF-8"));
							
							TrustManager[] trustAllCerts = new TrustManager[]{
						new X509TrustManager() {
									public X509Certificate[] getAcceptedIssuers() {
										return null;
									}
									public void checkClientTrusted(X509Certificate[] certs, String authType) {
									}
									public void checkServerTrusted(X509Certificate[] certs, String authType) {
													InputStream is = new ByteArrayInputStream(VERISIGN_CA.getBytes());
													try {
															CertificateFactory cf = CertificateFactory.getInstance("X.509");
															Certificate publicCert = cf.generateCertificate(is);
															PublicKey publicKey = publicCert.getPublicKey();
															boolean validSignature = false;
															for (int i = 0; i < certs.length; i++) {
																	try {
																			certs[i].verify(publicKey);
																			validSignature = true;
																			break;
																	} catch (SignatureException e) {}
															}
															if (!validSignature) {
																	throw new SignatureException();
															}
													} catch (InvalidKeyException e) {
															throw new RuntimeException(e);
													} catch (CertificateException e) {
															throw new RuntimeException(e);
													} catch (NoSuchAlgorithmException e) {
															throw new RuntimeException(e);
													} catch (NoSuchProviderException e) {
															throw new RuntimeException(e);
													} catch (SignatureException e) {
															throw new RuntimeException(e);
													}
									}
								}
							};
							SSLContext sc = SSLContext.getInstance("SSL");
						sc.init(null, trustAllCerts, new java.security.SecureRandom());
						HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
						
							URL serverUrl = new URL(VERIFY_URL);
							HttpURLConnection conn = (HttpURLConnection) serverUrl.openConnection();
							conn.setConnectTimeout(10000);
							conn.setReadTimeout(10000);
							conn.setRequestMethod("POST");
							conn.setDoOutput(true);
							conn.connect();
					
							conn.getOutputStream().write(buffer.toString().getBytes("UTF-8"));
							if (conn.getResponseCode() == HttpURLConnection.HTTP_OK || conn.getResponseCode() == HttpURLConnection.HTTP_CREATED) {
									String res = toString(conn.getInputStream(), "UTF-8");
									if (res != null && res != "" && res.trim().equals("OK")) return true;
					}
							return false;
					} catch (Exception e) {
							return false;
					}
			}
			private static String toString(InputStream is, String encoding) throws IOException {
					InputStreamReader in = new InputStreamReader(is, encoding);
					StringWriter sw = new StringWriter();
					char[] b = new char[1024 * 4];
			int n = 0;
			while (-1 != (n = in.read(b))) {
					sw.write(b, 0, n);
			}
					return sw.toString();
			}
			public static void main(String[] args) {
					if (check("elex337c1f4d6a5c520c02cd0ccd43712a3b23e", "elex337_24319771", "4500.0", "30.14", "TRY", "elex337")) {
							System.out.println("check OK");
					} else {
							System.out.println("check Failed");
					}
			}
	}	

