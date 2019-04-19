
# 河长云第三次迭代接口

## 河长云类结构
![image](https://github.com/hyswqqsl/wiki/blob/master/image/%E6%B2%B3%E9%95%BF%E4%BA%91%E6%96%B0%E6%9E%B6%E6%9E%84.jpg)

## 一 平台管理,platfromManageController
* 角色:系统管理员
1 新建平台，/platfromManage/platform/create,POST
   * 参数:
	 * name: 平台名称
	 * regionCode：对应的行政区编码
	 * type: 平台类型，FULL_PLATFORM：完整，WECHAT_PLATFORM： 微信公众号
	 * appId: 微信appId
	 * appSecret: 微信appSecret
   * 返回:
	 * OK，新建成功
	 * DATA_EXIST: 对应的行政区编码已有平台，或appId已使用	 
2 编辑平台，/platformManage/platform/edit,PUT
   * 参数:
	 * id: 平台id
	 * name: 平台名称
	 * regionCode：对应的行政区编码
	 * type: 平台类型，FULL_PLATFORM：完整，WECHAT_PLATFORM： 微信公众号
	 * appId: 微信appId
	 * appSecret: 微信appSecret
   * 返回:
	 * nOK，编辑成功
	 * DATA_NOEXIST: 平台不存在
	 * DATA_EXIST: 对应的行政区已有平台，或appId已使用
	 * DATA_LOCK: 平台下已有行政区结构，不能修改行政区编码   
