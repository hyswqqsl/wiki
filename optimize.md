## 优化-用户与子账号、消息、日志相关接口

### 一. 子账号信息说明
1. 子账号没有认证流程，所以在注册时，就应该输入真实姓名；如果子账号是被邀请的，那么登录进去后，首先弹出要求完善真实姓名的对话框
2. 子账号和用户类似，也是在账号管理处上传头像，不用base64保存，头像由前台保存到阿里云，路径是qqsl/account/{accountId}/avatar.jpg
3. 有验证码的不用再发手机号或邮箱！！
4. 子账号登录后，如果发现没有真实姓名，应提示完善真实姓名，弹出用户管理界面，并拒绝其他操作

### 二. AccountController 子账号控制层

1.  注册时发送手机验证码:/account/phone/getRegistVerify
    * 参数：phone:手机号
    * 返回：OK:发送成功
    * FIAL:手机号不合法
    * EXIST：手机号已被使用
2. 修改密保手机发送验证码： /account/phone/getRegistVerify 改为 /account/phone/getUpdateVerify ??
    * 参数：phone:手机号
    * 返回：OK:发送成功
    * FIAL:手机号不合法
    * EXIST：手机号已被使用
3. 手机找回密码时发送验证码：/account/getVerifyCode 改为/account/phone/getGetbackVerify ??
    * 参数：phone:手机号
    * 返回：OK:发送成功
    * FIAL:手机号不合法
    * EXIST：账号不存在
4. web端登录发送验证码: /account/getVerifyCode 改为 /account/login/getLoginVerify
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，如果手机号注册，发送手机验证码；如果是email，如果邮箱绑定，发送邮箱验证码
    * 参数：code:登录凭证
    * 返回：OK:发送成功,FAIL:手机号/邮箱不合法，EXIST：账号不存在
5. email绑定时发送验证码:/account/email/getBindVerify
    * 参数：email:手机号
    * 返回：OK:发送成功,FAIL:邮箱不合法，EXIST：邮箱已存在
6. email找回密码时发送验证码：/account/email/getGetbackVerify
    * 参数：email:邮箱
    * 返回：OK:发送成功,FAIL:email不合法，EXIST：邮箱不存在
7. 注册用户：/account/register
    * 参数：**name:真实姓名**，password:密码，验证码：verification:验证码
    * 返回：OK:注册成功，INVALID:验证码过期，FAIL:验证码错误  
8. 手机找回密码: /account/changePassword 改为 /account/phone/getbackPassord，忘记密码时找回密码 ??
    * 参数：verification:验证码;passord:重设的密码; 
    * 返回：OK:修改成功，INVALID:验证码过期，FAIL:验证码错误
9. email找回密码:/account/email/getbackPassord，忘记密码时找回密码
    * 参数：verification:验证码;passord:重设的密码;   
    * 返回：OK:修改成功，INVALID:验证码过期，FAIL:验证码错误 
10. 修改密码:在基本资料的修改密码处点击保存时调用， /account/updatePassword POST
    * 参数：oldPassword:当前密码，newPassword：新密码
    * 返回：OK:修改成功，UNKNOWN:当前密码错误
11. 修改密保手机：在基本资料的密保手机处点击保存时调用， /account/changePhone 改为 /account/updatePhone POST ??
    * 参数：verification:验证码
    * 返回：OK:修改成功，INVALID:验证码过期，FAIL:验证码错误 
12. 绑定邮箱：在基本资料里的绑定邮箱处点击保存时调用，/account/updateEmail POST
    * 参数：verification:验证码
    * 返回：OK:修改成功，INVALID:验证码过期，FAIL:验证码错误 
13. 获取子账号信息：在基本资料的基本信息处显示，/account/getAccount GET
    * 参数：
    * 返回：OK(用户对象)，包含昵称、手机号，身份认证状态，企业认证状态，绑定微信的昵称，微信openid等
16. web端登录：/account/login 改为 /account/web/login POST ??
    * 取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 参数：code：登录凭证，手机号或邮箱；password:密码；cookie：cookie
    * 返回：OK(用户对象):登录成功；FAIL：密码错误；EXIST:用户不存在;OTHER:需要验证码，UNKNOWN:用户已锁定
17. 移动端登录：/account/phone/login POST
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码；如果是email，通过email取得用户，验证用户密码,微信第一次登录也是这个
    * 参数：code：登录凭证，手机号或邮箱；password:密码
    * 返回：OK(用户对象):登录成功；FAIL：密码错误；EXIST:用户不存在,UNKNOWN:用户已锁定
20. web端验证码登录：/account/loginByVerifyCode 改为 /account/web/loginByVerify POST ??
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 参数：password:密码；verification:验证码
    * 返回：OK(用户对象):登录成功；FAIL：密码或验证码错误；UNKNOWN:用户已锁定；INVALID:验证码过期
21. 判断输入的用户密码是否正确：/account/checkPassword,POST
    * 进入账号管理时，要求输入密码，保证安全
    * 参数：password:密码
    * 返回：OK:密码正确；FAIL：密码错误
    
### 三. UserMessage用户消息
消息内容中可以加入html超链接，可以点击项目名或测站名，跳转相应的页面。这样的话，就不用再userMessage中保存projectId
```
public class UserMessage extends BaseEntity {

	private static final long serialVersionUID = -3452664373859421059L;

	/** 用户 */
	private User user;
	/** 内容 */
	private String content;
	/** 状态 */
	private Status status;
	/** 类型 */
	private Type type;

	/**
	 * 类型
	 */
	public enum Type {
		// 购买套餐，包含购买，续费，升级
		BUY_PACKAGE,
		// 购买测站，包含购买，续费
		BUY_STATION,
		// 分享项目
		SHARE_PROJECT,
		// 分享测站
		SHARE_STATION
	}

	/**
	 * 状态
	 */
	public enum Status {
		UNREAD,
		READED
	}
}
```
### 四. AccountMessage子账号消息
消息内容中可以加入html超链接，可以点击项目名或测站名，跳转相应的页面。这样的话，就不用再userMessage中保存projectId
```
public class AccountMessage extends BaseEntity{

    private static final long serialVersionUID = -3452664373859421059L;

    private Account account;
    /** 内容 */
    private String content;
    /** 状态 */
    private Status status;
    /** 类型 */
    private Type type;

    /**
     * 类型
     */
    public enum Type {
        INVITE__ACCOUNT,
        SHARE_PROJECT
    }

    /**
     * 状态
     */
    public enum Status {
        UNREAD,
        READED
    }
}
```

### 五. 项目日志

<div align="center">
   <img src="http://112.124.104.190:10001/soft/wiki/uploads/5a5fecd9adf34bb805d9c9cf571e5407/log.png" width="680px" />
</div>

日志目前只需保存项目操作，分为要素和文件，由于用户只能编辑自己的项目，子账号能编辑协同给他的项目，所以在内容里记录“用户或子账号”操作了那个要素，在描述里记录要素的alias，参考下面的例子：

- 要素content：子账号-测试人于2016-08-25 19:42:03修改：招投标--业主通讯录--建设单位，招投标--业主通讯录--建设单位法人，招投标--业主通讯录--联系电话，
- 要素description: 1A,1B,1C,
- 文件content:用户于2016-08-26 15:33:12修改：初设(实施)--设计文件--附件上传--文件上传，上传：水文地质.rar。已使用11.345G空间，还剩85.234G空间
- 用户description: 241D1,

#### 管理员登录后，可以查看日志，根据项目编号检索，LogController接口：
1. 取得项目日志：/log/lists/{projectId}，GET
    - 根据项目id取得项目所有日志，包括要素和文件
    - 参数：
    - projectId:项目id
    - 返回：
    - OK：成功，data是日志列表，包含日志类中所有属性
    - EXIST：项目不存在

```
public class Log extends BaseEntity {

	/** 内容 */
	private String content;
	/** 描述 */
	private String description;
	/** 项目id */
	private Long projectId;
	/** 类型 */
	private Type type;

	/**
	 * 类型
 	 */
	public enum Type {
		// 项目要素
		ELEMENT,
		// 项目文件
		FILE
	}
}
```
### 六. 套餐详情
在当前套餐界面，显示套餐各项已使用/最大数量,各项是：
- 存储空间
- 上传流量
- 下载流量
- 子账号数
- 项目数
- 全景数

### 七. 地图定制图例
https://developers.google.cn/maps/documentation/javascript/adding-a-legend?hl=zh-cn
### 八. 前台gulp压缩html和js
### 讨论一下
-  帮助文档，点击左上角logo，显示帮助列表
-  反馈意见，审核







