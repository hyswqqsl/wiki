## 套餐接口设计

<div align="center">
   <img src="http://112.124.104.190:10001/soft/wiki/uploads/698c785380709211e98c097ccb282ea7/package.png" width="680px" />
   
   <img src="http://112.124.104.190:10001/soft/wiki/uploads/33a0228e9e7cd9fd2917ae274c952929/createPackage_ali.png" width="680px" />
   
   <img src="http://112.124.104.190:10001/soft/wiki/uploads/3e9cea4eba50dd08d328457ce8a0a14b/createPackage_weixin.png" width="680px" />
</div>

### 一. tradeController 订单控制层
>
1. 套餐购买:在点击'提交订单'时调用， /trade/createPackage POST
    * 参数：packageType：欲购买的配置类型
    * 返回：OK(订单对象):订单建立成功，跳转到支付界面，EXIST：有未处理的订单，UNKNOWN:不符合购买条件，NOCERTIFY:未认证
2. 配置续费: 在点击'提交订单'时调用，/trade/renewPackage POST 
    * 参数：唯一标识：instanceId
    * 返回：OK(订单对象):订单建立成功，跳转到支付界面，EXIST：有未处理的订单，UNKNOWN:未知错误
3. 配置升级： 在点击'提交订单'时调用，/trade/updatePackage POST

    * 参数：唯一标识：instanceId,预升级的配置：packageType
    * 返回：OK(订单对象):订单建立成功，跳转到支付界面，EXIST：有未处理的订单，UNKNOWN:未知错误
4. 获取用户订单列表: 获取订单列表 /trade/lists GET
    * 参数：
    * 返回：OK(订单列表)，UNKNOWN:未知错误
5. 用户获取订单详情 /trade/trade/{outTradeNo} GET
    * 参数：outTradeNo:
    * 返回：OK(订单对象)，UNKNOWN:如果是用户，需要判断是不是自己的订单，如果不是，返回UNKNOWN
6. 管理员获取订单详情 /trade/admin/trade/{outTradeNo} GET
    * 参数：outTradeNo
    * 返回：OK(订单对象)
7. 删除订单 /trade/remove POST
    * 参数：outTradeNo
    * 返回：OK:删除成功，FAIL:未支付，交易中的订单，不能删除；UNKNOWN:需要判断是不是自己的订单，如果不是，返回UNKNOWN
    * 注意：订单并不是真实删除，只是改变删除状态
8. 关闭订单 /trade/close POST
    * 参数：outTradeNo
    * 返回：OK:关闭成功，UNKNOWN:需要判断是不是自己的订单，如果不是，返回UNKNOWN
9. **订单remark属性：{"type":"UPDATE","oldType":"青春版","newType":"朝阳版", expireDate:过期时间(long)}**
        
### 二. turnoverController 流水控制层
>
1. 管理员获取流水 /turnover/admin/lists GET
   * 参数：begin:开始时间,long型，end：结束时间，long型
   * 返回：OK(流水列表)

### 三.  WXPayController 微信支付控制层
>
1. 获取微信支付信息:订单生产成功后，前台跳转到支付界面，调用此接口获得微信支付二维码，/wxPay/unifiedOrderPay/{outTradeNo} GET 
    * 参数：outTradeNo:订单号
    * 返回：OK(qrCode 二维码支付地址，需前台生成二维码):成功,FAIL:参数验证失败
2. payNotice: 微信主动发送的支付成功回调 
    * 成功后发出短信提醒
3. 管理员获取微信订单状态：wxPay/admin/queryTrade/{outTradeNo} ??
    * 参数:outTradeNo:订单号
    * 返回：OK(tradeState:微信订单状态)
4. 管理员根据微信订单更新订单: wxPay/admin/refreshTrade POST
    * 成功后发出短信提醒
    * 参数：outTradeNo
    * 返回：OK

### 四. aliPayController 支付宝控制层
>
1. 获取支付宝信息:订单生产成功后，前台跳转到支付界面，调用此接口获得支付页面，/aliPay/pcPay/{outTradeNo} GET 
    * 参数：outTradeNo:订单号
    * 返回：直接将完整的表单html输出到页面
2. payNotice:支付宝主动发送的支付成功回调
    * 成功后发出短信提醒
3. 管理员取支付宝订单状态：alyPay/admin/queryTrade/{outTradeNo}
    * 参数:outTradeNo:订单号
    * 返回：OK(tradeState:订单状态)    
    * 参数:outTradeNo:订单号
    * 返回：OK(tradeState:订单状态)         
<pre>
    // ACQ.SYSTEM_ERROR	系统错误重新发起请求
    // ACQ.INVALID_PARAMETER	参数无效检查请求参数，修改后重新发起请求
    // ACQ.TRADE_NOT_EXIST  查询的交易不存在  检查传入的交易号是否正确，修改后重新发起请求
    // WAIT_BUYER_PAY（交易创建，等待买家付款）
    // TRADE_CLOSED（未付款交易超时关闭，或支付完成后全额退款）
    // TRADE_SUCCESS（交易支付成功）
    // TRADE_FINISHED（交易结束，不可退款）
</pre>
4. 管理员根据支付宝订单更新订单: aliPay/admin/refreshTrade POST
    * 成功后发出短信提醒
    * 参数：outTradeNo
    * 返回：OK
       
### 五. modelControlelr 模板控制层
>
1. 套餐模板列表: /model/packages, GET
    * 参数：无
    * 返回：OK(套餐模板列表),list方式
2. 测站模板列表：/model/stations, GET
    * 参数：无
    * 返回：OK(测站模板列表),list方式

### 六. packageController 水利云配置控制层
>
1. 获取用户当前配置：/package/getPackage ,每次重新取，还是配置购买，升级后由前台动态更新
    * 参数：无
    * 返回：OK(当前配置对象)

### 七. 套餐过期
套餐过期后，所有操作都不能进行，如建立项目、绑定子账号、编辑项目要素、上传文件、上传工程布置、地图上绘图、编辑水文站、不能上传外业勘测数据，这些涉及到的操作都要判断套餐是否过期，这些操作后台上都加上判断套餐未过期标签IsExpire，返回EXPIRED类型的消息；前台根据用户下的套餐对象状态，也进行限制，不用到后台验证，提高效率，涉及到的接口有：
>
1. 建立项目，project/new POST
2. 编辑项目，project/update POST
3. 向项目的业主和负责人发送短信,project/sendMessage POST
    * **增加projectId**
4. 企业间分享项目，project/share POST
5. 将一个项目的多个权限分享给一个子账号,project/cooperateMul POST
6. 将多个项目的一个权限分享给一个子账号,project/cooperateSim POST
7. 向项目的业主和负责人发送短信,project/sendMessage POST
8. 是否允许上传接口:project/isAllowUpload GET,返回：ok 允许上传，no_allow 不允许上传
9. 保存单元下的复合单元列表,element/elementGroup POST
    * **id参数名字参数projectId**
10. 添加要素数据,element/makeElementDataGroup POST
11. 用户邀请子账号,user/account/invite 应为POST
12. 解除水利云账号的绑定,user/account/unbind POST
13. 新建坐标线面,地图上绘图,field/createShape POST 
14. 编辑坐标线面,地图上绘图,field/editShape POST
15. 新建建築物，地图上绘图,field/createBuild POST
16. 编辑建筑物,地图上绘图,field/editBuild POST
17. 移动端外业保存,移动端上传外业勘测数据，field/saveField POST
18. 坐标文件上传，oss/coordinateFile POST 可以改到field/coordinateFile
19. 添加仪表，station/addSensor POST
20. 添加摄像头，station/addCamera POST
21. 编辑仪表，station/editSensor POST
22. 编辑摄像头，station/editCamera POST
23. 编辑测站，station/editStation POST
24. 上传河道模型和水位流量关系曲线, station/uploadModel POST
25. 编辑测站参数，station/editParameter POST
26. 由于项目文件前台是直接上传至阿里云的，因此前台在上传前，先判断用户下套餐对象状态，如果是过期，拒绝上传；后台关于文件允许上传接口还是需要判断过期
27. 是否允许上传接口:project/isAllowUpload GET,返回：ok 允许上传，no_allow 不允许上传
28. 千寻连接接口，qxwz/getAccount接口
29. 是否有千寻连接权限：qxwz/isAllowQxwz,GET
30. 是否有bim协同权限：project/isAllowBim,GET
    - **增加projectId** 

### 八. 套餐限制

每种套餐都有限制，分为空间数、项目数、子账号数、全景数、千寻连接数，这些连接时都要判断是否超过限制，后台标签使用切面方式控制；由于前台的文件是由浏览器直接上传到阿里云的，因此文件上传、下载、删除需要先访问后台接口，判断是否允许上传，上传完文件再访问后台接口，告诉后台上传了多大的文件，让后台记录空间数变化；删除文件后也要访问后台接口，让后台记录空间数变化,同时记录日志；空间数只是限制阿里云空间大小，但是阿里云收费除了空间大小外，还有流量费，为了防止用户恶意操作浪费流量，还要限制流量使用，上传和下载流量最大是空间大小的10倍。

>
1. 向后台报告上传文件信息，project/reportUploadFileInfo POST
    * 根据文件大小更改空间使用情况，根据文件名记录上传日志(本迭代暂不考虑)
    * 参数：
    * projectId:项目id
    * fileNames:上传的一批文件名，多个用‘,’隔开
    * fileSize:上传的一批文件总大小
    * alias:要素别名
    * 返回：OK,附加数据是{fileSize:文件大小}，将文件大小回传给前台，FAIL，参数错误
2. 向后台报告下载文件信息，project/reportDownloadFileInfo POST
    * 根据文件大小更改空间使用情况，根据文件名记录下载日志(本迭代暂不考虑)
    * 参数：
    * projectId,项目id
    * fileName,下载文件名，多个用‘,’隔开
    * fileSize，下载文件大小
    * alias:要素别名
    * 返回：OK,附加数据是{fileSize:文件大小}，将文件大小回传给前台，FAIL，参数错误 
3. 是否允许上传接口:project/isAllowUpload GET
    * 不仅要判断空间数是否超限，判断上传流量是否超限，还要判断套餐是否过期，只要符合以上任一条件，即返回no_allow
    * 参数：projectId
    * 返回：OK 允许上传
    * **EXPIRED 套餐过期**
    * **NO_ALLOW 空间已满或流量用尽**
4. 是否允许下载接口:project/isAllowDownload GET   
    * 判断上传流量是否超限
    * 参数： projectId
    * 返回：OK 允许下载
    * **NO_ALLOW 流量用尽**
5. 新建项目，project/new接口，加入判断项目数超限
6. 子账号数，user/invite接口，加入判断子账号数超限
7. 全景数，interest/savePanorama接口，加入判断全景数超限
8. 批量添加全景，interest/savePanoramas接口，加入判断全景数超限
9. 千寻连接接口，qxwz/getAccount接口，加入判断套餐是否过期，是否有千寻连接权限，判断连接数是否超限

### 九. 相关实体类

```
// 配置
public class Package extends BaseEntity{
    /** 套餐类型 */
    private CommonEnum.PackageType type;
    /** 到期时间 */
    private Date expireDate;
    /** 当前总空间使用情况 */
    private long curSpaceNum;
    /** 当前流量使用情况 */
    private long curTrafficNum;
    /** 唯一标识 */
    private String instanceId;
    // 目前项目数
    private long curProjectNum;
    // 目前子账号数
    private long curAccountNum;

    private User user;
}
// 订单
public class Trade extends BaseEntity{
    /** 订单号 */
    private String outTradeNo;
    /** 支付日期 */
    private Date payDate;
    /** 支付类型（微信、支付宝） */
    private PayType payType;
    /** 服务类型 */
    private Type type;
    /** 购买类型（首次，续费）*/
    private BuyType buyType;
    /** 订单状态 */
    private Status status;
    /** 订单价格(分) */
    private long price;
    /** 服务唯一编码 */
    private String instanceId;
    /** 三类商品type整合 */
    private BaseType baseType;
    /** 有效时间 */
    private int validTime;
    /** 备注 */
    private String remark;
    /** Goods下载地址 */
    private String downloadUrl;
    /** Goods个数 */
    private int goodsNum;
    /** 退款日期 */
    private Date refundDate;
    // 删除状态
    private boolean deleteStatus;

    private User user;

    // 配置类型
    public enum BaseType{
        TEST,
        EXPERIENCE,
        YOUTH,
        SUN,
        SUNRISE
    }

    public enum PayType{
        /** 微信 */
        WX,
        /** 支付宝 */
        ALI;
    }

    public enum BuyType{
        /** 首次购买 */
        BUY,
        /** 续费 */
        RENEW,
        /** 升级 */
        UPGRADE
    }

    public enum Status{
        /** 未支付 */
        NOPAY,
        /** 已支付 */
        PAY,
        /** 已关闭 */
        CLOSE,
        /** 已撤销 */
        REVERSE,
        /** 已退款 */
        REFUND
    }

    public enum Type{
        /** 套餐 */
        PACKAGE,
        /** 测站 */
        STATION,
        /** 数据服务 */
        GOODS
    }
}
// 流水
public class Turnover extends BaseEntity{
    /** 流水号 */
    private String turboverNo;
    /** 订单号 */
    private String outTradeNo;
    /** 支付状态 */
    private Type type;
    /** 价格 */
    private long price;
    /** 余额 */
    private long balance;
}
```

