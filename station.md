## 水位站接口设计

### 一. 后台stationController接口 
1. **获取测站列表包括分享的测站:/station/list,GET**
    * 返回：OK(jsonObjectList)
    * jsonObjectList里包含测站所有信息(河道模型，流量曲线)，仪表和摄像头, 如果没有喝到模型，返回空数组
2. **获取测站列表包括分享的测站:/station/list,GET**
    * 返回：OK(jsonObjectList)
    * jsonObjectList里包含测站所有信息(河道模型，流量曲线)，仪表和摄像头, 如果没有河道模型，返回空数组
3. 安布雷拉水文用户新建站点,/station/abll/create,POST
    * 独立水文用户建立站点时，不用判断套餐先限制
    * 每个独立水文用户最多建立1000个测站
    * 后台生成唯一编码, 过期时间2099
    * 参数：
        * name: 测站名称
        * type: 普通类型
        * remark: 测站描述
    * 返回： 
        * OK： 新建成功
        * DATA_LOCK: 测站数量超限
        * UNAUTHORIZED: 不是安布雷拉用户      
4. **修改测站：/station/edit,POST**
    * 测站图片由前台上传到阿里云，测站照片路径是固定的，所以不用传递给后台
    * 参数：
        * id，测站id
        * name: 测站名
        * type: 默认传递普通测站
        * description:描述
        * address: 行政区地址
        * coor: 坐标
    * 返回：
        * OK，添加成功，
        * FAIL，参数有误
        * DATA_REFUSE：不是自己的测站
5. **添加仪表：/station/addSensor，POST**
    * 参数：
        * id 测站id
        * name: 仪表名
        * code 仪表编码
        * ciphertext 激活码
        * type 仪表类型
    * 返回：
        * OK，添加成功
        * FAIL，参数有误
        * DATA_EXIST: 编码已存在
        * DATA_REFUSE：不是自己的测站
6. **添加摄像头：/station/addCamera,POST**
    * 参数：
        * id: 测站id
        * name: 摄像头名
        * code: 摄像头编码
        * password: 密码 
    * 返回：
        * OK: 添加成功
        * FAIL: 参数有误
        * DATA_EXIST: 编码已存在
        * DATA_REFUSE：不是自己的测站
7. **编辑仪表：/station/editSensor，POST**
    * 编码,activate不可修改
    * isChanged，由后台设置保存
    * 前台判断扩展属性是否有修改，修改了把扩展属性加到extraParameters数组中 
    * 参数：
        * id: 仪表id
        * name： 仪表名
        * description： 仪表描述
        * type：类型
        * factory: 厂家
        * contact: 联系人
        * phone: 电话
        * settingHeight: 安装高度
        * settingElevation: 安装海拔
        * settingAddress：安装地点
        * measureRange: 测量范围
        * maxValue: 测量上限
        * isMaxValueNote: 超过测量上限是否发送短信
        * minValue: 测量下限
        * isMinValueNote: 低于测量下限是否发送短信
        * extraParameters: 扩展属性，[{id:xxx,value:xxx},{...}]
     * 返回：
        * OK，添加成功
        * FAIL，参数有误
        * DATA_NOEXIST: 仪表或扩展属性不存在
        * DATA_REFUSE：不是自己的仪表
8. 仪表添加自定义属性：/station/sensor/extra/create,POST
    * 自定义扩展属性最多添加20个
    * 添加的类型自动是自定义类型
    * 自定义扩展属性都是字符串格式
    * 自定义属性名使用displayName
    * 参数：
       * id：仪表id
       * displayName: 属性名
     * 返回：
        * OK，添加成功
        * DATA_REFUSE：不是自己的仪表
        * DATA_LOCK: 属性超多20个，不能再添加
9. 仪表删除自定义属性: /station/sensor/extra/delete,DELETE
    * 参数：
       * id：自定义属性id
     * 返回：
        * OK，添加成功
        * DATA_REFUSE：不是自己的仪表属性
        * DATA_LOCK: 属性类型是系统属性，不能删除
10. **编辑摄像头：/station/editCamera,POST**
    * 编码不可修改
    * 参数：
        * id: 摄像头id
        * name: 摄像头名
        * description: 描述
        * factory：厂家
        * contact: 联系人
        * phone: 电话
        * settingAddress：安装地点
        * password: 密码         
    * 返回：
        * OK，添加成功
        * FAIL，参数有误
        * DATA_NOEXIST: 摄像头不存在
11. **删除仪表：/station/deleteSensor,DELTE**
   * 删除仪表时，需要经扩展属性删除
    * 参数：id
    * 返回：
        * OK,EXIT:仪表不存在
        * DATA_REFUSE：不是自己的仪表
12. **删除摄像头：/station/deleteCamera，DELTEE**
    * 参数：id
    * 返回：
        * OK,EXIT:仪表不存在
        * DATA_REFUSE：不是自己的摄像头
13. **上传模型：/station/uploadModel，POST**
    * 参数：
        * id：测站id
        * file: 文件
    * 返回：
        * OK 上传成功
        * FIAL：格式错误
        * DATA_REFUSE：不是自己的测站
14. **下载模型：/station/downloadModel,GET**
    * 参数：
        * id：测站id
        * file: 文件
    * 返回：
        * OK 上传成功
        * FIAL：格式错误
        * DATA_REFUSE：不是自己的测站        
15. 获取token,/station/token,GET
    * 前台访问水利云后台
    * 这个接口返回的token包含用户信息，可以在水云端验证用户身份
    * 参数：无
    * 返回：
        * ok,data:{"token":"","noticeStr":""}
        * NO_SESSION   
16. 效验token,/station/intendedEffectToken,GET
    * 监测访问水利云后台
    * 参数：
        * code：仪表唯一编码
        * token：token
        * noticeStr：随机码
    * 返回：
        * OK：成功
        * UNAUTHORIZED：无权限             
17. **监测取得测站参数：/station/sensor/getParameters，GET**
    * 监测访问水利云后台
    * 监测子系统更改取得仪表参数，用于报警，报警只针对仪表，更简单
    * 这是监测系统取得所有已改变的参数列表
    * 参数：token
    * 返回：
        * OK：返回包含测站唯一编码，所有仪表编码，以及参数，格式：
        [{code:xxx,maxValue:xxx,isMaxValueWaring,minValue:xxx,isMinValueWaring,contact:xxx,phone:xxx}]        
18. 测站分享和协同不变，参见[[http://112.124.104.190:10001/soft/wiki/wikis/share]]

### 二. 后台TradeController接口
>
1. **购买测站：createStation,POST**
    * 购买成功后可以使用一个月
    * 参数：stationType:欲购买的测站类型
    * 返回：OK(订单对象):成功;FAIL:参数错误
2. **测站服务续费:renewStationService,POST ??**
    * 测站服务续费每次一年，成功后修改测站到期时间
    * 参数：instanceId：测站编码
    * 返回：OK(订单对象):成功；FAIL:参数错误

### 二. 相关后台实体类
```java
/**
 * 测站类型
 */
public enum StationType{
    /** 普通测站 */  
    NORMAL_STATION,
    /** 水文站 */
    HYDROLOGIC_STATION,
    /** 雨量站 */
    RAINFALL_STATION,
    /** 水位站 */
    WATER_LEVEL_STATION,
    /** 水质站 */
    WATER_QUALITY_STATION
}
```

````
oss图片地址
qqsl bucket
  |
  |->station（测站）
     |
     |->[测站唯一编码]
       |
       |->[图片唯一编码].jpd
  sensor(仪表)
     |
     |->[仪表code]
       |
       |->[图片唯一编码].jpd      
```



