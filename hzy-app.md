# 河长云app接口设计


![Main](/uploads/9ba17c1ebcfedf424c4838844a6c17f4/Main.png)


![ClassDiagram1](/uploads/4db4f09a675c86cfc59c48c8f7907bd0/ClassDiagram1.png)


## 一 河长登录接口
1. 登录发送验证码: /hzUser/login/getLoginVerify,GET
    * 河长登录时，取得验证码
    * 参数：
        * phone:登录手机号
    * 返回：
        * OK:发送成功
        * DATA_NOEXIST:手机号不存在
2. 登录：/hzUser/login,POST
    * 河长每次登录都是用动态验证码    
    * 参数：
        * code:验证码
    * 返回:
        * OK:登录成功
        * 4041: CODE_INVALID 验证码过期
        * 4042: CODE_ERROR 验证码输入错误

## 二 RiverSegmentController 河段控制层
1. 取得河段详情, /riverSegment/{id},GET
    * 参数：id:河段id
    * 返回：
        * OK，河段属性：{name,level(河段级别),length,coors,beginStation,endStation,regionName(行政区名)，hzUser:{name，phone},hzbUser(name,phone)，concerns:[{id,name,type,coor,address}]}
        * 4022，DATA_REFUSE，请求的河段不属于自己
2. 取得河长管辖的河段,/riverSegment/lists/{hzUserId},GET
    * 参数：
        * hzUserId:河长用户id
    * 返回：
        * OK,河道列表，[{id,name,level(河段级别),length,beginStation,endStation,regionName(行政区名)}]
        
## 三 CruiseController 巡河控制层
1. 取得河段巡河记录类型,/cruise/recordType,GET
   * 参数：无
   * 返回：
       * OK,[{type:类型英文,value:类型中文，description:["xxx","..."]}]
2. 上传巡河记录,/cruise/add,POST
   * 参数：
       * riverSegmentId:河道id
       * cruiseRecords:[{type,description,content,coor,address,concernId（关注点id）},{..}]
       * cruise:{beginTime(开始时间),content,path(巡河路径),duration(巡河时长),length(巡河里长),instanceId:唯一标识}
   * 返回：
       * OK,建立成功
       * 4022，DATA_REFUSE，请求的河段不属于自己
3. 查询河段所有巡河记录,/cruise/lists/{riverSegmentId},GET
    * 参数：riverSegmentId，河道id
    * 返回：
        * OK，河长在这个河段的巡河记录,[{id,beginTime,conten,path(巡河路径),duration(巡河时长),length(巡河里长),recordNum(记录数)}]
        * 4022，DATA_REFUSE，请求的河段不属于自己
4. 查看巡河记录详情,/cruise/cruise/{instanceId},GET
    * 参数：instanceId，巡河记录标识
    * 返回：
        * OK，{beginTime,conten,path,duration,length,cruiseRecords:[{type,description,content,coor,address,concern:{name,type}},{..}]}
        * 4022，DATA_REFUSE，请求的记录不属于自己


```
   /**
     * 巡河记录类型
     */
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
     * 河段关注点类型
     */
    public enum RiverSegmentConcernType {
        // 排污口
        OUTLET,
        // 公示牌
        BILLBOARD
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
```


```
河长云相关图片路径：
    qqslimage/hzy/{regionCode}/article/，新闻
    qqslimage/hzy/{regionCode}/river/，河流
    qqslimage/hzy/{regionCode}/complaint/{unionId}/{instanceId}/，投诉图片
    qqslimage/hzy/{regionCode}/complaint/{unionId}/{instanceId}/{handle}，投诉处理图片
    qqslimage/hzy/{regionCode}/cruise/record/{instanceId}/,巡河记录
```





