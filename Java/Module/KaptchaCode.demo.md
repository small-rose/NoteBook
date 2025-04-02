---
layout: default
title: Java  Kaptcha
parent: Java
nav_order: 93
---

Here are  Kaptcha code experience .
{: .fs-6 .fw-300 }


## Here are Kaptcha code demo .
{: .no_toc .text-delta }

1. TOC
{:toc}


## Kaptcha 

Kaptcha 是一个可高度配置的实用验证码生成工具，可自由配置的选项如：
 
- 验证码的字体
- 验证码字体的大小
- 验证码字体的字体颜色
- 验证码内容的范围(数字，字母，中文汉字！)
- 验证码图片的大小，边框，边框粗细，边框颜色
- 验证码的干扰线
- 验证码的样式(鱼眼样式、3D、普通模糊、…)


## Kaptcha 使用


### 1、添加依赖

一般使用

```xml
<dependency>
      <groupId>com.github.penggle</groupId>
      <artifactId>kaptcha</artifactId>
      <version>2.3.2</version>
</dependency>
```

无漏洞版本

```xml
<dependency>
    <groupId>pro.fessional</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.3</version>
</dependency>
```

### 2、编写配置

```java
import com.google.code.kaptcha.impl.DefaultKaptcha;
import com.google.code.kaptcha.util.Config;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;
import java.util.Properties;

@Component
public class KaptchaConfig {
    @Bean
    public DefaultKaptcha getDefaultKaptcha() {
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        Properties properties = new Properties();
        // 图片边框
        properties.setProperty("kaptcha.border", "no");
        // 边框颜色
        properties.setProperty("kaptcha.border.color", "black");// 105.179,90
        //边框厚度
        properties.setProperty("kaptcha.border.thickness", "1");
        // 图片宽
        properties.setProperty("kaptcha.image.width", "120");
        // 图片高
        properties.setProperty("kaptcha.image.height", "60");
        //图片实现类
        properties.setProperty("kaptcha.producer.impl", "com.google.code.kaptcha.impl.DefaultKaptcha");
        //文本实现类
        properties.setProperty("kaptcha.textproducer.impl", "com.google.code.kaptcha.text.impl.DefaultTextCreator");
        //文本集合，验证码值从此集合中获取
        properties.setProperty("kaptcha.textproducer.char.string", "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ");
        //验证码长度
        properties.setProperty("kaptcha.textproducer.char.length", "4");
        //字体
        properties.setProperty("kaptcha.textproducer.font.names", "Cambria,楷体,宋体,微软雅黑");
        //字体颜色
        properties.setProperty("kaptcha.textproducer.font.color", "136,68,17");
        //文字间隔
        properties.setProperty("kaptcha.textproducer.char.space", "5");
        //干扰实现类
        properties.setProperty("kaptcha.noise.impl", "com.google.code.kaptcha.impl.DefaultNoise");
        //干扰颜色
        properties.setProperty("kaptcha.noise.color", "blue");
        //干扰图片样式
        properties.setProperty("kaptcha.obscurificator.impl", "com.google.code.kaptcha.impl.WaterRipple");
        //干扰图片样式 - 无水印
        properties.setProperty("kaptcha.obscurificator.impl", "com.small.code.kaptcha.NoWaterRipple");
        //背景实现类
        properties.setProperty("kaptcha.background.impl", "com.google.code.kaptcha.impl.DefaultBackground");
        //背景颜色渐变，开始颜色
        properties.setProperty("kaptcha.background.clear.from", "white");
        //背景颜色渐变，结束颜色
        properties.setProperty("kaptcha.background.clear.to", "white");
        //文字渲染器
        properties.setProperty("kaptcha.word.impl", "com.google.code.kaptcha.text.impl.DefaultWordRenderer");

        Config config = new Config(properties);
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }

}
```
  
无水印的实现

```java
package com.small.code.kaptcha;
import com.google.code.kaptcha.GimpyEngine;
import com.google.code.kaptcha.NoiseProducer;
import com.google.code.kaptcha.util.Configurable;
import java.awt.Graphics2D;
import java.awt.Color;
import java.awt.image.BufferedImage;
import java.awt.image.Imageobserver;
/**
* 无水印实现
*/
public class NoWaterRipple extends Configurable implements GimpyEngine {

    @override
    public BufferedImage getDistortedImage(BufferedImage baseImage) {
        NoiseProducer noiseProducer = this.getConfig().getNoiseImpl() ;
        BufferedImage distortedImage = new BufferedImage (baseImage.getWidth(), baseImage.getHeight(), 2) ;
        Graphics2D graphics = (Graphics2D) distortedImage.getGraphics() ;
        graphics.drawImage(baseImage, 0, 0, (Color)null, (Imageobserver)null) ;graphics.dispose () ;
        noiseProducer.makeNoise(distortedImage, 0.1F, 0.1F, 0.25F, 0.25F) ;
        noiseProducer.makeNoise(distortedImage, 0.1F, 0.25F, 0.5F, 0.9F);
        return distortedImage;
    }
}
```  
  
### 4、后端请求实现


```java
package com.small.code.kaptcha.service;

import com.google.code.kaptcha.impl.DefaultKaptcha;
import org.springframework.stereotype.Service;

import javax.imageio.ImageIO;
import javax.servlet.ServletoutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.awt.image.BufferedImage;
import java.io.ByteArrayoutputStream;

@Service
public class KaptchaCodeService {
    @Resource
    private DefaultKaptcha  defaultKaptcha ;
    /**
    *生成验证码
    @param request
    @throws Exception
    @param response
    **/
    public void generateKaptchaCode (HttpServletRequest request, HttpServletResponse response) throws Exception {
        byte[] captchaChallengeAsJpeg = null;
        ByteArrayoutputStream jpegoutputStream = new ByteArrayoutputStream() ;
        try { //生产验证码字符串并保存到session中
            String createText = defaultKaptcha.createText() ;
            String tickit = SaltEncryption.encrypt(createText, SaltEncryption.SALT _RANDoM) ;
            request.getSession() .setAttribute("tickit", tickit) ;//使用生产的验证码字符串返回一个BufferedImage对象并转为byte写入到byte数组中
            BufferedImage challenge = defaultKaptcha.createImage (createText) ;
            ImageIo.write(challenge, "jpg", jpegoutputStream) ;
        } catch (IllegalArgumentException e) { 
            response.sendError(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        //定义response输出类型为image/jpeg类型，使用response输出流输出图片的byte数组
        captchaChallengeAsJpeg = jpegOutputStream.toByteArray() ;
        response.setHeader("Cache-Control", "no-store");
        response.setHeader("Pragma", "no-cache") ;
        response.setDateHeader("Expires", 0) ;
        response.setContentType("static.image/jpeg") ;
        ServletoutputStream responseoutputStream = response.getoutputStream();
        responseoutputStream.write (captchaChallengeAsJpeg) ; 
        responseoutputStream.flush() ; 
        responseoutputstream.close() ;
    }
}
```

```java
public class IndexController {
    
    @Resource
    private KaptchaCodeService kaptchaCodeService ;

    @RequestMapping(value="/getCode", method=RequestMethod.GET)
    @ResponseBody
    public void getcode(HttpServletRequest request, HttpServletResponse response) throws Exception {
        kaptchaCodeService.generateKaptchacode(request, response);
    }
}
```

对应的前端代码片段

```html
<form id="loginForm">
<img id="kaptcha_img" class="kaptcha" src="/getcode"/>
</form>

```
绑定一个刷新

```javascript
$(function(){

    function refreshKatpcha(){
        $("#captcha_image").attr("src","getCode?time="+(new Date()).getTime());
    }

    $("#loginForm .captcha").on('click',function() {
        refreshKatpcha();
    });
});


```

JSON 返回 base64 也可以

```java
public class IndexController {
    //生成验证码
    @RequestMapping("/Code")
    public ResultVo Code() throws IOException {
            
        // 生成文字验证码
        String text = defaultKaptcha.createText() ;
        System.out.println("文字验证码为"+text);//生成图片验证码
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        BufferedImage image = defaultKaptcha.createImage(text);
        
        ImageIO.write(image,"jpg",out) ;//对字节组Base64编码
        return ResultVo.success("img",Base64.getEncoder().encodeToString(out.toByteArray()));
    }
}
```

使用base64 的时候前端片段

```
<el-input placeholder="请输入验证码" v-model="user.code">
<template slot="append" class="input_append">
<img src="" alt="" id="codeImg" style=" height: 34px" @click="getCode()">
</template>
</el-input>
```

```
getCode(){
    this.axios({
        method: "post",
        url: "http://localhost:8182/Code"
    }).then(res => {
        console.log(res);
        document.getElementById("codeImg").src = 'data:image/jpeg;base64,' + res.data.img;
    });
}
```

> base64 部分的代码参考 https://blog.csdn.net/weixin_57467236/article/details/126973665

