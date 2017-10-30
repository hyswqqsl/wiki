## 坐标转换,全景等操作接口设计
### 一. ossController接口
>
1. 取得文件访问url：getFileUrl，GET
    * 返回的文件url有效期30分钟
    * 参数：key:文件路径,bucketName:bucket名
    * 返回：OK：内容是{url：访问地址}，如果文件不存在没有内容
2. 取得文件列表：objectFiles，GET
    * 参数：id:文件夹路径
    * 返回：OK：取得成功，数据是文件列表

### 二. fieldController接口(filed)，坐标转换
>
1. 工程布置坐标导入：coordinateFile, POST
    * 参数：
    * projectId:项目id
    * baseLevelType:基准面类型
    * WGS84Type:WGS84坐标格式
    * commonPointStr:公共点json，[{xyh:{x:12.43,y:34.343,h:65.343},blh:{b: 34.34,l:343.4,h:324.34}},{xyh:{x:12.43,y:34.343,h:65.343},blh:{b: 34.34,l:343.4,h:324.34}},{xyh:{x:12.43,y:34.343,h:65.343},blh:{b: 34.34,l:343.4,h:324.34}}]
    * fileName:file对象若干个
    * 返回：
    * OK：上传成功；
    * FAIL：上传失败:坐标文件或格式错误;
    * EXIST：中心坐标点(行政区)未选择；
    * EXPIRED：您当前的套餐已过期，请重新购买
2. 四参数解算：coordinate/parameter/four/caculate, POST
    * 参数：
    * projectId:项目id
    * commonPointStr:公共点json，[{xy:{x:12.43,y:34.343},bl:{b: 34.34,l:343.4}},{xy:{x:12.43,y:34.343},bl:{b: 34.34,l:343.4}}]，两组点
    * 返回：
    * OK:解算成功，数据是解算结果，七参数{X:12.2,Y:12.4,Z:12.7,K:12.6,EX:12.5,EY:17.65,EZ:12.5}
    * FAIL:参数错误
3. 七参数解算：
    * 参数：
    * projectId:项目id
    * commonPointStr:公共点json，[{xyh:{x:12.43,y:34.343,h:65.343},blh:{b: 34.34,l:343.4,h:324.34}},{xyh:{x:12.43,y:34.343,h:65.343},blh:{b: 34.34,l:343.4,h:324.34}},{xyh:{x:12.43,y:34.343,h:65.343},blh:{b: 34.34,l:343.4,h:324.34}}]，三组点
    * 返回:
    * OK:解算成功，数据是解算结果，四参数{X0:12.2,Y0:12.4,T:12.7,K:12.6}
    * FAIL:参数错误
3. 工程布置坐标WGS84导出：writeExcel,GET
    * 参数：projectId:项目id；type：design或field
    * 返回：直接下载

```
	/**
	 * 坐标转换基准面类型
	 */
	public enum BaseLevelType {
		WGS84, BEIJING54, XIAN80, CHINA2000;
		public static BaseLevelType valueOf(int ordinal) {
			if (ordinal < 0 || ordinal >= values().length) {
				throw new IndexOutOfBoundsException("Invalid ordinal");
			}
			return values()[ordinal];
		}
	}

	/**
	 * WGS84坐标格式
	 */
	public enum WGS84Type {
		// 35.429898
		DEGREE,
		// 35:23
		DEGREE_MINUTE_1,
		// 35^o23'
		DEGREE_MINUTE_2,
		// 35:23:45
		DEGREE_MINUTE_SECOND_1,
		// 35^o23'45''
		DEGREE_MINUTE_SECOND_2;
		public static WGS84Type valueOf(int ordinal) {
			if (ordinal < 0 || ordinal >= values().length) {
				throw new IndexOutOfBoundsException("Invalid ordinal");
			}
			return values()[ordinal];
		}
	}

```

### 三. 全景和兴趣点接口,interestController(/interest)
基础兴趣点操作，全景审核，个人兴趣点功能从用户移到管理员，所有涉及到以上操作的路径都增加/admin/：
>
1. 保存基础兴趣点:saveBaseInterest改为admin/saveBaseInterest
2. 编辑基础兴趣点:updateBaseInterest改为admin/updateBaseInterest
3. 删除基础兴趣点:deleteBaseInterest改为admin/deleteBaseInterest
4. 获取兴趣点列表(基础):baseInterestList改为admin/baseInterestList
5. 个人兴趣点审核列表:interestReview改为admin/interestReview
6. 全景审核列表:panoramaReview改为admin/panoramaReview
7. 个人兴趣点审核通过:interestReviewSuccess改为admin/interestReviewSuccess
8. 全景审核通过:panoramaReviewSuccess改为admin/panoramaReviewSuccess
9. 个人兴趣点审核未通过:interestReviewFail改为admin/interestReviewFail
10. 全景审核未通过:panoramaReviewFail改为admin/panoramaReviewFail

### 四. 项目操作接口,projectController(/project)
> 
1. 项目图标类型定制保存：iconType/update,POST
    * 参数：id:项目id; iconType:图标类型
    * 返回：OK:保存成功；FAIL:保存失败

```java
    projct.java:
    /**
     * 项目图标类型
     */
    public enum IconType {
        STYLE_0,
        STYLE_1,
        STYLE_2,
        STYLE_3,
        STYLE_4,
        STYLE_5
    }```


