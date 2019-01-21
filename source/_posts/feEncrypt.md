---
title: 前端加密那点事
date: 2019-01-21 20:08:02
tags: ‘加密’
categories:
	     - 密码学
---

<!--more-->

## 前奏

最近公司一个项目在传输数据的时候，测试部门安全扫描后，发现密码类型的数据是明文传输的，果断不符合要求，让加密，就有了接下来的故事。
## 使用场景
前后端使用HTTP协议进行交互的时候，由于HTTP报文为明文，所以通常情况下对于比较敏感的信息可以通过加密在前端加密，然后在后端解密实现"混淆"的效果，避免在传输过程中敏感信息的泄露（如，密码，证件信息等）。不过前端加密只能保证传输过程中信息是‘混淆’过的，对于高手来说，打个debugger，照样可以获取到数据，并不安全，所谓的前端加密只是稍微增加了攻击者的成本，并不能保证真正的安全。  
综上，服务端绝对不能相信前端传递过来的密文直接保存入库，只能通过服务端自己加密进行加密保存。那么前端加密是不是就没有意义了呢？答案是否定的，至少可以保证传输过程中不是明文传输，如果前后端交互需要安全的通道建议使用HTTPS协议进行通信。

## 分类  
简单来说，加密分两种方式
* 对称加密  
  **对称加密采用了对称密码编码技术，它的特点是文件加密和解密使用相同的密钥加密也就是密钥也可以用作解密密钥，这种方法在密码学中叫做对称加密算法，对称加密算法使用起来简单快捷，密钥较短，且破译困难，除了数据加密标准（DES），另一个对称密钥加密系统是国际数据加密算法（IDEA），它比DES的加密性好，而且对计算机功能要求也没有那么高**  
  常见的对称加密算法有DES、3DES、Blowfish、IDEA、RC4、RC5、RC6和AES 
* 非对称加密  
  **非对称加密算法需要两个密钥：公钥（publickey）和私钥（privatekey）。   公钥与私钥是一对，如果用公钥对数据进行加密，只有用对应的私钥才能解密；如果用私钥对数据进行加密，那么只有用对应的公钥才能解密。因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。
  非对称加密算法实现机密信息交换的基本过程是：甲方生成一对密钥并将其中的一把作为公钥向其它方公开；得到该公钥的乙方使用该密钥对机密信息进行加密后再发送给甲方；甲方再用自己保存的另一把专用密钥对加密后的信息进行解密。甲方只能用其专用密钥解密由其公钥加密后的任何信息。**  
  常见的非对称加密算法有：RSA、ECC（移动设备用）、Diffie-Hellman、El Gamal、DSA（数字签名用）

## 对称加密DES实现方式
(1)使用Crypto-JS通过DES算法在前端加密

```
npm install crypto-js
```
(2)加解密  
使用DES算法，工作方式为ECB，填充方式为PKcs7  

```
var CryptoJS = require("crypto-js");
const secretKey = 'com.sevenlin.foo.key';
var afterEncrypt = CryptoJS.DES.encrypt('encryptCode', CryptoJS.enc.Utf8.parse(secretKey), {
        mode: CryptoJS.mode.ECB,
        padding: CryptoJS.pad.Pkcs7
}).toString()
console.log(afterEncrypt);//8/nZ2vZXxOzPhU7ZHBwz7w==
var afterDecrypt = CryptoJS.DES.decrypt(afterEncrypt, CryptoJS.enc.Utf8.parse(secretKey), {
        mode: CryptoJS.mode.ECB,
        padding: CryptoJS.pad.Pkcs7
}).toString(CryptoJS.enc.Utf8);
console.log(afterDecrypt);//encryptCode
```
(3)使用BC通过DES算法在后端解密  
a.安装

```
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk16</artifactId>
    <version>1.46</version>
</dependency>
```
b.加解密工具

```
public class DesCipherUtil {

    private DesCipherUtil() {
        throw new AssertionError("No DesCipherUtil instances for you!");
    }

    static {
        // add BC provider
        Security.addProvider(new BouncyCastleProvider());
    }

    /**
     * 加密
     * 
     * @param encryptText 需要加密的信息
     * @param key 加密密钥
     * @return 加密后Base64编码的字符串
     */
    public static String encrypt(String encryptText, String key) {

        if (encryptText == null || key == null) {
            throw new IllegalArgumentException("encryptText or key must not be null");
        }

        try {
            DESKeySpec desKeySpec = new DESKeySpec(key.getBytes());
            SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("DES");
            SecretKey secretKey = secretKeyFactory.generateSecret(desKeySpec);

            Cipher cipher = Cipher.getInstance("DES/ECB/PKCS7Padding", "BC");
            cipher.init(Cipher.ENCRYPT_MODE, secretKey);
            byte[] bytes = cipher.doFinal(encryptText.getBytes(Charset.forName("UTF-8")));
            return Base64.getEncoder().encodeToString(bytes);

        } catch (NoSuchAlgorithmException | InvalidKeyException | InvalidKeySpecException | NoSuchPaddingException
            | BadPaddingException | NoSuchProviderException | IllegalBlockSizeException e) {
            throw new RuntimeException("encrypt failed", e);
        }

    }

    /**
     * 解密
     * 
     * @param decryptText 需要解密的信息
     * @param key 解密密钥，经过Base64编码
     * @return 解密后的字符串
     */
    public static  String decrypt(String decryptText, String key) {

        if (decryptText == null || key == null) {
            throw new IllegalArgumentException("decryptText or key must not be null");
        }

        try {
            DESKeySpec desKeySpec = new DESKeySpec(key.getBytes());
            SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("DES");
            SecretKey secretKey = secretKeyFactory.generateSecret(desKeySpec);

            Cipher cipher = Cipher.getInstance("DES/ECB/PKCS7Padding", "BC");
            cipher.init(Cipher.DECRYPT_MODE, secretKey);
            byte[] bytes = cipher.doFinal(Base64.getDecoder().decode(decryptText));
            return new String(bytes, Charset.forName("UTF-8"));

        } catch (NoSuchAlgorithmException | InvalidKeyException | InvalidKeySpecException | NoSuchPaddingException
            | BadPaddingException | NoSuchProviderException | IllegalBlockSizeException e) {
            throw new RuntimeException("decrypt failed", e);
        }
    }
}
```
c.解密前端的加密信息

```
String fromWeb = "8/nZ2vZXxOzPhU7ZHBwz7w==";
String key = "com.sevenlin.foo.key";
String afterDecrypt = DesCipherUtil.decrypt(fromWeb, key);
System.out.println(afterDecrypt);//encryptCode
```

## 非对称加密
(1)[下载加密文件依赖](https://github.com/travist/jsencrypt/archive/master.zip)  
(2) 引入依赖

```
<script src="bin/jsencrypt.min.js"></script>
```
或者

```
import '../../assets/js/jsencrypt.min.js';
```
(3)使用方式
```
this.privateKey=`-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQDlOJu6TyygqxfWT7eLtGDwajtNFOb9I5XRb6khyfD1Yt3YiCgQ
WMNW649887VGJiGr/L5i2osbl8C9+WJTeucF+S76xFxdU6jE0NQ+Z+zEdhUTooNR
aY5nZiu5PgDB0ED/ZKBUSLKL7eibMxZtMlUDHjm4gwQco1KRMDSmXSMkDwIDAQAB
AoGAfY9LpnuWK5Bs50UVep5c93SJdUi82u7yMx4iHFMc/Z2hfenfYEzu+57fI4fv
xTQ//5DbzRR/XKb8ulNv6+CHyPF31xk7YOBfkGI8qjLoq06V+FyBfDSwL8KbLyeH
m7KUZnLNQbk8yGLzB3iYKkRHlmUanQGaNMIJziWOkN+N9dECQQD0ONYRNZeuM8zd
8XJTSdcIX4a3gy3GGCJxOzv16XHxD03GW6UNLmfPwenKu+cdrQeaqEixrCejXdAF
z/7+BSMpAkEA8EaSOeP5Xr3ZrbiKzi6TGMwHMvC7HdJxaBJbVRfApFrE0/mPwmP5
rN7QwjrMY+0+AbXcm8mRQyQ1+IGEembsdwJBAN6az8Rv7QnD/YBvi52POIlRSSIM
V7SwWvSK4WSMnGb1ZBbhgdg57DXaspcwHsFV7hByQ5BvMtIduHcT14ECfcECQATe
aTgjFnqE/lQ22Rk0eGaYO80cc643BXVGafNfd9fcvwBMnk0iGX0XRsOozVt5Azil
psLBYuApa66NcVHJpCECQQDTjI2AQhFc1yRnCU/YgDnSpJVm1nASoRUnU8Jfm3Oz
uku7JUXcVpt08DFSceCEX9unCuMcT72rAQlLpdZir876
-----END RSA PRIVATE KEY-----`;
              this.publicKye=`-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDlOJu6TyygqxfWT7eLtGDwajtN
FOb9I5XRb6khyfD1Yt3YiCgQWMNW649887VGJiGr/L5i2osbl8C9+WJTeucF+S76
xFxdU6jE0NQ+Z+zEdhUTooNRaY5nZiu5PgDB0ED/ZKBUSLKL7eibMxZtMlUDHjm4
gwQco1KRMDSmXSMkDwIDAQAB
-----END PUBLIC KEY-----`;
  var encrypt = new JSEncrypt();
  encrypt.setPublicKey(this.publicKye);
  var encrypted = encrypt.encrypt('encryptCode');//w1a1FXmlbFj9yOxLCoqIzNo2ytXypyupZABsi/e4kMA9mERngmaDwlOuHsUDQKC0nK1v7Ehr3vYKcALFQvjscWEkGIW/UWCk73jArwqEYF1wd45eHSCPwUeB85Ellr+IYTqhZXcfmHZUCuprF2gayPUecq7F51aWxpfqMP0uvtY=
  // Decrypt with the private key...
  var decrypt = new JSEncrypt();
  decrypt.setPrivateKey(this.privateKey);
  var uncrypted = decrypt.decrypt(encrypted);//encryptCode
```
生成publickey和privateKey的[在线地址](http://travistidwell.com/jsencrypt/demo/index.html)  

虽然前端可以加密，终归不是安全方式，如果为了更加的安全还是使用https传输，后端加密保存吧！

参考文章：  
[通过DES实现JavaScript加密和Java解密](https://www.jianshu.com/p/24691c8d722c)  
[加密库](http://travistidwell.com/jsencrypt/)