```
注意事項：
	1. 內容為閱讀數篇文章後撰寫，如有雷同需修改請告知，謝謝。
	2. 文章內容僅供參考
```

## 目錄

[TOC]

## 前言

工作上的實際需求造成需要通過 Client 作為中繼進行資料交換。傳送至前端的資料由於安全性考量不能讓前端直接可以擷取。為了實現資料僅能由特定的對象才能解析，最終決定利用加密的方式來解決這個需求。

加密分成單向加密與雙相加密，而雙相加密又分成對稱式加密與非對稱式加密，本篇文章使用的是使用雙向加密的非對稱式加密。在此模式下會有一把公鑰及一把私鑰，任選其一可以對資料進行加密，而後使用另一把鑰匙進行解密。

由於網路傳輸時小編自己本身會利用 Json 進行傳輸，而 Json 無法直接支援 bytes 的傳輸，因此，文章內容有利用 Base64 進行轉碼的動作。

## Sample Code

```
import org.apache.commons.codec.binary.Base64;
import java.security.*;
import javax.crypto.Cipher;

import static java.lang.System.out;

/**
 * Created by Cookie on 16/2/14.
 */
public class EncryptRSA {

    public static byte[] encode(Key key, byte[] srcBytes) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, key);
        return cipher.doFinal(srcBytes);
    }

    public static byte[] decode(Key key, byte[] srcBytes) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, key);
        return cipher.doFinal(srcBytes);
    }

    public static String encodeBase64(Key key, byte[] srcBytes) throws Exception {
        return Base64.encodeBase64String(encode(key, srcBytes));
    }

    public static byte[] decodeBase64(Key key, String base64String) throws Exception {
        return decode(key, Base64.decodeBase64(base64String));
    }

    public static void main(String[] args) throws Exception {
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
        keyPairGen.initialize(512);
        KeyPair keyPair = keyPairGen.generateKeyPair();

        String msg = "這是測試加解密用的內容";
        out.println("明文:" + msg);

        PrivateKey privateKey = keyPair.getPrivate();
        out.println("私鑰:");
        out.println("\tFormat:" + privateKey.getFormat());
        out.println("\tkey:" + Base64.encodeBase64String(privateKey.getEncoded()));

        PublicKey publicKey = keyPair.getPublic();
        out.println("公鑰:");
        out.println("\tFormat:" + publicKey.getFormat());
        out.println("\tkey:" + Base64.encodeBase64String(publicKey.getEncoded()));
        {
            out.println("公鑰加密，私鑰解密:");
            String base64Str = encodeBase64(keyPair.getPublic(), msg.getBytes());
            out.println("\t加密後:" + base64Str);
            byte[] decBytes = decodeBase64(keyPair.getPrivate(), base64Str);
            out.println("\t解密後:" + new String(decBytes));
        }
        {
            out.println("私鑰加密，公鑰解密:");
            String base64Str = encodeBase64(keyPair.getPrivate(), msg.getBytes());
            out.println("\t加密後:" + base64Str);
            byte[] decBytes = decodeBase64(keyPair.getPublic(), base64Str);
            out.println("\t解密後:" + new String(decBytes));
        }
    }
}
```

## 執行結果

```
明文:這是測試加解密用的內容
私鑰:
	Format:PKCS#8
	key:MIIBVgIBADANBgkqhkiG9w0BAQEFAASCAUAwggE8AgEAAkEAgUdsEoDH4Kg8q64pN
	    WA4qBR3QG8RGJ0xaySnwEDbyFZg3Y/xOoVVouCisMFTzkbH4WE3UxwGhPlDSLktR/
	    bpDwIDAQABAkBGxno1GwnSRWiJuNxYm2gJJMMwpF2gsxZGCRhJmXh5kYEqljfcMag
	    rBBazFlmHAu64ityCKH9ti0C38S5FI4cZAiEAvltkHdxDWBgXFUmr8GZRG8uWcL1r
	    RVi/1lQXIwDEITMCIQCt3BliqMmKUeIT5ApCo6CNbeus3qWQBM/1vbcCOGbQtQIhA
	    Inc1p1psLxUdiNMv+HTuFpREBuGk/IdXJJ1RGxtWZ5RAiEArAmJjRjcJWUVQv16Ma
	    rcalaEyNCgH7zDU7Xg6++HvakCIQCDjr3MZq6Kf7CSKhBMvqkf9uqb1ekg1BBIohT
	    lSWCrqA==
	    
公鑰:
	Format:X.509
	key:MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAIFHbBKAx+CoPKuuKTVgOKgUd0BvERidM
	    Wskp8BA28hWYN2P8TqFVaLgorDBU85Gx+FhN1McBoT5Q0i5LUf26Q8CAwEAAQ==
	    
公鑰加密，私鑰解密:
	加密後:AqaQ3bHigGEmc/Me8D5jnrJQZnwJMegPfMyOcQhU9SMA+gkellyxO0X/66XP12b
	      kiG43tCXRa9lk7rOXba08XQ==
	解密後:這是測試加解密用的內容
	
私鑰加密，公鑰解密:
	加密後:TDW5uEYyQ1n5ESk4WlDjPxBCL+0q3u8Qp4RDy4YtIqxoXMcgivRrjd0xH6Yqk+7
	      8U2FPxkRHFDv82ufkMmMHig==
	解密後:這是測試加解密用的內容
```
