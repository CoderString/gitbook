### RSA进行公私钥加解密

参考前任解决的方案，工具类源码。

```
import javax.crypto.Cipher;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

@SuppressWarnings("all")
public class RSAUtil {

        //取值 SHA256WithRSA  或者 SHA1WithRSA
        //RSA 对应 SHA1WithRSA
        //RSA2 对应 SHA256WithRSA  至少2048 位以上
        public static final String SHA_WITH_RSA_ALGORITHM = "SHA256WithRSA";

        public static final String MD5_WITH_RSA = "MD5WithRSA";

        public static final int KEY_SIZE_2048 = 2048;

        public static final int KEY_SIZE_1024 = 1024;

        public static final String  ALGORITHM = "RSA";

        public static final String  CHARSET = "UTF-8";


        /**
         * SHAWithRSA签名
         */
        public static String sign(String in,String pivateKey,String algorithm) throws Exception{
            PKCS8EncodedKeySpec priPKCS8 = new PKCS8EncodedKeySpec(Base64.getDecoder().decode(pivateKey));
            KeyFactory keyf = KeyFactory.getInstance(ALGORITHM);
            PrivateKey priKey = keyf.generatePrivate(priPKCS8);
            Signature signa = Signature.getInstance(algorithm!=null?algorithm:SHA_WITH_RSA_ALGORITHM);
            signa.initSign(priKey);
            signa.update(in.getBytes());
            byte[] signdata = signa.sign();
            return Base64.getEncoder().encodeToString(signdata);
        }

        /**
         * SHAWithRSA签名
         * 默认 SHAWithRSA签名
         */
        public static String sign(String in,String pivateKey) throws Exception{
            return sign(in,pivateKey,null);
        }

        /**
         * SHAWithRSA验签
         * 默认SHAWithRSA验签
         */
        public static boolean isValid(String in,String signData,String publicKey,String algorithm) throws Exception{
            KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);
            byte[] encodedKey = Base64.getDecoder().decode(publicKey);
            PublicKey pubKey = keyFactory.generatePublic(new X509EncodedKeySpec(encodedKey));
            Signature signa = Signature.getInstance(algorithm!=null?algorithm:SHA_WITH_RSA_ALGORITHM);
            signa.initVerify(pubKey);
            signa.update(in.getBytes());
            byte[] sign_byte = Base64.getDecoder().decode(signData);
            boolean flag = signa.verify(sign_byte);
            return flag;
        }

        /**
         *  SHAWithRSA验签
         */
        public static boolean isValid(String in,String signData,String publicKey) throws Exception{
            return isValid(in,signData,publicKey,null);
        }

        /**
         * RSA公钥加密
         */
        public static String encrypt( String str, String publicKey) throws Exception{
            //base64编码的公钥
            byte[] decoded = Base64.getDecoder().decode(publicKey);
            RSAPublicKey pubKey = (RSAPublicKey) KeyFactory.getInstance(ALGORITHM).generatePublic(new X509EncodedKeySpec(decoded));
            //RSA加密
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, pubKey);
            String outStr = Base64.getEncoder().encodeToString(cipher.doFinal(str.getBytes(CHARSET)));
            return outStr;
        }

        /**
         * RSA私钥解密
         */
        public static String decrypt(String str, String privateKey) throws Exception{
            //64位解码加密后的字符串
            byte[] inputByte = java.util.Base64.getDecoder().decode(str.getBytes(CHARSET));
            //base64编码的私钥
            byte[] decoded = Base64.getDecoder().decode(privateKey);
            RSAPrivateKey priKey = (RSAPrivateKey) KeyFactory.getInstance(ALGORITHM).generatePrivate(new PKCS8EncodedKeySpec(decoded));
            //RSA解密
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, priKey);
            String outStr = new String(cipher.doFinal(inputByte));
            return outStr;
        }

        /**
         * 随机生成密钥对
         * 也可以用阿里的生成工具
         */
        public static void genKeyPair(Integer keySize) throws NoSuchAlgorithmException {
            //推荐 2048位以上，保证安全性
            if(keySize==null){
                keySize = KEY_SIZE_2048;
            }
            // KeyPairGenerator类用于生成公钥和私钥对，基于RSA算法生成对象
            KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(ALGORITHM);
            // 初始化密钥对生成器，密钥大小为96-1024位
            keyPairGen.initialize(keySize, new SecureRandom());
            // 生成一个密钥对，保存在keyPair中
            KeyPair keyPair = keyPairGen.generateKeyPair();
            RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();   // 得到私钥
            RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();  // 得到公钥
            String publicKeyString = new String(Base64.getEncoder().encode(publicKey.getEncoded()));
            // 得到私钥字符串
            String privateKeyString = new String(Base64.getEncoder().encode(privateKey.getEncoded()));
            // 将公钥和私钥保存到Map
            System.out.println("公钥:"+publicKeyString);
            System.out.println("私钥:"+privateKeyString);
        }

        //简单测试
        public static void main(String[] args) throws Exception {
            String content = "我的数据";
            //支付宝RSA生成的公私钥 也同样适用，可以自行尝试
            genKeyPair(null);
            String publicKey = "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAkGv/70Te42PS5fyuLCNr/UiMtDqhasMZpmA92aDTWjh9RT5r4Kl3sKRO63RitWuEgFAZHljJUB9afEzWnP+EGpkeiq3SMpIfpUWy89nqZxUUyOg6aIvfEZBjKFFc3umM9uasFTLC8l4NCfDnlkpHxwQjc5emUGJ5ZnqqGOmrdP5se7dyO7QEtdI7/jfbzPDmMeWyegWE5iN8CHJh2V0foBur7gS/syTnbLFcRr3wzsbXh9RyLP9OMmHx9MO1fhTOnTZnN+xWXNggTgymoUM27+bZd/nrKaZNeUF50DIyTFEe6yqwkbtVf2jeR+CNjcWO+uP7CvTpDKk+9h3DZyZ5RwIDAQAB";
            String privateKey = "MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQCQa//vRN7jY9Ll/K4sI2v9SIy0OqFqwxmmYD3ZoNNaOH1FPmvgqXewpE7rdGK1a4SAUBkeWMlQH1p8TNac/4QamR6KrdIykh+lRbLz2epnFRTI6Dpoi98RkGMoUVze6Yz25qwVMsLyXg0J8OeWSkfHBCNzl6ZQYnlmeqoY6at0/mx7t3I7tAS10jv+N9vM8OYx5bJ6BYTmI3wIcmHZXR+gG6vuBL+zJOdssVxGvfDOxteH1HIs/04yYfH0w7V+FM6dNmc37FZc2CBODKahQzbv5tl3+esppk15QXnQMjJMUR7rKrCRu1V/aN5H4I2NxY764/sK9OkMqT72HcNnJnlHAgMBAAECggEAYq4o1miMk3rl49fferFJXGtyGMPm/3gH0rL4D/ff8kme7u1T8NJawgvDEQcZWzT3+GTChQXNqD2EKmKmUegVb8coI0HZ2kwV62vQduZzT7QL26syHbVU2j96QVY2yulyNFIxStrAcbLp3d0JoJtoqAef4Z/BODPRF8DA8PzY9rrKM38EIki/lywgnhmmavfB4Z0jOi+yjL7uwj497YAC6XK5A4ZKFXYp1D5b0af7N7VyZNte/r/3nrH+/T2FpLGB3I8y/VB7cAS5GXB0GrUMOvVfnm576tukh4nOi+F0J3YxHW2AJU92RYE4qLT9nq5FsHFtu83+0Lo6C51wl4WcAQKBgQDNiM0J2gc10gNd0lv47CIUxgprKDJgxOs7xm7H+MVGzy2xpfsm3kjB/M4fHVDFLhSbqgo3TYBd9SgMxJRcmuE1HBs2Vd0NXMM+Q/XdIYA57YV3rclx7xUf95XZ2G2PFFFh45VW68KamZKufs4kbXwb3nrFgT48qkFWiKxTrXi7gQKBgQCz4d9da6+WVuDR5tURnZDATwbEefxpmSaIGbF16JAR1sY4Wnj5hB0phD0Pv0b/8+35ZpKGpK/ca7qm1VR9fEfMvP1f311ICXqovoydHktgEhTC+ictDMoHjSjJ8VHMKz+ahmQKB/AFHqReg+lfkcc3zOf/A99cvqWWkyGmq4i4xwKBgGZHjmk5o17oDJ7SwMwFjgwyZRrgHPnE5J6RZ62BoYJUNRPzWiEEesZ2LIiVSQ1mmgDAxGay3Y9kITMBXCcdN7b7LpuCbQdqQwqoPSB2vF2XUlS1Gcrlw+hth5epuRN7c+g3nahsmCHhDHpjReggx6MCuquwXi1IOE18o+zcJXmBAoGAVmTBVqkFp/sJ90YaR1+ZygMqiOrdpAn+S5erd6m+qBKzGRW6zHv7VZlBinKfswaA4Su2bBxkqkTDXKVQ8wPhqB+MwaMRtit3UdxSxJNsODP27L4gWq6tyXqugG76jkinP5wUKA0v5gWVhB9u0ou9Vrt/ISfG+1BFT1BS9S2leLkCgYAbFtXUQTNKd9IuI1pZguuwVI8US/xAOM4uTQoQx/tUO3OwgLt9iziKqg1n8GtRESAGwfbtJli85fhYSB36CdPs3ad22Y1SM6xnpLt5TnA8Vc3IuME4Y1ogH6Z3EBVvTPBb5BX2XoBvbpx8nMxtPIrh5ZkS8Cb9aoOoEhTrvEXrNw==";
            /*加解密测试*/
            String encryptContent = RSAUtil.encrypt(content, publicKey);
            System.out.println("加密后数据:\n" + encryptContent);
            String decryptContent = RSAUtil.decrypt(encryptContent,privateKey);
            System.out.println("解密后数据:\n"+decryptContent);
            /*加签验签测试*/
            String signOriginalContent = "验签功能测试";
            String geneSignContent = RSAUtil.sign(signOriginalContent, privateKey);
            System.out.println("生成的签名:\n"+geneSignContent);
            boolean valid = RSAUtil.isValid(signOriginalContent,geneSignContent, publicKey);
            System.out.println("结果验签:\n"+valid);
        }
}

```

### 可能遇到的问题

```
error=Internal Server Error, message=Illegal base64 character 20,
trace=java.lang.IllegalArgumentException: Illegal base64 character 20
at java.util.Base64$Decoder.decode0(Base64.java:714)
at java.util.Base64$Decoder.decode(Base64.java:526)
at java.util.Base64$Decoder.decode(Base64.java:549)
```

造成的原因：base64 编码中使用了加号（+），而 + 在 URL 传递时会被当成空格，因此造成了base64字符串被更改，在服务器端解码后就会出错。

解决方案：解密后的数据需要简单处理下，String s = str.replaceAll(" ", "+") 或者