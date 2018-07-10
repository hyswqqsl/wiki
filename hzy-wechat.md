# 河长云web端接口设计

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

## 二 MatterController，事件管理

    * 前台事件图片上传
    * 登记图片: /qqsl/hzy/{regionCode}/matter/{code}/register
    * 承办图片: /qqsl/hzy/{regionCode}/matter/{code}/{handleId}/handle
    * 办结图片: /qqsl/hzy/{regionCode}/matter/{code}/{handleId}/complete
    * 事件状态：0-登记 1-交办 2-责任退回 3-承办 4-重办 5-办结 6-审核退回 7-归档
     
1. 事件登记,/matter/register,POST
   * 督查督办编号目前由唯一编码方式生成
   * 目前事件管理都是由市级河长办发起的，交办给县级河长办
   * 生成的事件状态是登记
   * 记录登记人员id
   * 事件来源：0-上级交办 1-系统推送 2-社会媒体监督 3-群众投诉 4-河长报告
   * 参数：
       * code: 督查督办编号
       * title: 标题
       * content: 内容
       * source: 事件来源
       * sourceId: 来源id,如果来源是河长报告，那么来源id就是河长报告id,如果来源是投诉，那么来源id就是投诉id
   * 返回：
       * OK，建立成功
2. 事件交办,/matter/deliver,POST
    * 事件状态必须是登记或责任退回才能交办
    * 事件改为状态是交办
    * 紧急程度：0-一般 1-加急 2-紧急            
    * 参数：
        * id: 事件id
        * hzbId: 交办给的县级河长办id
        * requirement: 办理要求
        * deadline: 时限要求
        * emergency: 紧急程度        
      * 返回：
        * OK: 交办成功
        * 4111: MATTER_TYPE_ERROR 事件状态错误
3. 事件承办,/matter/handle,POST
   * 事件是交办或审核退回的，可以进行承办
   * 事件改为状态是承办
   * 承办时可以上传图片，由前台处理
   * 参数： 
       * id: 事件id
       * organizerId: 承办单位id
       * description: 承办描述
    * 返回： 
       * OK: 承办成功
       * 4111: MATTER_TYPE_ERROR 事件状态错误
       * 4022: DATA_REFUSE，事件不属于该河长办
4. 事件办结,/matter/complete,POST
    * 事件状态改为办结
    * 办结时可以上传图片，由前台处理
    * 参数：
        * id：事件id
        * description: 办结描述
        * 4111: MATTER_TYPE_ERROR 事件状态错误
        * 4022: DATA_REFUSE，事件不属于该河长办        
5. 事件审核不通过，审核退回,/matter/returnBack,POST
    * 事件状态改为审核退回
    * 审核时可以上传图片，由前台处理
    * 参数：
        * id： 事件id
        * description: 审核描述
    * 返回:
        * OK: 退回成功       
        * 4111: MATTER_TYPE_ERROR 事件状态错误
6. 事件归档,/matter/archive,POST
    * 事件状态改为归档
    * 参数：
        * id： 事件id
        * description: 审核描述 
    * 返回：
        * OK: 归档成功
        * 4111: MATTER_TYPE_ERROR 事件状态错误
7. 事件催办,/matter/urgent,POST
    * 市河长办发起
    * 事件状态必须是承办，才能接受催办
    * 参数：
        * id：事件id
        * content: 催办内容
    * 返回：
        * OK：催办成功
        * 4111: MATTER_TYPE_ERROR 事件状态错误
8. 事件催办回复,/matter/urgent/result,POST
    * 县区河长办分回复
    * 参数：
        * id: 事件催办id
        * result: 回复内容
    * 返回：
        * OK：回复成功
        * 4022: DATA_REFUSE，事件催办不属于该河长办        
9. 事件反馈,/matter/feedback,POST
    * 县区河长办发起
    * 事件状态必须是承办，才能反馈
    * 参数：
        * id：事件id
        * content: 反馈内容
    * 返回：
        * OK：反馈成功
        * 4111: MATTER_TYPE_ERROR 事件状态错误
10. 事件反馈回复：/matter/feedback/result,POST
    * 市河长办回复
    * 参数：
        * id: 事件催办id
        * result: 回复内容
    * 返回：
        * OK：回复成功
        * 4010: UNAUTHORIZED，非市级河长办不能进行操作
9. 事件责任退回,/matter/deliver/returnBack,POST
   * 事件状态必须是交办
   * 事件必须在24小时内
   * 参数：
       * id: 事件id
       * description：责任退回描述
   * 返回：
        * OK：反馈成功
        * 4111: MATTER_TYPE_ERROR 事件状态错误
        * 4112: MATTER_TIMEOUT 交办超时，不能责任退回
10. 取得承办单位列表，/matter/organizer/lists，GET
   * 参数：无
   * 返回:
       * OK，承办单位所有属性，【{id：xxx,code:xxx, ...},{id:xxx,code:xxx,...}】
       * FAIL, 行政区编码和河长办不一致
11. 查看事件详情,/matter/{code},GET
    * 列出事件所有数据
    * 参数：code，督查督办编号
    * 返回：
        * 事件所有属性，事件图片list
        * 事件处理列表，每个列表中的承办文件list，办结文件list
        * 事件下发,上报列表
~~~
{code:xxx,title:xxx,...,
    files: [path1,path2,...],
    handles: [
        {id:xxx,handleDescription:xxx,organizerId:xxx, handleFiles: [path1,path2,...], 
            completeImages: [path1,path2,...]
        },
        {id:xxx,handleDescription:xxx,organizerId:xxx, handleFiles: [path1,path2,...], 
            completeFiles: [path1,path2,...]
        }
    ], 
    hastens: [
        {content:xxx,result:xxx,type:xxx},
        {...}
    ], 
    feedbacks: [
        {content:xxx,result:xxx,type:xxx},
        {...}
    ]
}
~~~

## 三 ArticleControler 新闻动态和政策方案 
1. 取得新闻动态列表,/article/newses,GET
    * 参数：
        * regionCode：行政区编码
    * 返回：
        * OK，所属行政区的新闻动态列表
        * FAIL，行政区不存在
2. 取得政策法规列表，/article/laws, GET
    * 参数：
       * regionCode：行政区编码
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
    * 前台编辑器上传图片时调用后台接口，把图片上传到阿里云的qqslimage/hzy/{regionCode}/article/中，上传时使用 https://www.cnblogs.com/jdonson/archive/2009/07/22/1528466.html 方式生成图片的唯一编码
    * 上传时对图片进行压缩，以便用户能快速浏览，参见https://blog.csdn.net/niuch1029291561/article/details/17377903 ,压缩到图片宽度600px
    * 参数： 
    * 使用HttpServletRequest request，转换为MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request，取得图片，参见https://blog.csdn.net/qq_24419601/article/details/79784773
    * 返回:
        * OK,数据：{"fileName":"文件名.文件格式","url":"上传成功后得资源路径url"} ,资源路径url是阿里云的路径
        * FAIL
                        
## 四 StationController 测站
1. 取得测站列表：/station/lists
    * 参数：
       * regionCode：行政区编码
    * 返回：
        * OK,行政区下的所有测站，包含属性，测站下仪表所有属性，返回和水利云返回测站列表数据保持一致
2. 取得仪表当日数据：121.40.82.11:8080/sensor/{code}, GET    
    * 参数：code，仪表编码，token
    * 返回：
        * OK，仪表当日数据
        * FAIL,仪表编码不存在

## 五 PanoramaController，全景
1. 取得全景列表，/panorama/lists,GET
    * 参数：
       * regionCode：行政区编码
    * 返回：
        * OK，行政区下的全景列表,包含name，createDate(建立时间),instanceId(唯一编码),thumbUrl(缩略图)，coor(坐标)，address(位置)

## 六 ComplaintControler，投诉
1. 公众号提交投诉,/complaint/wechat/submit,POST
    * 投诉实体中增加imageUrl，上传图片时，把第一张图的地址保存在imageUrl中
    * 不需要regionCode
    * 参数：
        * title：标题，必需
        * description：描述，由几组关注信息自动组成的，必需
        * content：投诉内容，必需
        * coor:由网页定位识别的经纬度，必需
        * address:由网页定位识别的位置，必需
        * riverSegmentId:选择的河段id，必需
        * riverSegmentName:选择的河段名，必需
        * name:投诉人姓名，可以使匿名
        * @ phone:投诉人电话
        * instanceId:唯一标识
        * mediaIds: "xx,xx,.."
        * unionId，微信唯一标识
    * 返回：
        * OK，建立成功，与登录的user关联
2. 手机app提交投诉,/complaint/phone/submit,POST
    * 不需要regionCode
    * 参数：
        * title：标题，必需
        * description：描述，由几组关注信息自动组成的，必需
        * content：投诉内容，必需
        * coor:由网页定位识别的经纬度，必需
        * address:由网页定位识别的位置，必需
        * riverSegmentId:选择的河段id，必需
        * riverSegmentName:选择的河段名，必需
        * name:投诉人姓名，可以使匿名
        * @ phone:投诉人电话
        * instanceId:唯一标识
        * imageUrl:封面图片
        * unionId，微信唯一标识
    * 返回：
        * OK，建立成功，与登录的user关联        
3. 公众号取得微信用户的投诉列表，/complaint/lists,GET
    * 参数：
        * unionId，微信唯一标识
    * 返回：
        * OK，返回投诉列表，只需返回id，createDate, title，imageUrl，riverSegmentName, status,handleName, handleDate
4. 取得河长办下的投诉列表:/complaint/hzb/lists,GET
    * 参数：无
    * 返回：
        * OK，返回投诉列表，只需返回id，createDate, name, phone, title，imageUrl，riverSegmentName, status, handleName, handleDate
5. 取得投诉详情，/complaint/complaint,GET
    * 投诉实体中增加imageUrl，保存第一张图的地址
    * 参数：
        * instanceId:投诉唯一标识
        * unionId，微信唯一标识
    * 返回：
        * OK，投诉所有属性
        * FAIL，唯一标识不存在
        * 4022，DATA_REFUSE，请求的投诉不属于自己
6. 回复投诉反馈，/complaint/handle,POST
    * 如果投诉的regionCode和session中保存的regionCode不一致，返回错误
    * 参数：
        * id,投诉id
        * handleName,处理人姓名
        * handleContent,回复内容
    * 返回：
        * OK,回复成功        
        * 4022: DATA_REFUSE 投诉不属于河长办 
7. 编辑反馈回复，/complaint/handle/update,POST
    * 如果投诉的regionCode和session中保存的regionCode不一致，返回错误
    * 参数：
        * id,投诉id
        * handleName,处理人姓名
        * handleContent,回复内容
    * 返回：
        * OK,编辑成功        
        * 4022: DATA_REFUSE 投诉不属于河长办  
8. 编辑器上传回复图片，/complaint/image/create,POST
   * 前台编辑器上传图片时调用后台接口，把图片上传到阿里云的qqslimage/hzy/{regionCode}/complaint/handle/，投诉处理图片中，上传时使用https://www.cnblogs.com/jdonson/archive/2009/07/22/1528466.html 方式生成图片的唯一编码
    * 上传时对图片进行压缩，以便用户能快速浏览，参见https://blog.csdn.net/niuch1029291561/article/details/17377903 ,压缩到图片宽度600px
    * 参数： 
    * 使用HttpServletRequest request，转换为MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request，取得图片，参见 https://blog.csdn.net/qq_24419601/article/details/79784773
    * 返回:
        * OK,数据：{"fileName":"文件名.文件格式","url":"上传成功后得资源路径url"} ,资源路径url是阿里云的路径
        * FAIL

## 七 HzUsrController 河长
1. 取得河长名录，/hzUser/lists,GET
    * 参数：
        * regionCode：行政区编码    
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
1. 取得河段列表，/riverSegment/region/lists, GET
    * 参数：
        * regionCode：行政区编码, 这个regionCode是当先位置的县区级行政编码    
    * 返回：
        * OK，行政区下的所有河段，[{id,name},{..}]，如果行政区下没有河道，返回流过市州的所有河流id和name

## 九 WeChatController,微信控制层
 1. 根据code取得unionId,/weChat/unionId,GET
    * 取得unionId成功后，将unionId存在session中，直接登录
    * 参数：
        * code，微信code
        * regionCode:行政区编码 
    * 返回：
        * OK，{unionId:xxx}，目前没有unionId时，返回openId
2. 移动端微信用户登录,/weChat/login,POST
   * 微信登录，把unionId存入session中
   * 参数：
       * unionId：微信unionId
       * regionCode:行政区编码       
   * 返回
       * OK，登录成功        
        
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
    * 参数: 
       * regionCode:行政区编码     
    * 返回：
        * OK,返回河流的所有属性
        
    