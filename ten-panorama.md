# 全景接口设计
设计PanoramaController控制层，把以前在InterestController中关于全景的代码删除。所有接口起始名字都是/panorama。
请注意：4000(PARAMETER_ERROR参数错误)，表示服务器收到的参数不合法，这个错误每个接口都可能发生，就不每个接口写了。

![panorama](/uploads/3335ef51528f31f62bd7ac8f3187119e/panorama.png)

## 一 查看全景
>
1. 取得全景数据：panorama/panorama/{instanceId},GET
    * 返回全景和场景的所有数据 
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
3. 取得全景皮肤xml：panorama/tour.xml/skin.xml,GET
    * 参数：无
    * 返回：
        * xml在后台，读取出来，使用response写xml到页面
4. 取得用户/子账号自己的全景列表: panorama/lists
    * 取得自己的全景，需要通过用户id查询,或需要通过子账号id查询
    * 参数：无
    * 返回：
        * OK: 返回json数组，每个全景除去userId，accountId, hotspot，angleOfView, sceneGroup外，别的字段都传递 
6. 取得，在地图上显示的，所有全景列表: panorama/all/lists
    * 取得地图上显示的全景列表，包含以下三种情况，用户登录后，子账号登陆后，未登录 
    * 参数：无
    * 返回：
        * OK: 返回json数组，每个全景只传递，name，instanceId, thumbUrl, coor 
7. 取得，管理员取得所有待审核的共享类型全景列表: panorama/admin/lists
    * 取得管理员上显示的全景列表，只需返回共享的，待审核的全景列表
    * 参数：无
    * 返回：
        * OK: 返回json数组，每个全景只传递，name，instanceId, info, thumbUrl, coor, region

## 二 新建全景
>
1. 取得直传token: oss/directToken,GET
    * 这是OssController定义的，用于前台使用fileinput控件上传文件使用的直传授权，参见https://help.aliyun.com/document_detail/31926.html?spm=a2c4g.11186623.4.10.etc6Id， 具体算法下载java源码。还有两个网址可以参考： https://help.aliyun.com/knowledge_detail/39535.html  / https://help.aliyun.com/document_detail/31988.html?spm=a2c4g.11186623.2.3.HUAJpN#h2-post-policy2。  
2. 新建全景：panorama/add,POST
    * 新建全景过程，请查看设计文档流程图。使用userId/时间戳建立根据临时目录，图片从qqsl的bucket下载，路径在qqsl/panorama/{userid}/，所有图片下载完成后，执行krapno的切片命令，等待完成；使用时间戳进行MD5签名，作为全景的唯一编码，然后把切片后的图片，vtour/panos/下所有文件夹和图片循环上传至，qqslimage/panorama/{userid}/；每个图片都有一个切片目录，目录的名字是图片名(经过fileinput改名)，目录名作为图片的instanceId，每个目录下都有一个thumb.jpg文件，是缩略图。然后在数据库中添加全景表：
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
 sceneGroup: 场景顺序,空json，"{}"
```
    
```
循环在数据库中添加场景表：
fileName: 图片名
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
"images":[{"name":"001-西宁", "fileName":"1522811870947bik.jpg"},
{"name":"333-西安","fileName":"152281187095756l.jpg"}]}
```

## 三. 全景其他操作
>
1. 编辑全景: panorama/update,POST
    * 可以编辑name，info，isShare，coor，region，但是全景图片不能添加和删除了,可以把全景由共享改为非共享，反之亦然
    * 已审核的共享全景一旦编辑，状态改为待审核
    * 参数：id,全景id，name，info，isShare，coor，region
    * 返回：
        * OK:编辑成功
        * FAIL:全景不属于当前用户或子账号，不能编辑
2. 编辑全景热点等信息: panorama/updateHotspot,POST
    * 用户或子账号编辑全景热点，后台直接把热点json保存到数据库
    * 已审核的共享全景一旦编辑，状态改为待审核
    * 参数: id,全景id，angleOfView,起始视角json, hotspot,热点json,sceneGroup,场景顺序
    * 返回：
        * OK:编辑成功
        * FAIL:全景不属于当前用户或子账号，不能编辑
3. 删除全景：panorma/delete,POST
    * 用户和子账号均可删除全景，子账号可以删除新建24小时之内的全景
    * 参数: id,全景id
    * 返回：
        * OK:删除成功
        * FAIL: 子账号不能删除超过24小时的全景
4. 管理员全景审核通过：panorama/admin/review/success，POST
    * 审核通过时，不需要填写审核意见
    * 参数: id,全景id
    * 返回：
        * OK:审核完成
5. 管理员全景审核未通过：panorama/admin/review/fail，POST
    * 审核未通过时，需要填写审核意见
    * 参数: id,全景id,advice,审核意见
    * 返回：
        * OK:审核完成 
6. 下载场景原图: panorama/download, POST
    * 全景属于用户或子账号时，可以下载全景下各场景原图，由于场景原图保存在qqsl的私有bucket下，必须通过后台生成下载url，注意在新建时，在场景中保存originUrl，这样可由originUrl生成downloadUrl
    * 参数: id,全景id
    * 返回：
        * OK:数据是[{id:场景id，downloadUrl:xxx},{id:下一个场景id,downloadUrl:xxx}]
        * DATA_NOEXIST:全景或场景不存在
        * DATA_REFUSE:场景不属于当前用户或子账号


```
热点hotspot:
{
  "scene_75c116470f1a1e3b": 
    {"scene":[{"iconType":"custom","imgPath":"http://qqslimage.oss-cn-hangzhou.aliyuncs.com/1/media/img/a2b3e82dded8e3fd.png","thumbPath":"http://qqslimage.oss-cn-hangzhou.aliyuncs.com/1/media/img/a2b3e82dded8e3fd.png","isDynamic":"0","ath":"7.2739317319591","atv":"37.587015248987","name":"schp_fjcex3bwkd","linkedscene":"scene_590a999cd358fb6e","sceneTitle":"湟中维新渠化效果(1)","sceneImg":"http://qqslimage.oss-cn-hangzhou.aliyuncs.com/1/works/590a999cd358fb6e/thumb.jpg"}], "link":[],"image":[], "text":[{"iconType":"custom","imgPath":"http://qqslimage.oss-cn-hangzhou.aliyuncs.com/1/media/img/95b84432a515fec1.png","thumbPath":"http://qqslimage.oss-cn-hangzhou.aliyuncs.com/1/media/img/95b84432a515fec1.png","isDynamic":"0","ath":"21.609106152338","atv":"48.473176347164","name":"schp_jk2fwtstbi","hotspotTitle":"你好","wordContent":"你的应用加了身份认证，有人（或者你自己，呵呵）试图用manager用户登陆你的应用，密码输入错误5次或者5次以上（缺省是5次），就会在日志中记录警告信息，并锁定并禁止该用户的进一步登陆。以提醒你可能有人恶意猜测你的管理员密码。是tomcat为了阻止brute-force攻击（基于密码加密的暴力破解法）的安全策略。","isShowSpotName":""}], "voice":[],"imgtext":[],"obj":[]},
   "scene_590a999cd358fb6e":
    {"scene":[],"link":[],"image":[],"text":[],"voice":[],"imgtext":[],"obj":[]}
}

起始视角:
{"viewSettings":[
   {"sceneName":"scene_75c116470f1a1e3b","hlookat":"24.088790438777","vlookat":"56.670667874245","fov":"90","fovmin":"5","fovmax":"120","vlookatmin":"-90","vlookatmax":"90","keepView":"0"},
   {"sceneName":"scene_590a999cd358fb6e","hlookat":"14.186482857673","vlookat":"23.921350220188","fov":"90","fovmin":"5","fovmax":"120","vlookatmin":"-90","vlookatmax":"90","keepView":"0"}
]}

场景显示顺序
{"sceneGroups":[
  {"iconType":"system","imgPath":"/static/images/skin1/vr-btn-scene.png","groupName":"场景选择",
    "scenes":[
      {"sceneName":"scene_de1ec09cd94d47ba","viewuuid":"de1ec09cd94d47ba","imgPath":"http://p4sxhdbsp.bkt.clouddn.com/1/works/de1ec09cd94d47ba/thumb.jpg","sceneTitle":"666(1)"},
     {"sceneName":"scene_9ef9165de43a0a0d","viewuuid":"9ef9165de43a0a0d","imgPath":"http://p4sxhdbsp.bkt.clouddn.com/1/works/9ef9165de43a0a0d/thumb.jpg","sceneTitle":"湟中维新渠化效果(1)"}
    ]
  }
]}
```









