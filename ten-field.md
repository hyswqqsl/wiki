# 工程布置，外业测量及工程布置模板下载设计
在FieldController中，以/field开头
## 坐标转换接口设计
### 一. fieldController接口(filed)，坐标转换
>
1. *工程布置坐标导入：/field/uplaodCoordinate, POST*
    以前接口名为coordinateFile 
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
2. *工程布置坐标WGS84导出：/field/downloadCoordinate,POST*
    以前接口名为writeExcel，GET请求 
    * 参数：
        * projectId:项目id；
        * type：design或field
        * baseLevelType:基准面类型
        * WGS84Type:WGS84坐标格式
    * 返回：
        * OK，下载成功
        * FAIL，下载失败
3. 编辑建筑物:/field/editBuild,POST
    * 这个接口用在两处，1是工程布置处，2是地图上编辑，两个地方的参数不同，后台都能处理
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
## 二. 坐标模板下载接口设计
> 
1. 获取所有模板信息: /field/templateInfo,GET
    * 参数：无
    * 返回：OK，数据是所有模板信息的json，见下表
2. 下载选择的模板：/field/downloadTemplete, POST
    前台选择线面和建筑物类型后，需根据线面和建筑物分类分成三组，分别显示，向后台传递参数时，根据线、面、建筑物顺序排列所选类型。
    * 参数：json:["type1","type2","type3"],不管是线面还是建筑物，只¯传递类型数组
    * 返回：
        * OK，下载成功
        * FAIL，下载失败
    
```
获取所有模板信息接口返回：
{"total":null,
"data":[{"baseType":"line","type":"POINT","name":"普通点"},{"baseType":"line","type":"GSGG","name":"供水干管"},{"baseType":"line","type":"GSZG","name":"供水支管"},{"baseType":"line","type":"GSDG","name":"供水斗管"},{"baseType":"line","type":"GSGQ","name":"供水干渠"},{"baseType":"line","type":"GSZQ","name":"供水支渠"},{"baseType":"line","type":"GSDQ","name":"供水斗渠"},{"baseType":"line","type":"PSGG","name":"排水干管"},{"baseType":"line","type":"PSZG","name":"排水支管"},{"baseType":"line","type":"PSDG","name":"排水斗管"},{"baseType":"line","type":"PSGQ","name":"排水干渠"},{"baseType":"line","type":"PSZQ","name":"排水支渠"},{"baseType":"line","type":"PSDQ","name":"排水斗渠"},{"baseType":"line","type":"GONGGXM","name":"公共线面"},{"baseType":"line","type":"FHD","name":"防洪堤"},{"baseType":"line","type":"PHQ","name":"排洪渠"},{"baseType":"area","type":"GGFW","name":"灌溉范围"},{"baseType":"area","type":"BHFW","name":"保护范围"},{"baseType":"area","type":"GSQY","name":"供水区域"},{"baseType":"area","type":"ZLFW","name":"治理范围"},{"baseType":"area","type":"KQYMFW","name":"库区淹没范围"},{"baseType":"area","type":"SHUIY","name":"水域"},{"baseType":"build","type":"QS","name":"泉室"},{"baseType":"build","type":"JSLD","name":"截水廊道"},{"baseType":"build","type":"DKJ","name":"大口井"},{"baseType":"build","type":"TJ","name":"土井"},{"baseType":"build","type":"JJ","name":"机井"},{"baseType":"build","type":"LC","name":"涝池"},{"baseType":"build","type":"FSZ","name":"闸"},{"baseType":"build","type":"DHX","name":"倒虹吸"},{"baseType":"build","type":"DS","name":"跌水"},{"baseType":"build","type":"XIAOLC","name":"消力池"},{"baseType":"build","type":"HUT","name":"护坦"},{"baseType":"build","type":"HAIM","name":"海漫"},{"baseType":"build","type":"DC","name":"渡槽"},{"baseType":"build","type":"HD","name":"涵洞"},{"baseType":"build","type":"SD","name":"隧洞"},{"baseType":"build","type":"NK","name":"农口"},{"baseType":"build","type":"DM","name":"斗门"},{"baseType":"build","type":"GLQ","name":"公路桥"},{"baseType":"build","type":"CBQ","name":"车便桥"},{"baseType":"build","type":"GJQD","name":"各级渠道"},{"baseType":"build","type":"JCJ","name":"检查井"},{"baseType":"build","type":"FSJ","name":"分水井"},{"baseType":"build","type":"GSJ","name":"供水井"},{"baseType":"build","type":"JYJ","name":"减压井"},{"baseType":"build","type":"JYC","name":"减压池"},{"baseType":"build","type":"PAIQJ","name":"排气井"},{"baseType":"build","type":"FANGSJ","name":"放水井"},{"baseType":"build","type":"XSC","name":"蓄水池"},{"baseType":"build","type":"GJGD","name":"各级管道"},{"baseType":"build","type":"DANGQ","name":"挡墙"},{"baseType":"build","type":"GF","name":"谷坊"},{"baseType":"build","type":"YDB","name":"淤地坝"},{"baseType":"build","type":"YHD","name":"溢洪道"},{"baseType":"build","type":"DG","name":"滴灌"},{"baseType":"build","type":"PT","name":"喷头"},{"baseType":"build","type":"JSS","name":"给水栓"},{"baseType":"build","type":"SFSS","name":"施肥设施"},{"baseType":"build","type":"GLXT","name":"过滤系统"},{"baseType":"build","type":"LD","name":"林地"},{"baseType":"build","type":"GD","name":"耕地"},{"baseType":"build","type":"CD","name":"草地"},{"baseType":"build","type":"JMQ","name":"居民区"},{"baseType":"build","type":"GKQ","name":"工矿区"},{"baseType":"build","type":"DL","name":"电力"},{"baseType":"build","type":"CJJT","name":"次级交通"},{"baseType":"build","type":"HEC","name":"河床"},{"baseType":"build","type":"SHUIM","name":"水面"},{"baseType":"build","type":"SHUIW","name":"水位"},{"baseType":"build","type":"SHUIWEN","name":"水文"},{"baseType":"build","type":"YUL","name":"雨量"},{"baseType":"build","type":"SHUIZ","name":"水质"},{"baseType":"build","type":"BZ","name":"泵站"},{"baseType":"build","type":"DZCF","name":"电站厂房"},{"baseType":"build","type":"DIZD","name":"地质点"},{"baseType":"build","type":"TSD","name":"其他"}],
"status":2000}
```

