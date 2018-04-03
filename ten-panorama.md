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
1. 新建全景：panorama/add,POST
    新建全景过程，请查看设计文档流程图。使用userId/时间戳建立根据临时目录，图片从qqsl的bucket下载，路径在qqsl/panorama/{userid}/{时间戳}，所有图片下载完成后，执行krapno的切片命令，等待完成；使用时间戳进行MD5签名，作为全景的唯一编码，然后把切片后的图片，vtour/panos/下所有文件夹和图片循环上传至，qqslimage/panorama/{userid}/{唯一编码}；每个图片都有一个切片目录，目录的名字是图片名(经过fileinput改名)，目录名作为图片的instanceId，每个目录下都有一个thumb.jpg文件，是缩略图。
    然后在数据库中添加全景表：
```
 name: 全景名称
 info: 全景描述
 instanceId: 唯一编码,使用生产的唯一编码
 thumbUrl: 缩略图,qqslimage/panorama/{userid}/{场景唯一编码}/{第一个图片的唯一编码}/thumb.jpg
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
        

