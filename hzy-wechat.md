# 河长云微信公众号接口设计

## 一 HzbUserController,河长办用户控制层
1. 河长办用户登录,/hzbUser/login,POST
    * 登录成功，除了把hzbUser保存在session中，还要保存regionCode
    * 参数: 
        * phone：手机号
        * password:密码
        * cookie：cookie
    * 返回：
        * OK:登录成功,返回河长办用户属性+河长办属性，{name:xxx,...,hzb:{name：xxx,regionCode:xxx,...}}
        * 4040 CODE_NEED 需要验证码
        * 4023 DATA_LOCK 用户已锁定
        * 4021：DATA_NOEXIST 账号不存在
        * FAIL：密码错误
2. 河长办用户验证码登录：/hzbUser/loginByVerify POST
    * 参数：password:密码；verification:验证码
    * 返回：
        * OK(用户对象):登录成功
        * 4041: CODE_INVALID 验证码过期
        * 4042: CODE_ERROR 验证码输入错误
3. 登录发送验证码: /hzbUser/login/getLoginVerify
   * 参数
       * phone:手机号
   * 返回：
       * OK:发送成功
       * 4021：DATA_NOEXIST 账号不存在
4. 修改密码:在基本资料的修改密码处点击保存时调用， /hzbUser/updatePassword POST
   * 参数：
       * oldPassword:当前密码
       * newPassword：新密码
   * 返回：
       * OK:修改成功
       * FAIL:当前密码错误
5. 获取当前河长办用户：/hzbUser/getUser GET
       * 参数：
       * 返回：
           * OK(用户对象)，包含河长办用户所有属性，不包含河长办属性
           * 4011: NO_SESSION,未登录
6. 河长办用户注销,/hzbUser/logout POST
   * 参数：无
   * 返回：OK,注销成功
       
## 二 微信用户登录,/wechat/login,POST
   * 微信登录，把unionId存入session中
   * 参数：
       *  unionId：微信unionId      
   * 返回
       * OK，登录成功
   
## 三 ArticleControler 新闻动态和政策方案 
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
4. 新建文章，/article/create, POST
    * 河长办用户操作，参数中不需要regionCode，因为session中有这个信息  
    * 参数： 
        * title,文章标题
        * content，文章内容
        * imageUrl，图片
        * type，WATERPOLICY(水利法规), WATERNEW(新闻动态)
        * level，CITY(州级),COUNTY(县级)
    * 返回:
        * OK,保存成功
5. 修改文章，/article/update, POST
    * 参数： 
        * id,新闻动态id
        * title,文章标题        
        * content，文章内容
        * imageUrl，图片
        * type，WATERPOLICY(水利法规), WATERNEW(新闻动态)
        * level，CITY(州级),COUNTY(县级)
    * 返回:
        * OK,保存成功
 6. 上传文章图片，/article/image/create POST
    * 前台编辑器上传图片时调用后台接口，把图片上传到阿里云的qqslimage/hzy/{regionCode}/article/中，上传时使用https://www.cnblogs.com/jdonson/archive/2009/07/22/1528466.html方式生成图片的唯一编码
    * 上传时对图片进行压缩，以便用户能快速浏览，参见https://blog.csdn.net/niuch1029291561/article/details/17377903,压缩到图片宽度600px
    * 参数： 
    * 使用HttpServletRequest request，转换为MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request，取得图片，参见https://blog.csdn.net/qq_24419601/article/details/79784773
    * 返回:
        * OK,数据：{"fileName":"文件名.文件格式","url":"上传成功后得资源路径url"} ,资源路径url是阿里云的路径
        * FAIL
                        
## 四 StationController 测站
1. 取得测站列表：/station/lists
    * 参数：regionCode，行政区编码
    * 返回：
        * OK,行政区下的所有测站，包含属性，测站下仪表所有属性，返回和水利云返回测站列表数据保持一致
2. 取得仪表当日数据：121.40.82.11:8080/sensor/{code}, GET    
    * 参数：code，仪表编码
    * 返回：
        * OK，仪表当日数据
        * FAIL,仪表编码不存在

## 五 PanoramaController，全景
1. 取得全景列表，/panorama/lists,GET
    * 参数：regionCode，行政区编码
    * 返回：
        * OK，行政区下的全景列表,包含name，createDate(建立时间),instanceId(唯一编码),thumbUrl(缩略图)，coor(坐标)，address(位置)

## 六 ComplaintControler，投诉
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
2. 取得微信用户的投诉列表，/complaint/lists,GET
    * 参数：unionId，微信唯一标识
    * 返回：
        * OK，返回投诉列表，只需返回id，createDate, title，imageUrl，riverSegmentName, handleName, handleDate
3. 取得河长办下的投诉列表:/complaint/hzb/lists,GET
    * 参数：
        * regionCode：行政区全国统一编码    
    * 返回：
        * OK，返回投诉列表，只需返回id，createDate, title，imageUrl，riverSegmentName, handleName, handleDate
4. 取得投诉详情，/complaint/complaint,GET
    投诉实体中增加imageUrl，保存第一张图的地址
    * 参数：
        * instanceId:投诉唯一标识
        * unionId，微信唯一标识
    * 返回：
        * OK，投诉所有属性
        * FAIL，唯一标识不存在
        * 4022，DATA_REFUSE，请求的投诉不属于自己
5. 回复投诉反馈，/complaint/handle,POST
    * 河长办用户操作,回复反馈时可以上传图片，直接存储在阿里云上地址：qqslimage/hzy/{regionCode}/complaint/{unionId}/{instanceId}/{handle}
    * 如果投诉的regionCode和session中保存的regionCode不一致，返回错误
    * 参数：
        * id,投诉id
        * handleName,处理人姓名
        * handleContent,回复内容
    * 返回：
        * OK,回复成功        
        * 4022: DATA_REFUSE 投诉不属于河长办 
6. 编辑反馈回复，/complaint/handle/update,POST
    * 如果投诉的regionCode和session中保存的regionCode不一致，返回错误
    * 参数：
        * id,投诉id
        * handleName,处理人姓名
        * handleContent,回复内容
    * 返回：
        * OK,编辑成功        
        * 4022: DATA_REFUSE 投诉不属于河长办  
7. 回复反馈时，上传图片，/complaint/image/create,POST
   * 前台编辑器上传图片时调用后台接口，把图片上传到阿里云的qqslimage/hzy/{regionCode}/complaint/handle/，投诉处理图片中，上传时使用https://www.cnblogs.com/jdonson/archive/2009/07/22/1528466.html方式生成图片的唯一编码
    * 上传时对图片进行压缩，以便用户能快速浏览，参见https://blog.csdn.net/niuch1029291561/article/details/17377903,压缩到图片宽度600px
    * 参数： 
    * 使用HttpServletRequest request，转换为MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request，取得图片，参见https://blog.csdn.net/qq_24419601/article/details/79784773
    * 返回:
        * OK,数据：{"fileName":"文件名.文件格式","url":"上传成功后得资源路径url"} ,资源路径url是阿里云的路径
        * FAIL

## 七 HzUsrController 河长
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
## 八 riverSegmentController，河段接口
1. 取得河段列表，riverSegment/region/lists, GET
    * 参数：
        * regionCode：行政区全国统一编码
    * 返回：
        * OK，行政区下的所有河段，[{id,name},{..}]，如果行政区下没有河道，返回流过市州的所有河流id和name

 ## 九 WeChatController,微信控制层
 1. 根据code取得unionId,/weChat/unionId,GET
    * 取得unionId成功后，将unionId存在session中，直接登录
    * 参数：code，微信code
    * 返回：
        * OK，{unionId:xxx}，目前没有unionId时，返回openId
        
## 十 OssController,阿里云Oss控制层
1. 根据treePath(阿里云路径)得到文件列表，/oss/objectFiles,GET
    * 参数:
        * dir: 文件路径
        * bucket: bucket名
    * 返回：
        * OK，文件数组，ObjectFile对象全属性
2. 根据key得到文件访问路径，/oss/objectUrl，GET
    * 参数:
        * key:文件key
        * bucket:bucket名
    * 返回：
        * OK，{url:xxx}
3. 获取sts安全令牌,用于上传或获取oss存储的文件, /oss/sts,GET
    * 参数：无
    * 返回：
        * OK，数据格式：
```
{requestId:xxx, AssumedRoleUser:            \"AssumedRoleId\":" + "\""
				+ AssumedRoleId + "\"" + ",\n"
				+ "                     \"Arn\":" + "\""
				+ CommonAttributes.ROLE_ARN + "\"" + "\n"
				+ "                                        },\n"
				+ "                     \"Credentials\":{\n"
				+ "                     \"AccessKeyId\":" + "\"" + AccessKey_Id
				+ "\"" + ",\n" + "                     \"AccessKeySecret\":"
				+ "\"" + AccessKey_Secret + "\"" + ",\n"
				+ "                     \"Expiration\":" + "\"" + Expiration
				+ "\"" + ",\n" + "                     \"SecurityToken\":"
				+ "\"" + Security_Token + "\"" + "\n"
				+ "                                      }\n" + "}\n";
```               

## 十一 RiverController,河湖控制层
1. 取得河湖列表,/river/lists,GET   
    * 参数
        * regionCode，行政区code
    * 返回：
        * OK,返回河流的所有属性
        
                  