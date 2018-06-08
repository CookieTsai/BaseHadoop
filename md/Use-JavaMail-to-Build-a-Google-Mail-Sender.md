# Use JavaMail to Build a Google Mail Sender

```
注意事項：
	1. 帳密需更改為自己的帳號
	2. 文章僅供參考
```

## 目錄

## 前言

## 程式

### GMailSender.java

```java
package com.netcat.net.common.utils;

import javax.activation.DataHandler;
import javax.activation.DataSource;
import javax.activation.FileDataSource;
import javax.mail.*;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeMultipart;
import java.io.File;
import java.io.FileNotFoundException;
import java.util.Properties;

public class GMailSender {
    private static final String USER_NAME = "";
    private static final String PASSWORD = "";

    private static Session SESSION;

    public GMailSender(SendType sendType) {
        Properties props = new Properties();

        switch (sendType) {
            case SSL:
                props.put("mail.smtp.host", "smtp.gmail.com");
                props.put("mail.smtp.socketFactory.port", "465");
                props.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
                props.put("mail.smtp.auth", "true");
                props.put("mail.smtp.port", "465");
            case TSL:
                props.put("mail.smtp.host", "smtp.gmail.com");
                props.put("mail.smtp.auth", "true");
                props.put("mail.smtp.starttls.enable", "true");
                props.put("mail.smtp.port", "587");
        }

        // Get the Session object.
        SESSION = Session.getInstance(props, new Authenticator() {
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(USER_NAME, PASSWORD);
            }
        });
    }

    public void send(String to, String subject, String text, String attachPath) {
        send(to, subject, text, (attachPath == null || attachPath.isEmpty())? null : new File(attachPath));
    }

    public void send(String to, String subject, String text, File... files) {
        try {
            // Create a default MimeMessage object.
            Message message = new MimeMessage(SESSION);

            // Set From: header field of the header.
            message.setFrom(new InternetAddress(USER_NAME));

            // Set To: header field of the header.
            message.setRecipients(Message.RecipientType.TO, InternetAddress.parse(to));

            // Set Subject: header field
            message.setSubject(subject);

            // Create the message part
            BodyPart bodyPart = new MimeBodyPart();

            // Now set the actual message
            bodyPart.setText(text);

            // Create a multipar message
            Multipart multipart = new MimeMultipart();

            // Set text message part
            multipart.addBodyPart(bodyPart);

            // Set Attachments
            addAttach(multipart, files);

            // Send the complete message parts
            message.setContent(multipart);

            // Send message
            Transport.send(message);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private void addAttach(Multipart multipart, File[] files) throws Exception {
        if (files == null) return;
        for (File file :files) {
            if (file.exists()) {
                BodyPart bodyPart = new MimeBodyPart();
                DataSource source = new FileDataSource(file);
                bodyPart.setDataHandler(new DataHandler(source));
                bodyPart.setFileName(file.getName());
                multipart.addBodyPart(bodyPart);
            } else {
                throw new FileNotFoundException(
                        String.format("File is not exist: %s", file.getAbsolutePath()));
            }
        }
    }

    enum SendType {
        TSL, SSL;
    }
}
```



