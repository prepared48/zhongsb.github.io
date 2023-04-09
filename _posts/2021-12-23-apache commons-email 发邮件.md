---
layout: post
title:  "apache commons-email 发邮件"
date:   2021-12-23 09:11:54
categories: Java
tags: Java 发邮件
mathjax: true
---

* content
{:toc}

Java 怎么实现一个给用户发邮件的需求呢？





入职新公司不久，接到一个给用户发邮件的需求，有两点需要说明的：1）正文需要格式化；2）需要带附件。

大概了解了一下需求，我马上开始思考，现有项目中是否有类似的接口可以支持，如果可以支持，那我就不用再开发了。

通过和老员工的沟通，发现以前有发邮件的接口。详细看了下代码，历史需求都是发送短信验证码，重置密码链接等需求，没有带附件的需求，也就没有带附件的接口。

既然要重新开发，想着肯定前人已经有过这种需求，然后再网上google一下，发现真的有。

## MultiPartEmail


```JAVA
public void doSendMulitiPartEmail(String subject, String sendHtml, String receiveUser, File attachment) {
    MultiPartEmail multiPartEmail = new MultiPartEmail();
    try {
        multiPartEmail.setHostName(mailHost);
        multiPartEmail.setAuthentication(sender_username, sender_password);
        // 发件人
        multiPartEmail.setFrom(sender_username, "prepared.com");
        // 收件人
        multiPartEmail.addTo(receiveUser);

        // 邮件主题
        multiPartEmail.setSubject(subject);
        multiPartEmail.setMsg(sendHtml);
//            multiPartEmail.setContent(sendHtml, "text/html; charset=utf-8");
        EmailAttachment attachmentLogs = new EmailAttachment();
        attachmentLogs.setPath(attachment.getPath());
        attachmentLogs.setDisposition(EmailAttachment.ATTACHMENT);
        attachmentLogs.setDescription("Logs");
        attachmentLogs.setName(attachment.getName());

        // 将multipart对象放到message中
        multiPartEmail.attach(attachmentLogs);

        multiPartEmail.send();
        System.out.println("send success!");
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (transport != null) {
            try {
                transport.close();
            } catch (MessagingException e) {
                e.printStackTrace();
            }
        }
    }
}
```    

但是这个代码有个问题：`multiPartEmail.setMsg(sendHtml);`通过setMsg设置正文，正文不支持html格式，也就是不能格式化，不满足需求1。

但是通过`multiPartEmail.setContent(sendHtml, "text/html; charset=utf-8")`可以支持正文格式化，但是附件又进不去了。不满足需求2。

通过查看官方文档：

> To create a multi-part email, call the default constructor and then you can call setMsg() to set the message and call the different attach() methods.

简单翻译一下：就是通过`setMsg（）`设置正文，通过`attach（）`设置附件。
应该是不支持同时格式化正文和增加附件。（如果有问题，可以指正一下）

## HtmlEmail

上代码

```JAVA
public void sendHTMLEmail (EmailMessage message) {
    try {
        HtmlEmail htmlEmail = new HtmlEmail();
        htmlEmail.setAuthentication(sender_username, sender_password);
        htmlEmail.setHostName(mailHost);
        htmlEmail.addTo(message.getEmailTo());
        htmlEmail.setFrom(sender_username, "prepared.com");
        htmlEmail.setSubject(message.getEmailSubJect());
        htmlEmail.setMsg(message.getEmailContent());
        htmlEmail.setCharset("utf-8");
        // 附件中文乱码解决
        String name = new String(message.getAttachFile().getName().toString().getBytes(),"utf-8");
        sun.misc.BASE64Encoder enc = new sun.misc.BASE64Encoder();
//            String name1="=?UTF-8?B?"+enc.encode("测试.txt".getBytes("utf-8"))+"?=";
        String fileName = "=?UTF-8?B?"+enc.encode(name.getBytes("utf-8"))+"?=";
        htmlEmail.embed(new FileDataSource(message.getAttachFile()), fileName);
        htmlEmail.send();
    } catch (EmailException e) {
        e.printStackTrace();
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    }
}
```

![](https://files.mdnice.com/user/13344/6db2e34b-c929-4af5-b3d0-590890cff158.png)

通过查看官方文档，很明显的说明，1）可以带附件；2）正文可以格式化，通过`setHtmlMsg()`方法即可。

当然，格式化正文，也可以使用`setMsg()`API.


![](https://files.mdnice.com/user/13344/9d128519-66ab-4e8c-a131-e012af80c9f6.png)

继续查看文档，可以发现，HtmlEmail 支持正文中嵌入图片、文件等内容（所以 api 名字叫 embed）。


使用这个的过程中，发现一个问题，如果附件文件名称带中文，可能会乱码，有两种方式解决。

方式一：

```
MimeUtility.encodeText(new String("带中文的文件名".getName().getBytes(),"utf-8"),"utf-8","B")
```

方式二：

```
String name = new String("带中文的文件名".getBytes(),"utf-8");
sun.misc.BASE64Encoder enc = new sun.misc.BASE64Encoder();
String fileName = "=?UTF-8?B?"+enc.encode(name.getBytes("utf-8"))+"?=";
```


## 总结

在这次需求中，做的不足的地方就是，没有先看官方文档，而是网上各种找现成代码，走了很多弯路。比如：1）刚开始觉得 HtmlEmail 不支持附件，转战 MultiPartEmail ；2）然后整了半天，发现这个不能格式化正文；3）最后又回到HtmlEmail。

官方文档很重要！！！

官方文档地址：https://commons.apache.org/proper/commons-email/javadocs/api-1.2/index.html



