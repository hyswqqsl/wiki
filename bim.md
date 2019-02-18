<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [BIM参数传递](#bim%E5%8F%82%E6%95%B0%E4%BC%A0%E9%80%92)
  - [一. 消力池](#%E4%B8%80-%E6%B6%88%E5%8A%9B%E6%B1%A0)
  - [二. 检查井](#%E4%BA%8C-%E6%A3%80%E6%9F%A5%E4%BA%95)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## BIM参数传递
unity程序发布为webgl后，可以接收到前台发送的参数，使用的接口是：
```
  int pm = Application.absoluteURL.IndexOf("?");
  if (pm != -1)
  {
      sceneName = Application.absoluteURL.Split("?")[1];
  }
```
### 一. 消力池
消力池有六个参数在后台定义是这样的：
```
    <attribeGroup alias="5" status="normal" name="控制尺寸">
        <dimension name="设计标高" alias="D5BG" unit="m" type="number"/>
        <dimension name="长度a" alias="D51" unit="m" type="number"/>
        <dimension name="宽度b" alias="D52" unit="m" type="number"/>
        <dimension name="池深h1" alias="D53" unit="m" type="number"/>
        <dimension name="消力坎高h2" alias="D54" unit="m" type="number"/>
        <dimension name="底板厚度t" alias="D55" unit="m" type="number"/>
   </attribeGroup>
```

浏览器上的URL格式为：

```
http://xxxx.com/webgl/xiaolc?type=xiaolc&parameter={D5BG:xx,D51:xx,D52:xx,D53:xx,D54:xx,D55:xx}
```
### 二. 检查井
检查井参数在后台定义是这样的：

```
    <attribeGroup alias="15" status="normal" name="控制尺寸">
        <dimension name="设计标高" alias="D15BG" unit="m" type="number"/>
        <dimension name="井深h" alias="D151" unit="m" type="number"/>
        <dimension name="直径d" alias="D152" unit="m" type="number"/>
        <dimension name="壁厚t" alias="D153" unit="m" type="number"/>
    </attribeGroup>
```
前台会把参数以json方式发给BIM，格式如下：

```
http://xxxx.com/webgl/xiaolc?type=jcj&parameter={D15BG:xx,D151:xx,D152:xx,D153:xx}
```


