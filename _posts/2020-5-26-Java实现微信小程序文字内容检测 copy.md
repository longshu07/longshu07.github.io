---
layout:     post                    # 使用的布局（不需要改）
title:      Java实现微信小程序文字内容检测               # 标题 
subtitle:  Java后台检查一段文本是否含有违法违规内容。security.msgSecCheck #副标题
date:       2020-05-26              # 时间
author:     BY 龙叔                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片

catalog: true                       # 是否归档

tags:                               #标签
    - 微信小程序
---

@[TOC](文章目录)

# Java实现微信小程序文字内容检测

本人在做毕设时，想要将微信小程序正式上线，但是无法通过审核，原因是没有对发布的内容进行审核。而微信开发平台提供了一个开放接口，可以在服务端实现这个接口，对一段文字进行审核。[点击进入微信开放接口文档](https://developers.weixin.qq.com/minigame/dev/api-backend/open-api/sec-check/security.msgSecCheck.html#HTTPS%20%E8%B0%83%E7%94%A8)

## 1.微信开放接口描述

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200526224022135.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzY1ODY3,size_16,color_FFFFFF,t_70)
请求地址：

```java
POST https://api.weixin.qq.com/wxa/msg_sec_check?access_token=ACCESS_TOKEN
```

可以看到在请求之前我们需要获取**access_token**，才可调用这个接口。
接下来看如何获取AccessToken，如果已经会此步骤的可以跳过。

## 2.获取AccessToken

请求地址：

```Java
https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code
```

在请求之前，需要**appid**、**secret**这两个参数，都需要从微信小程序平台获取。通过OKhttp（只需在Maven中导入maven依赖即可）。通过对其调用，即可获取AccessToken。AccessToken的有效期为2小时，2小时失效，需要重新获取最新的才可调用其他的接口。
maven依赖如下：

```java
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>3.8.1</version>
        </dependency>
```

Java中获取AccessToken代码入下：

```java
//获取AccessToken
public String getAccessToken() {
//%s是占位符
        String url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=%s&secret=%s";
        OkHttpClient okHttpClient = new OkHttpClient();
        Request request = new Request.Builder()
                .addHeader("content-type", "application/json")
                //appid,secret修改为你个人的
                .url(String.format(url, appid, secret))
                .build();
        try {
            Response execute = okHttpClient.newCall(request).execute();
            if (execute.isSuccessful()) {
                String getAccessTokenString = execute.body().string();
                System.out.println("getAccessTokenString:" + getAccessTokenString);
                return getAccessTokenString;
            } else {
            //这里抛出错误，可自行定义
                throw new RuntimeException();
            }

        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```

## 3调用微信开放接口msgSecCheck检测文字内容 

请求地址：

```java
POST https://api.weixin.qq.com/wxa/msg_sec_check?access_token=ACCESS_TOKEN
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200526230019261.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzY1ODY3,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200526230042672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzY1ODY3,size_16,color_FFFFFF,t_70)
 从上面可以看到是post请求，请求参数有**access_token**是接口调用凭证，**content**	要检测的文本内容，长度不超过 500KB。**accessToken是第二个步骤中获取的AccessTokentoken才能有权限正常接入接口**。
以下代码如果复制黏贴有错误，需要导入一定的maven依赖即可。

```java
<!--调用 http 请求-->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.3</version>
        </dependency>
```

Java后端请求代码如下：

```java
   	//使用http导入的部分包，以防导入出错，可以查看对比一番
   	import org.apache.http.client.HttpClient;
	import org.apache.http.client.methods.HttpPost;
	import org.apache.http.entity.ByteArrayEntity;
	import org.apache.http.entity.ContentType;
	import org.apache.http.entity.StringEntity;
	import org.apache.http.impl.client.HttpClients;
	import org.apache.http.util.EntityUtils;
	 /**
     * 校验输入文字信息
     * 输入AccessToken以及要检测的文字
     * accessToken是第二个步骤中获取的AccessTokentoken才能有权限正常接入接口。
     */
    public void checkMsg(String msg, String accessToken){
        String url = "https://api.weixin.qq.com/wxa/msg_sec_check?access_token=" + accessToken;
        //创建客户端
        HttpClient httpclient = HttpClients.createDefault();
        //创建一个post请求
        HttpPost request = new HttpPost(url);
        //设置响应头
        request.setHeader("Content-Type", "application/json;charset=UTF-8");
        //通过fastJson设置json数据
        JSONObject postData = new JSONObject();
        //设置要检测的内容
        postData.put("content", msg);
        String jsonString = postData.toString();
        request.setEntity(new StringEntity(jsonString,"utf-8"));
        //// 由客户端执行(发送)请求
        try {
            HttpResponse response = httpclient.execute(request);
            // 从响应模型中获取响应实体
            HttpEntity entity = response.getEntity();
            //得到响应结果
            String result = EntityUtils.toString(entity,"utf-8");
            //打印检测结果
            System.out.println(result);
            //将响应结果变成json
            //JSONObject resultJsonObject = JSONObject.parseObject(result, JSONObject.class);
            

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

通过上面的代码即可获取到进行文字内容检测，
官方测试用例

```
特3456书yuuo莞6543李zxcz蒜7782法fgnv级
完2347全dfji试3726测asad感3847知qwez到
```

输入上面的文字，返回

```
87014	内容含有违法违规内容
```

即接入接口成功。
至此，文字检测结束。如有什么问题，欢迎━(*｀∀´*)ノ亻!评论和私聊(*￣︶￣)
下次更新**Java实现微信小程序图片检测**