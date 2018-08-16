---
title: 关于android系统中微信小程序连接后台报错
date: 2018-08-10 15:42:03
tags:
---
在远程调试我的微信小程序的时候发现了如下报错:    
```javascript 
errMsg:"request:fail ssl hand shake error:java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.
```
查询上述报错，发现在stackoverflow上的答案说在web浏览器中访问有时报上述错误主要是因为对应的https 服务器的https证书缺少中间证书导致不被信任：  
```text
https://stackoverflow.com/questions/6825226/trust-anchor-not-found-for-android-ssl-connection/16302527#16302527    
I hit the same problem while connecting to an Apache server with an incorrectly installed dynadot/alphassl certificate. I'm connecting using HttpsUrlConnection (Java/Android), which was throwing -

javax.net.ssl.SSLHandshakeException: 
  java.security.cert.CertPathValidatorException: 
    Trust anchor for certification path not found.
The actual problem is a server misconfiguration - test it with http://www.digicert.com/help/ or similar, and it will even tell you the solution:

"The certificate is not signed by a trusted authority (checking against Mozilla's root store). If you bought the certificate from a trusted authority, you probably just need to install one or more Intermediate certificates. Contact your certificate provider for assistance doing this for your server platform."
```
然后继续搜索，发现还有人遇到了类似的问题，而且只在android手机上复现，ios不会出现，亲测发现确实如此。   
[中间证书是什么](https://www.myssl.cn/home/article-0406-42.html)    
这样猜测是Android的webview像某些Web浏览器一样，在网站没有提供中间证书，只提供了自己被颁发的证书的的时候，不会自动通过被颁发证书在网络上获取网站的中间证书直至根证书，而是直接不信任网站。  
解决办法就是打开[这个链接](https://www.myssl.cn/tools/downloadchain.html)来获取中间证书，并把中间证书拼接到自己的证书文本后面（用文本编辑器打开证书进行编辑，拼接两个证书的内容，或者可以使用上述链接的网站上也有拼接工具，可以避免自己拼接多了空格导致不可用)。  
我的https服务器提供了.key和.cer两种证书，我只是给.cer的证书拼接了中间证书，问题就解决了。对于https我并不是很懂，只是把自己的操作记录下来备查，也给大家做个参考。   