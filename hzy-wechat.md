# 河长云接口设计

![Main](/uploads/c9320c2bf210134518be2ec906eef08c/Main.png)

![class](/uploads/7df65ee2bce66d8926b1083f07439114/class.png)



![22](/uploads/8d75fb2464b5cb4dbdb5aa26a6a0533d/22.png)

## 一 河长登录接口
1. 登录发送验证码: /hzUser/login/getLoginVerify,GET
    * **app端使用**
    * **角色：无**
    * 河长登录时，取得验证码
    * 参数：
        * phone:登录手机号
    * 返回：
        * OK:发送成功
        * DATA_NOEXIST:手机号不存在
2. 登录：/hzUser/login,POST
    * **app端使用**
    * **角色：无**
    * 河长每次登录都是用动态验证码    
    * 参数：
        * code:验证码
    * 返回:
        * OK:登录成功,返回河长详情，管辖的河段，上级河长，下级河长信息，河长办 {name:xx,regionId,regionName,phone:xx,job:xx,type:xx,riverSegments:[{id,name:xx,length:xx,beginStation:xx,endStation:xx,riverName:xx,regionId,regionName:xx},{...}],upHzUsers:[{id,name:xx,regionId,regionName:xx.phone:xx,job:xx,type:xx},{...}], downHzUsers:[{id,name:xx,regionId,regionName:xx,phone:xx,job:xx,type:xx},{...}], hzb:{name,regionId,regionName,remark,phone},upHzb:{name,regionId,regionName,remark,phone}}
        * 4041: CODE_INVALID 验证码过期
        * 4042: CODE_ERROR 验证码输入错误
3. 取得河长名录，/hzUser/lists,GET
    * **weChat,web端使用**
    * **角色：无**
    * 参数：
        * regionCode：行政区编码    
    * 返回： 
        * OK，格式:
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
4. 村级河长直接发报告给乡级河长办,/hzUser/village/report,POST
      * 图片单独上传
      * 村级报告type=0
      * **app端使用**
      * **角色：村级河长**
      * 村级报告给乡级河长办，直接报告
      * 报告中加入上级河长id，乡河长办id，就是说村河长报告不仅乡级河长能看到，乡河长办也能看到
      * 参数：
          * title,标题
          * content,内容
          * instanceId,唯一编码
          * riverSegmentId 河段id
          * upHzbId 乡级河长办id
      * 返回：
          * OK,报告成功
          * 4010: UNAUTHORIZED,不是村级和乡级河长
          * 4021：DATA_NOEXIST,数据不存在,匹配不到河段
6. 村河长取得自己的报告列表, /hzUser/report/village/lists,GET
   * **app端使用**
   * **角色：村级河长**
   * 参数：无
   * 返回：
       * OK,发出自己的报告列表，报告所有属性
       * 4010: UNAUTHORIZED,不是村级河长
7. 取得河长办报告详情,/hzUser/report/{instanceId}，GET
   * **app,web端使用**
   * **角色：村级，乡级河长，县河长办**
   * 如果是村级河长，必须是自己的河长报告
   * 如果是乡,县，市级河长，必须是发给对应河长办的河长办报告
    * 参数：
        * instanceId，唯一编码
    * 返回：
        * OK，河长报告所有属性
       * 4010: UNAUTHORIZED,不是属于自己的报告
8. 村河长删除自己的报告，/hzUser/report/delete/{instanceId},DELETE
   * **app端使用**
   * **角色：村级河长**
   * 只能删除发出后1小时内，且没有处理的报告
   * 参数：
       * instanceId，报告唯一编码
   * 返回：
       * OK，删除成功
       * 4010: UNAUTHORIZED,不是属于自己的报告
9. 村河长编辑自己的报告，/hzUser/report/update,POST
   * **app端使用**
   * **角色：村级河长**
   * 只能编辑发出后1小时内，且没有处理的报告
   * 参数：
       * instanceId，报告唯一编码
       * title,标题
       * content,内容       
   * 返回：
       * OK，编辑成功
       * DATA_NOEXIST，不存在       
       * 4010: UNAUTHORIZED,不是属于自己的报告
10. 河长发送任务给河长办,/hzUser/task/create,POST
   * 图片单独上传
   * **app端使用**
   * **角色：市，县，乡级河长**
   * 参数：
       * title 标题
       * content 内容
       * riverSegmentId,河段id
   * 返回：
       * OK，任务建立成功
11. 编辑河长办任务，/hzUser/task/update,POST
   * **app端使用**
   * **角色：市，县，乡级河长**
   * 只能编辑发出后1小时内，且没有处理的报告
   * 参数：
       * instanceId，报告唯一编码
       * title,标题
       * content,内容       
   * 返回：
       * OK，编辑成功
       * DATA_NOEXIST，不存在       
       * 4010: UNAUTHORIZED,不是属于自己的任务
12. 删除河长办任务，/hzUser/task/delete/{instanceId},DELETE
   * **app端使用**
   * **角色：市，县，乡级河长**
   * 只能删除发出后1小时内，且没有处理的报告
   * 参数：
       * instanceId，报告唯一编码
   * 返回：
       * OK，删除成功
       * 4010: UNAUTHORIZED,不是属于自己的任务
13. 取得河长发的任务列表,/hzUser/task/lists,GET
   * **app端使用**
   * **角色：市，县，乡级河长**
   * 参数：无
   * 返回：
       * OK，任务列表
14. 取得任务详情,/hzUser/task/details/{instanceId},GET
   * **web, app端使用**
   * **角色：市，县，乡级河长**
   * 河长自己的任务，能取得
   * 河长办上级河长发的任务，能取得
   * 参数：instanceId,唯一编码
   * 返回：
       * OK，任务所有属性
       * 4010: UNAUTHORIZED,不是属于自己的任务

## 二 RiverArchiveController,一河一档控制层
1. 微信取得一河一档列表,/riverArchive/weChat/archive/lists,GET
    * **weChat端使用**
    * **角色：无**   
    * 参数：
       * regionCode:行政区编码     
    * 返回：
        * OK,返回一河一档列表,每个对象包含所有一河一档属性+兴趣点列表
2. 河长取得一河一档列表,/riverArchive/hzUser/archive/lists,GET
    * **app端使用**
    * **角色：各级河长，巡河员**
    * 根据session中的regionCode,取得行政区，如果regionCode为空，使用行政区id，取得市级行政区编码
    * 如果是巡河员，使用行政区id，取得市级行政区编码
    * 参数：无
    * 返回：
        * OK,返回一河一档列表,每个对象包含所有一河一档属性+兴趣点列表 
        
## 二 RiverSegmentController 河段控制层
1. 取得河段详情, /riverSegment/riverSegment/{id},GET
    * **app端使用**
    * **角色：各级河长**
    * 参数：id:河段
    * 返回：
        * OK，河段属性：{name,level(河段级别),length,areaCoors,leftCoors,rightCoors,beginStation,endStation,regionName(行政区名)，hzUser:{name，phone},hzb(name,phone)}
        * 4022，DATA_REFUSE，请求的河段不属于自己
2. 取得河长管辖的河段,/riverSegment/lists,GET
    * **app端使用**
    * **角色：各级河长**
    * 参数：无
    * 返回：
        * OK,河道列表，[{id,name,level(河段级别),length,beginStation,endStation,regionName(行政区名),times(本月巡河次数)},{...}]
3. 移动端原生取得河段的面坐标,/riverSegment/android/coors,GET
    * **app端使用**
    * 参数：
        * id:河段id
        * token:原生访问传递的token
    * 返回：
        * OK,{areaCoors:[{lat:36.934,lon:102.23,ele:0}, {...}], leftCoors:[{...},{...}], rightCoors:[{...},{...}]}
4. 取得河段列表，/riverSegment/region/lists, GET
    * **webChat使用**
    * **角色：weChat**
    * 参数：
        * regionCode：行政区编码, 这个regionCode是当先位置的县区级行政编码
    * 返回：
        * OK，行政区下的所有河段，[{id,name},{..}]，如果行政区下没有河道，返回流过市州的所有河流id和name
        
## 三 CruiseController 巡河控制层
1. 取得河段巡河记录类型,/cruise/recordType,GET
    * **weChat, app端使用**
    * **角色：weChat, 各级河长**
   * 参数：无
   * 返回：
       * OK,[{type:类型英文,value:类型中文，description:["xxx","..."]}]
2. 上传巡河记录,/cruise/add,POST
    * **app端使用**
    * **角色：各级河长**
   * 参数：
       * riverSegmentId:河道id
       * cruiseRecords:[{type,description,content,coor,address,,instanceId(唯一标识),interestId(关注点id)},{..}]
       * cruise:{beginTime(开始时间),content,path(巡河路径),duration(巡河时长),length(巡河里长),instanceId:唯一标识}
   * 返回：
       * OK,建立成功
       * 4022，DATA_REFUSE，请求的河段不属于自己
3. 查询河段所有巡河,/cruise/lists/{riverSegmentId},GET
    * **app端, web端使用**
    * **角色：各级河长, 河长办用户**
    * 参数：riverSegmentId，河道id
    * 返回：
        * OK，河长在这个河段的巡河记录,[{instanceId,beginTime,conten,path(巡河路径),duration(巡河时长),length(巡河里长),recordNum(记录数)}]
        * 4022，DATA_REFUSE，请求的河段不属于自己
4. 查看巡河详情,/cruise/cruise/{instanceId},GET
    * **app端, web端使用**
    * **角色：各级河长, 河长办用户**
    * 巡河必须是属于河长或河长办
    * 参数：instanceId，巡河记录标识
    * 返回：
        * OK，{beginTime,content,path,duration,length,cruiseRecords:[{type,description,content,coor,address,instanceId,interest:{name,type}},{..}]}
5. 查看巡河，/cruise/lists/byHzb,GET
    * **app端, web端使用**
    * **角色：河长办用户**
    * 参数：
        * date，long类型，包含正确的年份，月份，日期指定为1号
    * 返回：
        * OK，河长办下所有河段，每个河段下当月的巡河记录,[{riverSegmentId:xx,riverSegmentName:xx,cruise:[{instanceId,beginTime,conten,path(巡河路径),duration(巡河时长),length(巡河里长),recordNum(记录数)},{...}]}, {...}]

## 四 HzbUserController,河长办用户控制层
1. 河长办用户登录,/hzbUser/login,POST
    * **web端使用**
    * **角色：无**
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
    * **web端使用**
    * **角色：无**
    * 参数：password:密码；verification:验证码
    * 返回：
        * OK: 返回用户对象所有属性
        * 4041: CODE_INVALID 验证码过期
        * 4042: CODE_ERROR 验证码输入错误
3. 登录发送验证码: /hzbUser/login/getLoginVerify
    * **web端使用**
    * **角色：无**
   * 参数
       * phone:手机号
   * 返回：
       * OK:发送成功
       * 4021：DATA_NOEXIST 账号不存在
4. 修改密码:在基本资料的修改密码处点击保存时调用， /hzbUser/updatePassword POST
    * **web端使用**
    * **角色：河长办用户**
   * 参数：
       * oldPassword:当前密码
       * newPassword：新密码
   * 返回：
       * OK:修改成功
       * FAIL:当前密码错误
5. 获取当前河长办用户：/hzbUser/getUser GET
    * **web端使用**
    * **角色：河长办用户**
    * 参数：
    * 返回：
       * OK(用户对象)，包含河长办用户所有属性，不包含河长办属性
       * 4011: NO_SESSION,未登录
6. 获取当前河长办信息：/hzbUser/hzb/info GET
    * **web端使用**
    * **角色：河长办用户**
    * 返回河长办属性，河长办管理的所有河段列表，行政区及管理的所有河长列表,下级河长办
    * 参数：
    * 返回：
       * OK，河长办信息，河长办管理的所有河段列表，下级河长办，行政区及管理的所有河长列表，{id,name:xx,regionName:xx,remark:xx,phone:xx, riverSegments:[{id,name:xx,length:xx,beginStation:xx,endStation:xx,riverName:xx,regionName:xx,hzUserId,hzUserName}, {...}], downHzbs:[{id,name:xx,regionId,regionName:xx,remark:xx,phone:xx}, {...}],regions:[{id:xx,name:xx,code:xx, hzUsers:[{id,name:xx,phone:xx,job:xx,type:xx}, {...}], childes:[{id:xx,name:xx,code:xx, hzUsers:[{id,name:xx,phone:xx,job:xx,type:xx}, {...}]}, {...}]}]}
       * 4011: NO_SESSION,未登录       
7. 河长办用户注销,/hzbUser/logout POST
   * **web端使用**
   * **角色：河长办用户**
   * 参数：无
   * 返回：OK,注销成功
8. 河长办取得发给自己的报告列表,/hzbUser/report/receive/lists,GET
   * 河长办取得的是发给自己的报告
   * **web端使用**
   * **角色：各级河长办**
   * 参数：无
   * 返回：
       * OK,发给自己报告列表，报告所有属性
       * 4010: UNAUTHORIZED,不是河长办用户    
10. 河长办处理报告，/hzbUser/report/handle,POST
   * 回复图片放在编辑器中，不用单独上传
   * **app端使用**
   * **角色：各级河长办**
   * 处理发给河长办自己的报告
   * 参数：
       * instanceId,唯一编码
       * handleContent,内容
   * 返回：
       * OK,处理成功
       * 4010: UNAUTHORIZED,不是提交给自己的报告
11. 河长办向上级河长办发送报告，/hzbUser/report,POST
   * 内容图片放在编辑器中，不用单独上传
   * **app端使用**
   * **角色：乡，县级河长**
   * 乡河长办报告给县河长办，县河长办报告给市河长办，记录上级河长办id，河道id
   * 参数：
       * title,标题
       * content,内容
       * instanceId,唯一编码
       * riverSegmentId 河道id
   * 返回：
       * OK,报告成功
       * 4010: UNAUTHORIZED,不是县河长办
12. 河长办取得自己发出的报告，/hzbUser/report/send/lists,POST
   * 河长办取得的是自己发送给上级的河长办的报告
   * **web端使用**
   * **角色：各级河长办**
   * 参数：无
   * 返回：
       * OK,自己发出的报告列表，报告所有属性
       * 4010: UNAUTHORIZED,不是河长办用户    
13. 取得河长办报告详情,/hzbUser/report/details/{instanceId},GET
   * **wen端使用**
   * **角色：各级河长办用户**
   * 报告必须是属于自己河长办，或下级发给自己所属河长办的
   * 参数：
       * instanceId，报告唯一编码
   * 返回：
       * OK，返回报告所有属性
       * 4010: UNAUTHORIZED,不属于自己或下级报告
14. 河长办删除自己的报告，/hzbUser/report/{instanceId},DELETE
   * **app端使用**
   * **角色：乡，县级河长**
   * 只能删除发出后1小时内，且没有处理的报告
   * 参数：
       * instanceId，报告唯一编码
   * 返回：
       * OK，删除成功
       * DATA_NOEXIST，不存在
       * 4010: UNAUTHORIZED,不是属于自己的报告
15. 河长办编辑自己的报告，/hzbUser/report/update,POST
   * **app端使用**
   * **角色：乡，县级河长**
   * 只能编辑发出后1小时内，且没有处理的报告
   * 参数：
       * instanceId，报告唯一编码
       * title,标题
       * content,内容
   * 返回：
       * OK，编辑成功
       * DATA_NOEXIST，不存在       
       * 4010: UNAUTHORIZED,不是自己的报告
16. 河长办取得发给自己的任务列表,/hzbUser/task/lists,GET
   * 河长办取得的是发给自己的任务
   * **web端使用**
   * **角色：各级河长办**
   * 参数：无
   * 返回：
       * OK,发给自己任务列表
       * 4010: UNAUTHORIZED,不是河长办用户
17. 河长办取得任务详情,/hzbUser/task/details/{instanceId},GET
   * **wen端使用**
   * **角色：各级河长办用户**
   * 参数：
       * instanceId，报告唯一编码
   * 返回：
       * OK，返回所有属性，包括任务的图片列表
       * 4010: UNAUTHORIZED,不是发给自己的任务
18. 河长办回复河长任务，/hzbUser/task/reply,POST
   * 回复图片放在编辑器中，不用单独上传
   * **app端使用**
   * **角色：各级河长办**
   * 处理发给河长办自己的任务
   * 参数：
       * instanceId，唯一编码
       * handleContent,内容
   * 返回：
       * OK,处理成功
       * 4010: UNAUTHORIZED,不是自己的任务             

## 五 MatterController，事件管理

    * 前台事件图片上传
    * 事件状态：0-登记 1-交办 2-责任退回 3-承办 4-重办 5-办结 6-审核退回 7-归档
     
1. 事件登记,/matter/register,POST
    * **web端使用**
    * **角色：市河长办用户**
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
    * **web端使用**
    * **角色：市河长办用户**
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
    * **web端使用**
    * **角色：县河长办用户**
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
    * **web端使用**
    * **角色：县河长办用户**
    * 事件状态改为办结
    * 办结时可以上传图片，由前台处理
    * 参数：
        * id：事件id
        * description: 办结描述
        * 4111: MATTER_TYPE_ERROR 事件状态错误
        * 4022: DATA_REFUSE，事件不属于该河长办        
5. 事件审核不通过，审核退回,/matter/returnBack,POST
    * **web端使用**
    * **角色：市河长办用户**
    * 事件状态改为审核退回
    * 审核时可以上传图片，由前台处理
    * 参数：
        * id： 事件id
        * description: 审核描述
    * 返回:
        * OK: 退回成功       
        * 4111: MATTER_TYPE_ERROR 事件状态错误
6. 事件归档,/matter/archive,POST
    * **web端使用**
    * **角色：市河长办用户**
    * 事件状态改为归档
    * 参数：
        * id： 事件id
        * description: 审核描述 
    * 返回：
        * OK: 归档成功
        * 4111: MATTER_TYPE_ERROR 事件状态错误
7. 事件催办,/matter/urgent,POST
    * **web端使用**
    * **角色：市河长办用户**
    * 市河长办发起
    * 事件状态必须是承办，才能接受催办
    * 参数：
        * id：事件id
        * content: 催办内容
    * 返回：
        * OK：催办成功
        * 4111: MATTER_TYPE_ERROR 事件状态错误
8. 事件催办回复,/matter/urgent/result,POST
    * **web端使用**
    * **角色：县河长办用户**
    * 县区河长办分回复
    * 参数：
        * id: 事件催办id
        * result: 回复内容
    * 返回：
        * OK：回复成功
        * 4022: DATA_REFUSE，事件催办不属于该河长办        
9. 事件反馈,/matter/feedBack,POST
    * **web端使用**
    * **角色：县河长办用户**
    * 县区河长办发起
    * 事件状态必须是承办，才能反馈
    * 参数：
        * id：事件id
        * content: 反馈内容
    * 返回：
        * OK：反馈成功
        * 4111: MATTER_TYPE_ERROR 事件状态错误
10. 事件反馈回复：/matter/feedBack/result,POST
    * **web端使用**
    * **角色：市河长办用户**
    * 市河长办回复
    * 参数：
        * id: 事件催办id
        * result: 回复内容
    * 返回：
        * OK：回复成功
        * 4010: UNAUTHORIZED，非市级河长办不能进行操作
9. 事件责任退回,/matter/deliver/returnBack,POST
    * **web端使用**
    * **角色：县河长办用户**
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
   * **web端使用**
   * **角色：市河长办用户，县河长办用户**
   * 参数：无
   * 返回:
       * OK，承办单位所有属性，【{id：xxx,code:xxx, ...},{id:xxx,code:xxx,...}】
       * FAIL, 行政区编码和河长办不一致
11. **`取得办理的事件列表,/matter/handle/lists,GET`**
   * **web端使用**
   * **角色：市河长办用户，县河长办用户**
   * 取得未归档的事件列表
   * 如果是市级河长办，取得事件regionCode等于市级regionCode的所有事件
   * 如果是县级河长办，根据外键取得事件列表
   * 参数：无
   * 返回:
       * OK，列表，事件对象所有属性，不包含文件列表，包含子对象
12. **`取得归档事件列表,/matter/archive/lists,GET`**
   * **web端使用**
   * **角色：市河长办用户，县河长办用户**
   * 取得已归档的事件列表
   * 如果是市级河长办，取得事件regionCode等于市级regionCode的所有事件
   * 如果是县级河长办，根据外键取得事件列表
   * 参数：无
   * 返回:
       * OK，列表，事件对象所有属性，不包含文件列表，包含子对象
13. 编辑器上传事件图片，/matter/image/create,POST
    * 涉及到登记，承办，办结时，编辑器上传的图片
    * **web端使用**
    * **角色：县，市河长办用户**    
    * 前台编辑器上传图片时调用后台接口，把图片上传到阿里云的/qqsl/hzy/{regionCode}/matter/{code}/image中，上传时使用 https://www.cnblogs.com/jdonson/archive/2009/07/22/1528466.html 方式生成图片的唯一编码
    * 上传时对图片进行压缩，以便用户能快速浏览，参见https://blog.csdn.net/niuch1029291561/article/details/17377903 ,压缩到图片宽度600px
    * 参数： 
    * 使用HttpServletRequest request，转换为MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request，取得图片，参见https://blog.csdn.net/qq_24419601/article/details/79784773
    * 返回:
        * OK,数据：{"fileName":"文件名.文件格式","url":"上传成功后得资源路径url"} ,资源路径url是阿里云的路径
        * FAIL           
14. 查看事件详情,/matter/{code},GET
    * **web端使用**
    * **角色：市河长办用户，县河长办用户**
    * 列出事件所有数据
    * 参数：code，督查督办编号
    * 返回：
        * 事件所有属性，事件图片list
        * 事件处理列表，每个列表中的承办文件list，办结文件list
        * 事件下发,上报列表
        
```
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
    feedBacks: [
        {content:xxx,result:xxx,type:xxx},
        {...}
    ]
}
```

15. **河长取得所管理河段的所有事件列表,/matter/byHzUser/lists,GET`**
    **web端使用**
    **角色：河长办用户*
    * 参数：无
    * 返回:
        * OK，列表，事件对象所有属性，不包含文件列表


## 六 ArticleController 新闻动态和政策方案 
1. 取得新闻动态列表,/article/newses,GET
    * **weChat,web端使用**
    * **角色：无**    
    * 参数：
        * regionCode：行政区编码
    * 返回：
        * OK，所属行政区的新闻动态列表
        * FAIL，行政区不存在
2. 取得政策法规列表，/article/laws, GET
    * **weChat,web端使用**
    * **角色：无**    
    * 参数：
       * regionCode：行政区编码
    * 返回：
        * OK，全国范围的政策法规+市级的政策法规列表  
        * FAIL，行政区id不存在
3. 取得新闻动态和政策法规详情,/article/article/{id},GET
    * **weChat,web端使用**
    * **角色：无**    
    * 参数：id: 新闻动态id
    * 返回：
        * OK，新闻动态所有属性
        * FAIL，id不存在
4. 新建文章，/article/admin/create, POST
    * **web端使用**
    * **角色：河长办管理员**    
    * 河长办用户操作，参数中不需要regionCode，因为session中有这个信息  
    * 参数： 
        * title,文章标题
        * content，文章内容
        * imageUrl，图片
        * type，WATERPOLICY(水利法规), WATERNEW(新闻动态)
        * level，CITY(州级),COUNTY(县级)
    * 返回:
        * OK,保存成功
5. 修改文章，/article/admin/update, POST
    * **web端使用**
    * **角色：河长办管理员**    
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
    * **web端使用**
    * **角色：河长办管理员**    
    * 前台编辑器上传图片时调用后台接口，把图片上传到阿里云的qqslimage/hzy/{regionCode}/article/中，上传时使用 https://www.cnblogs.com/jdonson/archive/2009/07/22/1528466.html 方式生成图片的唯一编码
    * 上传时对图片进行压缩，以便用户能快速浏览，参见https://blog.csdn.net/niuch1029291561/article/details/17377903 ,压缩到图片宽度600px
    * 参数： 
    * 使用HttpServletRequest request，转换为MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request，取得图片，参见https://blog.csdn.net/qq_24419601/article/details/79784773
    * 返回:
        * OK,数据：{"fileName":"文件名.文件格式","url":"上传成功后得资源路径url"} ,资源路径url是阿里云的路径
        * FAIL
 7. **`删除文章，/article/delete，POST`**
    * **web端使用**
    * **角色：河长办管理员**   
    * 参数： 
        * id: 文章id
    * 返回:
        * OK,删除成功
        * 4022，DATA_REFUSE，文章不属于自己
                        
## 七 StationController 测站
1. 取得测站列表：/station/lists
    * **weChat, web端使用**
    * **角色：无**   
    * 参数：
       * regionCode：行政区编码
    * 返回：
        * OK,行政区下的所有测站，包含属性，测站下仪表所有属性，返回和水利云返回测站列表数据保持一致
2. 取得token，用于和监测系统请求数据，/station/token
    * **角色：无** 
    * 参数：无
    * 返回：
        * OK，返回token
3. 取得仪表当日数据：121.40.82.11:8080/sensor/{code}, GET    
    * 参数：code，仪表编码，token
    * 返回：
        * OK，仪表当日数据
        * FAIL,仪表编码不存在        

## 八 PanoramaController，全景
1. 取得全景列表，/panorama/lists,GET
    * **weChat, web端使用**
    * **角色：无** 
    * 参数：
       * regionCode：行政区编码
    * 返回：
        * OK，行政区下的全景列表,包含name，createDate(建立时间),instanceId(唯一编码),thumbUrl(缩略图)，coor(坐标)，address(位置)

## 九 ComplaintControler，投诉
1. 公众号提交投诉,/complaint/weChat/submit,POST
    * **weChat使用**
    * **角色：weChat** 
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
    * **app端使用**
    * **角色：weChat** 
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
3. 公众号取得微信用户的投诉列表，/complaint/weChat/lists,GET
    * **weChat**
    * **角色：weChat** 
    * 参数：
        * unionId，微信唯一标识
    * 返回：
        * OK，返回投诉列表，只需返回id，createDate, title，imageUrl，riverSegmentName, status,handleName, handleDate
4. 取得投诉列表:/complaint/hzb/lists,GET
    * **web端使用**
    * **角色：市河长办** 
    * 参数：无
    * 返回：
        * OK，返回投诉列表，只需返回id，createDate, name, phone, title，imageUrl，riverSegmentName, status, handleName, handleDate
5. 取得投诉详情，/complaint/complaint,GET
    * **weChat, web端使用**
    * **角色：weChat, 市河长办** 
    * 投诉实体中增加imageUrl，保存第一张图的地址
    * 参数：
        * instanceId:投诉唯一标识
    * 返回：
        * OK，投诉所有属性
        * FAIL，唯一标识不存在
        * 4022，DATA_REFUSE，请求的投诉不属于自己
6. 回复投诉反馈，/complaint/handle,POST
    * **web端使用**
    * **角色：河长办用户** 
    * 如果投诉的regionCode和session中保存的regionCode不一致，返回错误
    * 参数：
        * id,投诉id
        * handleName,处理人姓名
        * handleContent,回复内容
    * 返回：
        * OK,回复成功        
        * 4022: DATA_REFUSE 投诉不属于河长办 
7. 编辑反馈回复，/complaint/handle/update,POST
    * **web端使用**
    * **角色：河长办用户** 
    * 如果投诉的regionCode和session中保存的regionCode不一致，返回错误
    * 参数：
        * id,投诉id
        * handleName,处理人姓名
        * handleContent,回复内容
    * 返回：
        * OK,编辑成功        
        * 4022: DATA_REFUSE 投诉不属于河长办  
8. 编辑器上传回复图片，/complaint/image/create,POST
    * **web端使用**
    * **角色：河长办用户** 
   * 前台编辑器上传图片时调用后台接口，把图片上传到阿里云的qqslimage/hzy/{regionCode}/complaint/handle/，投诉处理图片中，上传时使用https://www.cnblogs.com/jdonson/archive/2009/07/22/1528466.html 方式生成图片的唯一编码
    * 上传时对图片进行压缩，以便用户能快速浏览，参见https://blog.csdn.net/niuch1029291561/article/details/17377903 ,压缩到图片宽度600px
    * 参数： 
    * 使用HttpServletRequest request，转换为MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request，取得图片，参见 https://blog.csdn.net/qq_24419601/article/details/79784773
    * 返回:
        * OK,数据：{"fileName":"文件名.文件格式","url":"上传成功后得资源路径url"} ,资源路径url是阿里云的路径
        * FAIL

## 十 weChatController,微信控制层
 1. 根据code取得unionId,/weChat/unionId,GET
    * **weChat使用**
    * **角色：无**  
    * 取得unionId成功后，将unionId存在session中，直接登录
    * 参数：
        * code，微信code
        * regionCode:行政区编码 
    * 返回：
        * OK，{unionId:xxx}，目前没有unionId时，返回openId
2. 移动端微信用户登录,/weChat/login,POST
    * **app端使用**
    * **角色：无** 
   * 微信登录，把unionId存入session中
   * 参数：
       * unionId：微信unionId
       * regionCode:行政区编码       
   * 返回
       * OK，登录成功        


        
## 十二 OssController,阿里云Oss控制层
1. 根据treePath(阿里云路径)得到文件列表，/oss/objectFiles,GET
    * **weChat, app, web端使用**
    * **角色：weChat, 河长，河长办用户** 
    * 参数:
        * dir: 文件路径
        * bucket: bucket名
    * 返回：
        * OK，文件数组，ObjectFile对象全属性
2. 根据key得到文件访问路径，/oss/objectUrl，GET
    * **weChat, app, web端使用**
    * **角色：weChat, 河长，河长办用户** 
    * 参数:
        * key:文件key
        * bucket:bucket名
    * 返回：
        * OK，{url:xxx}
3. 获取sts安全令牌,用于上传或获取oss存储的文件, /oss/sts,GET
    * **weChat, app, web端使用**
    * **角色：weChat, 河长，河长办用户** 
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

## 十三 河长通报控制层,HzNoticeController
1. 新建河长通报,/hzNotice/admin/create,POST
    * **web端使用**
    * **角色：河长办管理员**    
    * 河长办用户操作，参数中不需要regionCode，因为session中有这个信息  
    * 参数： 
        * title,标题
        * content，内容
        * imageUrl，图片
        * type，0- 工作通报 1- 考核通报
    * 返回:
        * OK,保存成功
2. 取得河长通报列表,/heNotice/lists,GET
    * **weChat,app,web端使用**
    * **角色：无**    
    * 参数：
        * regionCode：行政区编码
    * 返回：
        * OK，所属行政区的河长通报
        * FAIL，行政区不存在
3. 取得河长通报详情,/hzNotice/details/{id},GET
    * **weChat,web,app端使用**
    * **角色：无**    
    * 参数：id: 通报id
    * 返回：
        * OK，通报所有属性
        * FAIL，id不存在
4. 修改通报，/hzNotice/admin/update, POST
    * **web端使用**
    * **角色：河长办管理员**    
    * 参数： 
        * id,id
        * title,标题        
        * content，内容
        * imageUrl，图片
        * type，0- 工作通报 1- 考核通报
    * 返回:
        * OK,保存成功
 5. 删除通报，/hzNotice/delete，POST
    * **web端使用**
    * **角色：河长办管理员**   
    * 参数： 
        * id: id
    * 返回:
        * OK,删除成功
        * 4022，DATA_REFUSE，通报不属于河长办的行政区         
5. 上传通报图片，/hzNotice/image/create POST
    * **web端使用**
    * **角色：河长办管理员**    
    * 前台编辑器上传图片时调用后台接口，把图片上传到阿里云的qqslimage/hzy/{regionCode}/heNotice/中，上传时使用 https://www.cnblogs.com/jdonson/archive/2009/07/22/1528466.html 方式生成图片的唯一编码
    * 上传时对图片进行压缩，以便用户能快速浏览，参见https://blog.csdn.net/niuch1029291561/article/details/17377903 ,压缩到图片宽度600px
    * 参数： 
    * 使用HttpServletRequest request，转换为MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request，取得图片，参见https://blog.csdn.net/qq_24419601/article/details/79784773
    * 返回:
        * OK,数据：{"fileName":"文件名.文件格式","url":"上传成功后得资源路径url"} ,资源路径url是阿里云的路径
        * FAIL
        
## 十四  巡河员报告控制层,PatrolUserController
1. 登录发送验证码: /patrolUser/login/getLoginVerify,GET
    * **app端使用**
    * **角色：无**
    * 巡河员登录时，取得验证码
    * 参数：
        * phone:登录手机号
    * 返回：
        * OK:发送成功
        * DATA_NOEXIST:手机号不存在
2. 登录：/patrolUser/login,POST
    * **app端使用**
    * **角色：无**
    * 巡河员每次登录都是用动态验证码    
    * 参数：
        * code:验证码
    * 返回:
        * OK:登录成功,返回巡河员详情，{name:xxx,location:xxx,phone:xx,riverSegment:{id,name:xx,length:xx,beginStation:xx,endStation:xx,riverName:xx,regionId,regionName:xx},upHzUser:{id,name:xx,regionId,regionName:xx.phone:xx,job:xx,type:xx}, hzb:{name,regionId,regionName,remark,phone}}
        * 4041: CODE_INVALID 验证码过期
        * 4042: CODE_ERROR 验证码输入错误
3. 取得巡河员名录，/patrolUser/lists,GET
    * **weChat,web, app端使用**
    * **角色：无**
    * 参数：
        * regionCode：行政区编码    
    * 返回： 
        * OK，格式: [{name:xxx,location:xxx,phone:xx,riverSegment(管辖河段):{id,name:xx,length:xx,beginStation:xx,endStation:xx,riverName:xx,regionId,regionName:xx}}
4. 上传巡河记录,/patrolUser/record/add,POST
    * **app端使用**
    * **角色：巡河员**
   * 参数：
       * riverSegmentId:河段id
       * record:{type,description,content,coor,address,,instanceId(唯一标识),interestId(关注点id)}
   * 返回：
       * OK,建立成功
       * 4022，DATA_REFUSE，请求的河段不属于自己
5. 取得巡河记录列表,/patrolUser/record/lists,GET
    * **app端使用**
    * **角色：巡河员,各级河长**
    * 参数： id:巡河员id
    * 返回：
       * OK,巡河员记录列表
6. 取得巡河记录详情，/patrolUser/record/{instanceId},GET
   * **app端使用**
   * **角色：巡河员,各级河长**
   * 参数： instanceId,唯一编码
   * 返回：
       * OK,巡河员记录属性，图片url列表
       * 4022，DATA_REFUSE，巡河员记录不属于自己
7. 河长取得所有所属巡河员列表,/patrolUser/lists/byHzUser,GET
   * **app端使用**
   * **角色：各级河长**
   * 参数：无
   * 返回：
       * OK,根据自己的河段列表，查询每个河段下的巡河员列表，组合起来，每个巡河员下的河段id，河段名也放在json中

```     
    巡河记录类型
    public enum CruiseRecordType {
        // 水体
        RIVER_BODY("水体异味,颜色异常"),
        // 河面
        RIVER_SURFACE("杂物漂浮,油污漂浮,病死动物"),
        // 河中
        RIVER_MIDDLE("围网养殖,污泥淤积,沉船，倾倒废土弃渣,工业固废和危废"),
        // 河岸
        RIVER_SIDE("生活垃圾,工业垃圾,建筑垃圾,农业生产废弃物,其他杂物堆放"),
        // 涉水活动
        ACTIVITY("非法捕鱼，非法采砂"),
        // 排污口
        OUTLET("污水直排,晴天雨水口有污水排放,废水颜色或气味异常,新增排污口,标识未设置"),
        // 河长牌
        BILLBOARD("未设置或或缺失,信息更新不及时,设置不规范,倾斜破损或变形变色老化"),
        // 水工建筑物
        BUILD("非法占有河道,涉水违章建(构)筑物,建筑物运行异常");

        private String type;

        CommonType(String type) {
            this.type = type;
        }

        public String getType() {
            return type;
        }

        public void setType(String type) {
            this.type = type;
        }

        public static CruiseRecordType valueOf(int ordinal) {
            if (ordinal < 0 || ordinal >= values().length) {
                throw new IndexOutOfBoundsException("Invalid ordinal");
            }
            return values()[ordinal];
        }
    }

    /**
     * 行政区级别
     */
    public enum RegionLevel {
        // 省
        PROVINCE,
        // 市
        CITY,
        // 县
        COUNTY,
        // 乡
        TOWN,
        // 村
        VILLAGE
    }

    /**
     * 河长级别
     */
    public enum HzType {
        // 总河长
        MASTER,
        // 副总河长
        SLAVE,
        // 河长
        NORMAL
    }    

    /**
     * 河流兴趣点类型
     */
    public enum RiverIntrestType {
        // 公示牌
        BILLBOARD,
        // 灌区
        IRRIGATION,
        // 水库
        RESERVOIR,
        // 污水处理厂
        SEWAGE_FACTORY,
        // 排污口
        OUTLET,
        // 水源
        WATER_SOURCE
    }    
```

```    
河长云相关图片路径：
    新闻
    qqslimage/hzy/{regionCode}/article/
    河长通报
    qqslimage/hzy/{regionCode}/hzNotice/    
    河流一河一档图片
    qqslimage/hzy/{regionCode}/riverArchive/
    投诉图片
    qqslimage/hzy/{regionCode}/complaint/{unionId}/{instanceId}/
    投诉处理图片
    qqslimage/hzy/{regionCode}/complaint/handle/
    河长巡河记录
    qqsl/hzy/{regionCode}/cruise/record/{instanceId}/
    巡河员巡河记录
    qqsl/hzy/{regionCode}/patrol/record/{instanceId}/    
    事件图片
    /qqsl/hzy/{regionCode}/matter/{code}/image/
    事件登记附件
    /qqsl/hzy/{regionCode}/matter/{code}/register
    事件承办附件
    /qqsl/hzy/{regionCode}/matter/{code}/{handleId}/handle
    事件办结附件
     /qqsl/hzy/{regionCode}/matter/{code}/{handleId}/complete
     河长办报告图片
     /qqsl/hzy/{regionCode}/report/{instanceId}
     河长办报告回复图片
     /qqsl/hzy/{regionCode}/report/{instanceId}/handle
     河长办任务图片
     /qqsl/hzy/{regionCode}/task/{instanceId}
     
```

```
河长角色：hzUser:city, hzUser:county, hzUser:town, hzUser:village
河长办角色：hzbUser:city, hzbUser:county, hzbUser:admin
weChat角色：weChatUser:simple
```