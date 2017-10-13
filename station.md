## 水位站接口设计
### 一. 涉及对象

<div align="center">
   <img src="http://112.124.104.190:10001/soft/wiki/uploads/cc01c724b094a678eeeb2595409764fd/station.png" width="680px" />
</div>


### 二. 后台stationCtroller接口 
>
1. **获取测站列表包括分享的测站:/list,GET**
    * 返回：OK(jsonObjectList)
    * jsonObjectList里包含测站所有信息(河道模型，流量曲线)，仪表和摄像头, 如果没有喝到模型，返回空数组
2. **获取测站列表包括分享的测站:/list,GET**
    * 返回：OK(jsonObjectList)
    * jsonObjectList里包含测站所有信息(河道模型，流量曲线)，仪表和摄像头, 如果没有河道模型，返回空数组
3. **修改测站：/edit,POST**
    * 参数：坐标必须是合法的,测站照片名是固定的，所以不用传递给后台
    * 返回：OK，添加成功，FAIL，参数有误，NO_AUTH：不是自己的测站
    * 测站图片由前台上传到阿里云，测站照片名是固定的，所以不用传递给后台
4. **添加仪表：addSensor，POST**
    * 参数：唯一编码，激活码必须，安装高度必须是浮点数，限制在0-100.0f，电话必须合法
    * 返回：OK，添加成功，FAIL，参数有误,EXIT:已存在，NO_AUTH：不是自己的测站
5. **添加摄像头：addCamera,POST**
    * 参数：电话必须合法,播放url合法
    * 返回：OK，添加成功，FAIL，参数有误,EXIT:已存在，NO_AUTH：不是自己的测站
6. **编辑仪表：editSensor，POST**
    * 参数：唯一编码，激活码必须，安装高度必须是浮点数，限制在0-200.0f，电话必须合法
    * 返回：OK，添加成功，FAIL，参数有误,EXIT:仪表不存在，NO_AUTH：不是自己的测站
7. **编辑摄像头：editCamera,POST**
    * 参数：电话必须合法,播放url合法
    * 返回：OK，添加成功，FAIL，参数有误,EXIT:仪表不存在，，NO_AUTH：不是自己的测站
8. **删除仪表：deleteSensor,DELTE**
    * 参数：id
    * 返回：OK,EXIT:仪表不存在，NO_AUTH：不是自己的测站
9. **删除摄像头：deleteCamera，DELTEE**
    * 参数：id
    * 返回：OK,EXIT:仪表不存在，NO_AUTH：不是自己的测站
10. **上传模型：uploadModel，POST**
    * 参数：id：测站id，file: 文件
    * 返回：OK 上传成功，FIAL：格式错误，NO_AUTH：不是自己的测站
11. **下载模型：downloadModel,GET**
    * 参数：id：测站id，file: 文件
    * 返回：OK 上传成功，FIAL：格式错误，NO_AUTH：不是自己的测站
12. **参数设置：editParameter,POST**
    * 参数：液位上下限：限制在0-100.0f，电话：合法
    * 返回：OK 上传成功，FIAL：格式错误，NO_AUTH：不是自己的测站
13. **请求token：token,GET,前台获取的用于和检测系统沟通，取得数据**
    * 有问题，请求token应该放在这个控制层，而不是sensorController??
    * 参数：
    * 返回：OK(token字符串)
14. **监测取得仪表列表：getParamters，GET**
    * 这是监测系统取得所有已改变的参数列表
    * 参数：token
    * 返回：OK：返回包含仪表编码和参数，格式：`{code:xx,paramter:{xx}}`
15. **测站分享：shares，POST**
    * 参数：userIds:欲分享的用户(id用‘，’隔开)，sensorIds:欲分享的测站(id用‘，’隔开)
    * 返回：OK
16. 取消测站分享：/station/unShare POST
    * 参数：Long sensorId，欲取消分享的仪表，String userIds，用逗号隔开，示例("12,34,24")
    * 返回：OK
    * 后台修改内容,station实体类添加share字段，在实体Station包下增加Share,ShareVisit类，和project分享的实体类Share,ShareVisit类似

### 二. 后台TradeController接口
>
1. **购买测站：createStation,POST**
    * 购买成功后可以使用一个月
    * 参数：stationType:欲购买的测站类型
    * 返回：OK(订单对象):成功;FAIL:参数错误
2. **测站续费:renewStation,POST**
    * 测站续费每次一年，成功后修改测站到期时间
    * 参数：instanceId：测站编码
    * 返回：OK(订单对象):成功；FAIL:参数错误

### 四. 前台与检测系统的接口
>
1. 取得宏电数据：www.qingqingshuili.cn:8080/irtu/{code} GET
2. 取得仪表数据：www.qingqingshuili.cn:8080/sensor/{code} GET
3. 取得仪表报警日志：www.qingqingshuili.cn:8080/sensor_warning/{code} GET
    
### 五. 后台调用检测系统接口
>
1. 取得仪表列表：www.qingqingshuili.cn:8080/sensor，GET
    * 参数：token
    * 返回：OK(仪表列表)

### 六. 相关后台实体类
```java
// 测站
public class Station extends BaseEntity {

    private static final long serialVersionUID = 1753324356353389598L;
    /** 站名 */
    private String name;
    /** 描述 */
    private String description;
    /** 站类型 */
    private CommonEnum.StationType type;
    /** 坐标 */
    private String coor;
    /** 位置 */
    private String address;
    /** 河道模型 */
    private String riverModel;
    /** 流量曲线 */
    private String flowModel;
    /** 参数 */
    private String parameter;
    /** 测站唯一标识 */
    private String instanceId;
    /** 测站分享 */
    private String shares;
    /** 是否修改过 */
    private boolean transform;
    /** 到期时间 */
    private Date expireDate;
    /** 测站图片(阿里云图片路径) */
    private String  picture;

    private User user;

    /** 河底高程*/
    private Double bottomElevation;
}
// 仪表
public class Sensor extends BaseEntity {

    private static final long serialVersionUID = -3097083860247123817L;
    /** 唯一编码 */
    private String code;
    /** 类型 */
    private Type type;
    /**　是否激活 */
    private boolean activate;
    /** 厂家factroy，联系人contact，电话phone等Json */
    private String info;
    /** 摄像头 */
    private String cameraUrl;

    private Station station;
    /** 安装高度*/
    private Double settingHeight;
}
// 摄像头
public class Camera {
    private Station station;
    /** 摄像头 */
    private String cameraUrl;
    /** 厂家factroy，联系人contact，电话phone等Json */
    private String info;    
}
```



