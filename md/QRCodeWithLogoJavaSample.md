```
注意事項：
    1. 此範例使用 Maven 建置相關內容請另行查閱
    2. 文章內容僅供參考
```

## 目錄

[TOC]

## 成果

<img src='data:image/png;base64,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' />


## 檔案配置

```
JavaSample
|____main
| |____java
| | |____sample
| |   |____qrcode
| |     |____QRCodeUtils.java
| |     |____SampleMain.java
| |____resources
|   |____qrcode-logo.png
|____output.png
|____pom.xml
```

## pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>tw.cookie.project</groupId>
    <artifactId>JavaSample</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <zxing.version>3.1.0</zxing.version>
    </properties>

    <dependencies>
        <!-- Official ZXing ("Zebra Crossing") -->
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>core</artifactId>
            <version>${zxing.version}</version>
        </dependency>
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>javase</artifactId>
            <version>${zxing.version}</version>
        </dependency>
    </dependencies>
</project>
```

## QRCodeUtils

```
package sample.qrcode;

import com.google.zxing.client.j2se.MatrixToImageWriter;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.qrcode.QRCodeWriter;

import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.*;

import static com.google.zxing.BarcodeFormat.QR_CODE;

/**
 * Created by Cookie on 16/1/30.
 */
public class QRCodeUtils {
    private static final String IMAGE_FORMAT = "png";

    private static final int BLACK = 0xFF000000;
    private static final int WHITE = 0xFFFFFFFF;

    private static final int WIDTH = 256;
    private static final int HEIGHT = 256;

    private static final String FRONT_IMAGE_NAME = "qrcode-logo.png";
    private static final BufferedImage FRONT_IMG;
    private static final int FRONT_IMG_START_X;
    private static final int FRONT_IMG_START_Y;

    private static final QRCodeWriter WRITER = new QRCodeWriter();

    static {
        InputStream inputStream = null;
        BufferedInputStream headBIS = null;
        BufferedImage image = null;
        int start_x = 0;
        int start_y = 0;
        try {
            inputStream = ClassLoader.getSystemResourceAsStream(FRONT_IMAGE_NAME);
            headBIS = new BufferedInputStream(inputStream);
            image = ImageIO.read(headBIS);

            start_x = (WIDTH - image.getWidth()) /2;
            start_y = (HEIGHT - image.getHeight()) /2;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (headBIS != null) {
                try {
                    headBIS.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        FRONT_IMG = image;
        FRONT_IMG_START_X = start_x;
        FRONT_IMG_START_Y = start_y;
    }

    public static ByteArrayOutputStream encode(String content) {
        ByteArrayOutputStream result = null;
        try {
            BitMatrix matrix = WRITER.encode(content, QR_CODE, WIDTH, HEIGHT);
            result = new ByteArrayOutputStream();
            MatrixToImageWriter.writeToStream(matrix, IMAGE_FORMAT, result);
        } catch (Exception e) {
            e.printStackTrace();
            close(result);
        }
        return result;
    }

    public static ByteArrayOutputStream encodeWithLogo(String content) {
        if (FRONT_IMG == null) {
            return encode(content);
        }
        ByteArrayOutputStream result = null;
        try {
            BitMatrix matrix = WRITER.encode(content, QR_CODE, WIDTH, HEIGHT);
            BufferedImage backImage = QRCodeUtils.toBufferedImage(matrix);

            Graphics2D g = backImage.createGraphics();
            g.drawImage(FRONT_IMG, FRONT_IMG_START_X, FRONT_IMG_START_Y, null);

            result = new ByteArrayOutputStream();
            if(!ImageIO.write(backImage, IMAGE_FORMAT, result)) {
                throw new IOException("Could not write an image of format png.");
            }
        } catch (Exception e) {
            e.printStackTrace();
            close(result);
        }
        return result;
    }

    private static BufferedImage toBufferedImage(BitMatrix matrix) {
        int w = matrix.getWidth();
        int h = matrix.getHeight();
        BufferedImage image = new BufferedImage(w, h, BufferedImage.TYPE_INT_RGB);
        for (int x = 0; x < w; x++) {
            for (int y = 0; y < h; y++) {
                image.setRGB(x, y, matrix.get(x, y) ? BLACK : WHITE);
            }
        }
        return image;
    }

    private static void close(Closeable obj) {
        if (obj == null) return;
        try {
            obj.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Sample Main

```
package sample.qrcode;

import java.io.ByteArrayOutputStream;
import java.io.FileOutputStream;

/**
 * Created by Cookie on 16/1/30.
 */
public class SampleMain {
    public static void main(String[] args) throws Exception {
        ByteArrayOutputStream stream = QRCodeUtils.encodeWithLogo(
            "http://tsai-cookie.blogspot.com/2016/01/qrcode-with-logo-java-sample.html");
        if (stream == null) throw new RuntimeException("Create QRCode Failed.");
        FileOutputStream out = new FileOutputStream("output.png");
        out.write(stream.toByteArray());
        stream.close();
        out.close();
    }
}
```


