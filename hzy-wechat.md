# 河长云微信公众号接口设计
## 一 ArticleControler 新闻动态和政策方案 
1. 取得新闻动态列表,/article/newses,GET
    * 参数：regionCode，行政区编码
    * 返回：
        * OK，所属行政区的新闻动态列表
        * FAIL，行政区不存在
2. 取得政策法规列表，/article/laws, GET
    * 参数：regionCode，行政区编码
    * 返回：
        * OK，全国范围的政策法规+市级的政策法规列表  
        * FAIL，行政区id不存在
3. 取得新闻动态和政策法规详情,/article/article/{id},GET
    * 参数：id: 新闻动态id
    * 返回：
        * OK，新闻动态所有属性
        * FAIL，id不存在
4. 保存新闻动态，/article/create, POST
    * region不用保存，因为session中有这个信息                 
    * 参数： 
        * title,文章标题
        * content，文章内容
        * imageUrl，图片
        * type，WATERPOLICY(水利法规), WATERNEW(新闻动态)
        * level，CITY(州级),COUNTY(县级)
    * 返回:
        * OK,保存成功
        
## 二 StationController 测站
1. 取得测站列表：121.40.82.11:8080/stations，GET
    * 参数：regionCode，行政区编码
    * 返回：
        * OK,行政区下的所有测站，包含createDate(建立时间),instanceId(唯一编码),type(测站类型),address(安装位置),value(最新数据)
2. 取得仪表当日数据：121.40.82.11:8080/sensor/{code}, GET    
    * 参数：code，仪表编码
    * 返回：
        * OK，仪表当日数据
        * FAIL,仪表编码不存在
3. **取得仪表列表：www.qingqingshuili.cn:8080/sensors，GET **
    * 参数：token
    * 返回：OK(仪表列表)

## 三 PanoramaController，全景
1. 取得全景列表，/panorama/lists,GET
    * 参数：regionCode，行政区编码
    * 返回：
        * OK，行政区下的全景列表,包含name，createDate(建立时间),instanceId(唯一编码),thumbUrl(缩略图)，coor(坐标)，address(位置)

## 四 ComplaintControler，投诉
1. 提交投诉,/complaint/submit,POST
    投诉实体中增加imageUrl，上传图片时，把第一张图的地址保存在imageUrl中
    * 参数：
        * title：标题，必需
        * description：描述，由几组关注信息自动组成的，必需
        * content：投诉内容，必需
        * coor:由网页定位识别的经纬度，必需
        * address:由网页定位识别的位置，必需
        * riverSegmentId:选择的河段id，必需
        * riverSegmentName:选择的河段名，必需
        * name:投诉人姓名，可以使匿名
        * phone:投诉人电话，可以为空
        * instanceId:唯一标识
        * mediaIds: "xx,xx,.."
        * regionCode：行政区全国统一编码
        * unionId，微信唯一标识
    * 返回：
        * OK，建立成功，与登录的user关联
2. 取得投诉列表，/complaint/lists,GET
    * 参数：unionId，微信唯一标识
    * 返回：
        * OK，返回投诉列表，只需返回id，createDate, title，imageUrl，riverSegmentName, handleName, handleDate
3. 取得投诉详情，/complaint/complaint,GET
    投诉实体中增加imageUrl，保存第一张图的地址
    * 参数：
        * instanceId:投诉唯一标识
        * unionId，微信唯一标识
    * 返回：
        * OK，投诉所有属性
        * FAIL，唯一标识不存在
        * 4022，DATA_REFUSE，请求的投诉不属于自己
4. 回复投诉反馈，/complaint/handle,POST
    * 回复反馈时可以上传图片，直接存储在阿里云上地址：qqslimage/hzy/{regionCode}/complaint/{unionId}/{instanceId}/{handle}
    * 如果投诉的regionCode和session中保存的regionCode不一致，返回错误
    * 参数：
        * id,投诉id
        * handleName,处理人姓名
        * handleContent,回复内容
    * 返回：
        * OK,回复成功        
        * 4022: DATA_REFUSE 投诉不属于河长办 

## 五 HzUsrController 河长
1. 取得河长名录，/hzUser/lists,GET
    * 参数：
        * regionCode，行政区编码
    * 返回： 
        * OK,

```
{
    city(市级):{
        master(总河长):[{name,job(职务)},{...}],
        slave(副总河长):[{name,job(职务)},{...}]，
        normal(责任河长):[{name,job(职务),riverSegments(管辖河段):[xx,xx,xx]
    },
    county(县级):{
        master(总河长):[{name,job(职务)},{...}],
        slave(副总河长):[{name,job(职务)},{...}]，
        normal(责任河长):[{name,job(职务),riverSegments(管辖河段):[xx,xx,xx]
    },      
    town(乡级):{
        master(总河长):[{name,job(职务)},{...}],
        slave(副总河长):[{name,job(职务)},{...}]，
        normal(责任河长):[{name,job(职务),riverSegments(管辖河段):[xx,xx,xx]
    },           
    village(乡级):{
        master(总河长):[{name,job(职务)},{...}],
        slave(副总河长):[{name,job(职务)},{...}]，
        normal(责任河长):[{name,job(职务),riverSegments(管辖河段):[xx,xx,xx]
    }           
}
```


## 六 riverSegmentController，河段接口
1. 取得河段列表，riverSegment/region/lists, GET
    * 参数：
        * regionCode：行政区全国统一编码
    * 返回：
        * OK，行政区下的所有河段，[{id,name},{..}]，如果行政区下没有河道，返回流过市州的所有河流id和name

 ## 七 WeChatController,微信控制层
 1. 根据code取得unionId,/weChat/unionId,GET
    * 参数：code，微信code
    * 返回：
        * OK，{unionId:xxx}，目前没有unionId时，返回openId
        
## 八 OssController,阿里云Oss控制层
1. 根据treePath(阿里云路径)得到文件列表，/oss/objectFiles,GET
    * 参数:
        * dir: 文件路径
        * bucket: bucket名
    * 返回：
        * OK，文件数组，ObjectFile对象全属性