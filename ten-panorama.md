# 全景接口设计
设计PanoramaController控制层，把以前在InterestController中关于全景的代码删除。所有接口起始名字都是/panorama。
请注意：4000(PARAMETER_ERROR参数错误)，表示服务器收到的参数不合法，这个错误每个接口都可能发生，就不每个接口写了。

![panorama-class](/uploads/e56230899240ea156d6ebe97029b6cab/panorama-class.jpg)

## 查看全景
>
1. 取得全景数据：panorama/{instanceId},GET
    * 参数：instanceId
    * 返回：
        - OK：取得全景数据，data包含全景中所有属性
        - DATA_NOEXIST：全景不存在，找不到instanceId
        - FAIL：取得全景数据出错
2. 取得全景加载xml：panorama/tour.xml,GET
    * 参数：instanceId
    * 返回：
        - 正常情况，使用response写xml到页面
        - 全景不存在，使用response写xml到页面，但xml中节点下标识错误(按照前后台错误码)
        - 全景下没有场景，xml中节点下标识错误(按照前后台错误码)：
```
<krpano version="1.19">
    <error>4021，全景不存在</error>
    <error>4101，全景下没有场景</error>
</krpano>        
```
3. 取得全景皮肤xml：panorama/vtourskin.xml,GET
    * 参数：无
    * 返回：
        - xml在后台，读取出来，使用response写xml到页面

## 新建全景
>
1. 取得直传token: oss/directToken,GET
    这是OssController定义的，用于前台使用fileinput控件上传文件使用的直传授权，参见https://help.aliyun.com/document_detail/31926.html?spm=a2c4g.11186623.4.10.etc6Id， 具体算法下载java源码。还有两个网址可以参考： https://help.aliyun.com/knowledge_detail/39535.html  / 
 https://help.aliyun.com/document_detail/31988.html?spm=a2c4g.11186623.2.3.HUAJpN#h2-post-policy2。  
2. 新建全景：panorama/add,POST
    新建全景过程，请查看设计文档流程图。使用userId/时间戳建立根据临时目录，图片从qqsl的bucket下载，路径在qqsl/panorama/{userid}/{时间戳}，所有图片下载完成后，执行krapno的切片命令，等待完成；使用时间戳进行MD5签名，作为全景的唯一编码，然后把切片后的图片，vtour/panos/下所有文件夹和图片循环上传至，qqslimage/panorama/{userid}/；每个图片都有一个切片目录，目录的名字是图片名(经过fileinput改名)，目录名作为图片的instanceId，每个目录下都有一个thumb.jpg文件，是缩略图。
    然后在数据库中添加全景表：
```
 name: 全景名称
 info: 全景描述
 instanceId: 唯一编码,使用生产的唯一编码
 thumbUrl: 缩略图,qqslimage/panorama/{userid}/{第一个图片的唯一编码}/thumb.jpg
 isShare: 布尔值，是否共享
 coor: 坐标，从地图上读取，使用“102.323,36.454,1233.23”的格式
 region: 行政区，从地图上读取的坐标转换而来    
 status: 审核状态，未审核
 advice: 审核意见,空
 reviewDate: 审核时间,空
 hotspot: 热点信息,空json，"{}"
 angleOfView: 起始视角,空json，"{}"
```
    循环在数据库中添加场景表：
```
name: 图片名
originUrl: qqsl上的原图地址
thumbUrl: qqslImage上的图片缩略图
instanceId: 唯一编码，用于生产场景名
```
    * 参数：
        - name: 全景名称
        - info: 全景描述
        - isShare: 布尔值，是否共享
        - coor: 坐标，从地图上读取，使用“102.323,36.454,1233.23”的格式
        - region: 行政区，从地图上读取的坐标转换而来
        - images: [{name:图片名，originUrl: qqsl上阿里云上的原地址},...]
    * 返回
        - OK: 新建成功
        - PANORAMA_IMAGE_NOT_EXIST: 下载图片失败
        - PANORAMA_SLICE_ERROE: 切图失败
        - FAIL：场景新建失败 
        
```
    // 这是js已经调通的生成directToken的代码
    oss_token.OSSAccessKeyId = "XXX";
    oss_token.OSSAccessKey = "XXX";
    var policyTxt = {
        "expiration": "2020-01-01T12:00:00.000Z",
        "conditions": [
            {"bucket": "qqsl" },
            ["content-length-range", 0, 1048576000],
            ["starts-with", "$key", "panorama/"]
        ]
    };
    // 返回给前台的
    oss_token.policy = Base64.encode(JSON.stringify(policyTxt));
    var signatureTxt = Crypto.HMAC(Crypto.SHA1, oss_token.policy, oss_token.OSSAccessKey, { asBytes: true }) ;
    oss_token.signature = Crypto.util.bytesToBase64(signatureTxt);
    oss_token.host = "oss-cn-hangzhou.aliyuncs.com";
    oss_token.prefix = 'panorama/';
```


```
前台新建全景时传递的参数示例：
{"name":"全景名称1111111111","info":"全景描述2222222222222222","coor":"103.77645101765913,36.05377593481913,0","isShare":"true","region":"中国甘肃省兰州市七里河区兰工坪南街190号 邮政编码: 730050",
"images":[{"originUrl":"http://qqsl.oss-cn-hangzhou.aliyuncs.com/panorama/26/1522811870947bik.jpg","name":"001-11.jpg"},
{"originUrl":"http://qqsl.oss-cn-hangzhou.aliyuncs.com/panorama/26/152281187095756l.jpg","name":"333--11.jpg"}]}
```








