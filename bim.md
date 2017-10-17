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
前台会把参数以json方式发给BIM，格式如下：

```
{D5BG:xx,D51:xx,D52:xx,D53:xx,D54:xx,D55:xx}
```
浏览器上个URL格式为：

```
http://xxxx.com/webgl/xiaol?parameter="{D5BG:xx,D51:xx,D52:xx,D53:xx,D54:xx,D55:xx}"
```

