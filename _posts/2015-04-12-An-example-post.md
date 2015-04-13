---
layout: post
title: post例子模板
date: 2015-04-12 19:17:37
category: "Others"
---

post的例子模板。

# 标题一

## 标题二

### 标题三

* 序号1
* 序号2
* 序号3
* 序号4

** 序号a
** 序号b
** 序号c
** 序号d

下面是highlight
{% highlight java %}

@Configurable
@ComponentScan(basePackages = "com.9leg.java.spring")
@PropertySource(value = "classpath:spring/config.properties")
public class AppConfigTest {
    
    @Bean
    public PropertySourcesPlaceholderConfigurer propertyConfigInDev() {
        return new PropertySourcesPlaceholderConfigurer();
    }
    
}
{% endhighlight %}

字体加重**这些字体颜色特殊**

此处插入图片
<img src="http://git-liubao.github.io/liubao/images/header.jpg" />


超级链接:[超级链接](http://www.9leg.com/java/2015/01/09/java-convert-string-to-enum-object.html)
[超级链接2](http://kuaitizi.com/?r=7ffa784e42249e0f){:target="_blank"}