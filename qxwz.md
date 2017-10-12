## 千寻位置接口设计
### QXWZController 千寻位置控制层
>
1. 查询是否有千寻连接权限：qxwz/canConnect,GET
    * 参数：token:令牌；projectId:项目id
    * 返回：OK:有权限;FAIL:无权限;EXPIRE:已过期
2. 请求千寻账号密码：qxwz/getAccount，GET
    * 参数：token:令牌；mac：手机mac地址；projectId:项目id
    * 返回：OK;FAIL:token失效，mac无效；UNKNOWN：连接池已满
3. 千寻心跳：qxwz/heardBeat，POST
    * 参数：token:令牌；userName:千寻账号
    * 返回：OK:发送成功;FAIL:token失效
4. 千寻过期时间: qxwz/setTimeout,POST
    * 参数：token:令牌；userName:千寻账号；timeout:过期时间
    * 返回：OK:发送成功;FAIL:token失效
5. 添加千寻账号: qxwz/admin/create,POST
    * 参数：userName:千寻账号；password:千寻密码;timeout:过期时间
    * 返回：OK:发送成功;FAIL:添加失败
6. 删除千寻账号：qxwz/admin/delete/{id},DELETE
    * 参数：userName:千寻账号；password:千寻密码;timeout:过期时间
    * 返回：OK:发送成功;FAIL:添加失败
7. 获取千寻列表：qxwz/admin/lists, GET
    * 参数：
    * 返回：OK (千寻对象列表)
