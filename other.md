<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [坐标转换,全景等操作接口设计](#%E5%9D%90%E6%A0%87%E8%BD%AC%E6%8D%A2%E5%85%A8%E6%99%AF%E7%AD%89%E6%93%8D%E4%BD%9C%E6%8E%A5%E5%8F%A3%E8%AE%BE%E8%AE%A1)
  - [一. ossController接口](#%E4%B8%80-osscontroller%E6%8E%A5%E5%8F%A3)
  - [二. fieldController接口(filed)，坐标转换](#%E4%BA%8C-fieldcontroller%E6%8E%A5%E5%8F%A3filed%E5%9D%90%E6%A0%87%E8%BD%AC%E6%8D%A2)
  - [三. 全景和兴趣点接口,interestController(/interest)](#%E4%B8%89-%E5%85%A8%E6%99%AF%E5%92%8C%E5%85%B4%E8%B6%A3%E7%82%B9%E6%8E%A5%E5%8F%A3interestcontrollerinterest)
  - [四. 项目操作接口,projectController(/project)](#%E5%9B%9B-%E9%A1%B9%E7%9B%AE%E6%93%8D%E4%BD%9C%E6%8E%A5%E5%8F%A3projectcontrollerproject)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

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
    * fileName:file对象若干个
    * 返回：
    * OK：上传成功；
    * FAIL：上传失败:坐标文件或格式错误;
    * EXIST：中心坐标点(行政区)未选择；
    * EXPIRED：您当前的套餐已过期，请重新购买
2. 工程布置坐标WGS84导出：writeExcel,GET
    * 参数：
    * projectId:项目id；
    * type：design或field
    * baseLevelType:基准面类型
    * WGS84Type:WGS84坐标格式
    * 返回：直接下载
3. 编辑建筑物:editBuild,POST
    * 这个接口用在两处，1是工程布置处，2是地图上编辑，两个地方的参数不用，后台都能处理
    * 参数：
    * 1 工程布置：
    * projectId:项目id
    * id:建筑物id
    * attribes:数组，包含{alias:xx,value:xx}格式的一系列属性
    * 2 地图上编辑
    * projectId:项目id
    * id:建筑物id
    * type:建筑物类型
    * remark:建筑物备注
    * 返回：
    * OK：保存成功
    * FAIL：保存失败
    * EXIST:建筑物不存在
    
```
	/**
	 * 坐标转换基准面类型
	 */
	public enum BaseLevelType {
		WGS84, CGCS2000;
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


