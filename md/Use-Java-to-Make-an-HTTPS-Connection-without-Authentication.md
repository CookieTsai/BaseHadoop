<!--How to Use Java to Make an HTTPS Connection without Authentication-->

```
注意事項：
	1. TrustModifier 使用上需確認 HTTPS Protocol 是否為 TLS
	2. 本文供參考及學習使用
```

## Catalog

[TOC]

## Abstract

過去小編曾撰寫過不同語言的 HTTPS 連線，必須 PASS 掉 HTTPS 中 X509 和 HostName 的狀況並不算少數，這時以 Python 來講只需要增加一個布林值即可，但在 Java 中卻需要實作兩個類別，為了能更簡單的有效的進行程式開發及測試，小編撰寫了一個 TrustModifier，此類別中僅有一個外部可以呼叫的方法，通過此方法可以快速 PASS 掉 X509 及 HostName 驗證

<h3>關於 HTTPS 的小知識</h3>

在撰寫爬蟲或串接第三方 API 時，不可避免的在某些情況下必須用到 HTTPS 協定，這時如果對方伺服器所提供的是合法的接口，那在實作上只需要直接用 HTTPS 方式去連接即可。但事實並非總是如此，在測試中或使用到內部網路時，會因為伺服器不被信任而造成程式中斷，就小編所知這時我們的選擇，有以下兩種：

1. 將對方伺服器中的憑證加入信任名單，並設法欺騙己方的伺服器，認為對方的 HostName 是正確的
2. 在連線過程中無條件 Pass 第一種的兩種檢查

P.S 無論哪一種在傳輸時都仍然有加密，只是目前所要連線的伺服器並未通過驗證可能不安全

## TrustModifier.java

```java
import javax.net.ssl.*;
import java.net.URLConnection;
import java.security.cert.*;

/**
 * Created by Cookie on 16/8/9.
 */
public class TrustModifier {
    private static final HostnameVerifier VERIFIER = new AllowHostnameVerifier();
    private static final TrustManager[] MANAGER = new TrustManager[]{new AllowManager()};
    private static final TrustModifier MODIFIER = new TrustModifier("TLS");

    private final SSLSocketFactory factory;

    private TrustModifier(String protocol) {
        try {
            SSLContext ctx = SSLContext.getInstance(protocol);
            ctx.init(null, MANAGER, null);
            factory = ctx.getSocketFactory();
        } catch (Exception e) {
            throw new RuntimeException("Create Trust Modifier Failed.", e);
        }
    }

    private HttpsURLConnection modify(HttpsURLConnection conn) {
        conn.setSSLSocketFactory(factory);
        conn.setHostnameVerifier(VERIFIER);
        return conn;
    }

    /** Call this method and it will modify connection with trust settings. */
    public static URLConnection trust(URLConnection conn) {
        return (conn instanceof HttpsURLConnection)?
                MODIFIER.modify((HttpsURLConnection) conn) : conn;
    }

    private static final class AllowHostnameVerifier implements HostnameVerifier {
        public boolean verify(String hostname, SSLSession session) {
            return true;
        }
    }

    private static final class AllowManager implements X509TrustManager {
        public void checkClientTrusted(X509Certificate[] arg0, String arg1) {}
        public void checkServerTrusted(X509Certificate[] arg0, String arg1) {}
        public X509Certificate[] getAcceptedIssuers() { return null; }
    }
}
```

## SampleMain.java

```java
import java.io.*;
import java.net.*;

/**
 * Created by Cookie on 16/8/9.
 */
public class SampleMain {

    public static void main(String[] args) throws IOException {
        URLConnection conn = new URL("https://74.125.203.94/").openConnection();

        /** Modify connection */
        TrustModifier.trust(conn);

        String inputLine;
        try(InputStreamReader stream = new InputStreamReader(conn.getInputStream());
            BufferedReader in = new BufferedReader(stream)) {
            while ((inputLine = in.readLine()) != null) {
                System.out.println(inputLine);
            }
        }
    }
}
```

## Output

在 Output 中可以看出成功拿到了一段內容並未出現例外，連線成功

```
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```