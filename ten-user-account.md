<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [子账号号改为一对多接口设计](#%E5%AD%90%E8%B4%A6%E5%8F%B7%E5%8F%B7%E6%94%B9%E4%B8%BA%E4%B8%80%E5%AF%B9%E5%A4%9A%E6%8E%A5%E5%8F%A3%E8%AE%BE%E8%AE%A1)
    - [一. 子账号信息说明](#%E4%B8%80-%E5%AD%90%E8%B4%A6%E5%8F%B7%E4%BF%A1%E6%81%AF%E8%AF%B4%E6%98%8E)
    - [二. AccountController 子账号控制层](#%E4%BA%8C-accountcontroller-%E5%AD%90%E8%B4%A6%E5%8F%B7%E6%8E%A7%E5%88%B6%E5%B1%82)
    - [三. 用戶信息說明](#%E4%B8%89-%E7%94%A8%E6%88%B6%E4%BF%A1%E6%81%AF%E8%AA%AA%E6%98%8E)
    - [四. userController 用户控制层](#%E5%9B%9B-usercontroller-%E7%94%A8%E6%88%B7%E6%8E%A7%E5%88%B6%E5%B1%82)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 子账号号改为一对多接口设计
### 一. 子账号信息说明
1. 子账号没有认证流程，没有注册；子账号和用户之间改为一对多，由用户添加、编辑、删除，用户和子账号关系就是企业与员工的关系
2. 用户新建子账号时，首先发送短信通知子账号新建确认，子账号用短信回复确认，后台收到确认后，生成随机密码，再次发送短信通知子账号账号建立，子账号就可以登录了
3. 子账号和用户类似，也是在账号管理处上传头像，不用base64保存，头像由前台保存到阿里云，路径是qsl/account/{accountId}/avatar.jpg
4. 有验证码的不用再发手机号或邮箱！！
5. 子账号登录后，也可以编辑自己的资料
6. 去除注册时发送手机验证码getUpdateVerify，注册子账号register;修改登录流程，如果子账号状态不是CONFIRMED，不能登录
7. Message中4090: ACCOUNT_NOINFO子账号信息不全的错误码，改为4090: ACCOUNT_NO_CONFIRMED，子账号未确认

### 二. AccountController 子账号控制层
>
1. 修改密保手机发送验证码： /account/phone/getUpdateVerify,GET
    * 参数：phone:手机号
    * 返回：OK:发送成功
    * FAIL:手机号不合法
    * EXIST：手机号已被使用
2. 手机找回密码时发送验证码：/account/phone/getGetbackVerify,GET
    * 参数：phone:手机号
    * 返回：OK:发送成功
    * FIAL:手机号不合法
    * EXIST：账号不存在
3. web端登录发送验证码: /account/login/getLoginVerify,GET
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，如果手机号注册，发送手机验证码；如果是email，如果邮箱绑定，发送邮箱验证码
    * 参数：code:登录凭证
    * 返回：OK:发送成功,FAIL:手机号/邮箱不合法，EXIST：账号不存在
4. email绑定时发送验证码:/account/email/getBindVerify,GET
    * 参数：email:手机号
    * 返回：OK:发送成功,FAIL:邮箱不合法，EXIST：邮箱已存在
5. email找回密码时发送验证码：/account/email/getGetbackVerify,GET
    * 参数：email:邮箱
    * 返回：OK:发送成功,FAIL:email不合法，EXIST：邮箱不存在 
6. 手机找回密码: /account/phone/getbackPassord,GET
    * 忘记密码时找回密码 
    * 参数：verification:验证码;passord:重设的密码; 
    * 返回：OK:修改成功，INVALID:请重新获取验证码，FAIL:验证码错误
7. email找回密码:/account/email/getbackPassord，GET
    * 忘记密码时找回密码
    * 参数：verification:验证码;passord:重设的密码;   
    * 返回：OK:修改成功，INVALID:请重新获取验证码，FAIL:验证码错误 
8. 修改密码:在基本资料的修改密码处点击保存时调用， /account/updatePassword POST
    * 参数：oldPassword:当前密码，newPassword：新密码
    * 返回：OK:修改成功，UNKNOWN:当前密码错误
9. 修改密保手机：在基本资料的密保手机处点击保存时调用account/updatePhone POST
    * 参数：verification:验证码
    * 返回：OK:修改成功，INVALID:请重新获取验证码，FAIL:验证码错误 
10. 绑定邮箱：在基本资料里的绑定邮箱处点击保存时调用，/account/updateEmail POST
    * 参数：verification:验证码
    * 返回：OK:修改成功，INVALID:请重新获取验证码，FAIL:验证码错误 
11. 获取子账号信息：在基本资料的基本信息处显示，/account/getAccount GET
    * 参数：
    * 返回：OK(用户对象)，包含昵称、手机号，身份认证状态，企业认证状态，绑定微信的昵称，微信openid等
12. web端登录：/account/web/login POST
    * 子账号状态不是已确认CONFIRMED，不能登录，返回ACCOUNT_NO_CONFIRMED
    * 取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 参数：code：登录凭证，手机号或邮箱；password:密码；cookie：cookie
    * 返回：
        * OK(用户对象):登录成功；
        * FAIL：密码错误；
        * EXIST:用户不存在;
        * OTHER:需要验证码，
        * UNKNOWN:用户已锁定
        * ACCOUNT_NO_CONFIRMED:子账号未确认
13. 移动端登录：/account/phone/login POST
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码；如果是email，通过email取得用户，验证用户密码,微信第一次登录也是这个
    * 参数：code：登录凭证，手机号或邮箱；password:密码
    * 返回：
        * OK(用户对象):登录成功；
        * FAIL：密码错误；
        * EXIST:用户不存在,
        * UNKNOWN:用户已锁定
        * ACCOUNT_NO_CONFIRMED:子账号未确认 
14. web端验证码登录：改为 /account/web/loginByVerify POST 
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 参数：password:密码；verification:验证码
    * 返回：
        * OK(用户对象):登录成功；
        * FAIL：密码或验证码错误；
        * UNKNOWN:用户已锁定；
        * INVALID:请重新获取验证码
        * ACCOUNT_NO_CONFIRMED:子账号未确认
15. 判断输入的用户密码是否正确：/account/checkPassword,POST
    * 进入账号管理时，要求输入密码，保证安全
    * 参数：password:密码
    * 返回：OK:密码正确；FAIL：密码错误
    
###  三. 用戶信息說明
>
1. 用户注册时有昵称、手机号、密码，用户信息里动态添加属性有，身份认证状态，企业认证状态，绑定微信的昵称，微信openid，**当前套餐对象{套餐名，套餐状态}**
2. 用户可以绑定邮箱，邮箱验证码确认，认证后的邮箱可以登录，也可以发送验证码
3. 用户真实姓名在身份认证中产生，用户还没有通过身份认证时，用户真实姓名保证和昵称一致
4. 用户身份认证时，由用户提交认证，填写真实姓名和身份证号,自动审核时调用阿里云的身份认证接口，获得认证结果
5. 企业认证时，可以填写企业信息， 阿里云的身份认证接口，获得认证结果,并发送短信，如果绑定邮箱，同时发送邮件
6. 用户头像不用base64保存，头像由前台保存到阿里云，路径是qqsl/user/{userId}/avatar.jpg，头像路径不用保存到数据库；同样的子账号头像也有前台保存到阿里云，路径是qqsl/account/{accountId}/avatar.jpg
7. 身份证正反面，企业营业执照也是由前台保存到阿里云，路径是qqsl/user/{userid}/identity1.jpg,qqsl/user/{userid}/identity2.jpg,qqsl/user/{userid}/licence.jpg
8. 动态权限问题：用户设定simple,identity,company,hydrology角色，前台方便显示界面；管理员设定simple,super角色；子账号设定simple角色
9. 有验证码的不用再发手机号或邮箱！！ 
10. 邀請子账号/account/invite改为新建子账号/account/create，解绑子账号/account/unbind改为删除子账号/account/delete,增加/account/update,/account/updatePhone

### 四. userController 用户控制层
>
1.  注册时发送手机验证码:/user/phone/getRegistVerify
    * 参数：phone:手机号
    * 返回：OK:发送成功,FIAL:手机号不合法，EXIST：手机号已被使用
2.  修改密保手机发送验证码：/user/phone/getUpdateVerify
    * 参数：phone:手机号
    * 返回：OK:发送成功,FIAL:手机号不合法，EXIST：手机号已被使用
3. 手机找回密码时发送验证码：/user/phone/getGetbackVerify
    * 参数：phone:手机号
    * 返回：OK:发送成功,FIAL:手机号不合法，EXIST：账号不存在
4. web端登录发送验证码: /user/login/getLoginVerify
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，如果手机号注册，发送手机验证码；如果是email，如果邮箱绑定，发送邮箱验证码
    * 参数：code:登录凭证
    * 返回：OK:发送成功,FAIL:手机号/邮箱不合法，EXIST：账号不存在
5. email绑定时发送验证码:/user/email/getBindVerify
    * 参数：email:手机号
    * 返回：OK:发送成功,FAIL:邮箱不合法，EXIST：邮箱已存在
6. email找回密码时发送验证码：/user/email/getGetbackVerify
    * 参数：email:邮箱
    * 返回：OK:发送成功,FAIL:email不合法，EXIST：邮箱不存在
7. 注册用户：/user/register
   * 水利云用户注册时填写的userName是昵称，都是英文字符和数字，用于在未实名时显示用户名称，在新建项目时使用昵称建立项目编号
   * 长度不超过36字符
    * 参数：userName:昵称，password:密码，验证码：verification:验证码
    * 返回：OK:注册成功，INVALID:请重新获取验证码，FAIL:验证码错误  
8. 手机找回密码:/user/phone/getbackPassord，忘记密码时找回密码
    * 参数：verification:验证码;passord:重设的密码; 
    * 返回：OK:修改成功，INVALID:请重新获取验证码，FAIL:验证码错误
9. email找回密码:/user/email/getbackPassord，忘记密码时找回密码
    * 参数：verification:验证码;passord:重设的密码;   
    * 返回：OK:修改成功，INVALID:请重新获取验证码，FAIL:验证码错误 
10. 修改密码:在基本资料的修改密码处点击保存时调用， /user/updatePassword POST
    * 参数：oldPassword:当前密码，newPassword：新密码
    * 返回：OK:修改成功，UNKNOWN:当前密码错误
11. 修改密保手机：在基本资料的密保手机处点击保存时调用， /user/updatePhone POST
    * 参数：verification:验证码
    * 返回：OK:修改成功，INVALID:请重新获取验证码，FAIL:验证码错误 
12. 绑定邮箱：在基本资料里的绑定邮箱处点击保存时调用，/user/updateEmail POST
    * 参数：verification:验证码
    * 返回：OK:修改成功，INVALID:请重新获取验证码，FAIL:验证码错误 
13. 获取用户信息：在基本资料的基本信息处显示，/user/getUser GET
    * 参数：
    * 返回：OK(用户对象)，包含昵称、手机号，身份认证状态，企业认证状态，绑定微信的昵称，微信openid等
14. 获取用户列表：/user/getUsers GET
    * 参数：
    * 返回：OK(用户对象)，包含昵称、手机号，身份认证状态，企业认证状态，绑定微信的昵称，微信openid等
15. 管理员获取用户列表：/user/admin/getUsers GET
    * 参数：
    * 返回：OK(用户对象)，包含昵称、手机号，身份认证状态，企业认证状态，绑定微信的昵称，微信openid等
16. web端登录：/user/web/login POST
    * 取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 判断用户角色，只有拥有user:simple角色的用户才能登录
    * 参数：
        * code：登录凭证，手机号或邮箱
        * password:密码
        * cookie：cookie
    * 返回：
        * OK(用户对象):登录成功
        * FAIL：密码错误
        * EXIST:用户不存在
        * OTHER:需要验证码
        * UNKNOWN:用户已锁定
        * UNAUTHORIZED： 不是水利云用户，无访问权限
17. 移动端登录：/user/phone/login POST
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码；如果是email，通过email取得用户，验证用户密码,微信第一次登录也是这个
    * 判断用户角色，只有拥有user:simple角色的用户才能登录
    * 参数：
        * code：登录凭证，手机号或邮箱
        * password:密码
    * 返回：
		* OK:登录成功,返回{name:xx,phone:xx, qqslBucket:xxx, qqslimageBucket:xxx}
        * FAIL：密码错误
        * EXIST:用户不存在
        * UNKNOWN:用户已锁定
        * UNAUTHORIZED： 不是水利云用户，无访问权限        
18. 微信公众号登录：/user/wechat/login POST
    * 微信公众号登录，微信号和用户站好会绑定
    * 判断用户角色，只有拥有user:simple角色的用户才能登录
    * 参数：
        * code：登录凭证，手机号或邮箱
        * password:密码
    * 返回：
        * OK(用户对象):登录成功
        * FAIL：密码错误；EXIST:用户不存在
        * UNKNOWN:用户已锁定
        * UNAUTHORIZED： 不是水利云用户，无访问权限        
19. 微信公众号自动登录：/user/wechat/autoLogin POST
    * 参数：code：微信公众号回传的标识码
    * 返回：OK(用户对象):登录成功；EXIST:用户不存在
20. web端验证码登录：/user/web/loginByVerify POST
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 判断用户角色，只有拥有user:simple角色的用户才能登录
    * 参数：password:密码；verification:验证码
    * 返回：
        * OK(用户对象):登录成功
        * FAIL：密码或验证码错误
        * UNKNOWN:用户已锁定
        * INVALID:请重新获取验证码
        * UNAUTHORIZED： 不是水利云用户，无访问权限        
21. 判断输入的用户密码是否正确：/user/checkPassword,POST
    * 进入账号管理时，要求输入密码，保证安全
    * 参数：password:密码
    * 返回：OK:密码正确；FAIL：密码错误
22. 新建子账号：/account/create,POST
    * 这个接口是用户新建子账号，只是在数据建立子账号记录，但是没有密码，也不能登录
    * 添加子账号记录后，密码为空，状态是AWAITING
    * 同时发送通知短信，告诉子账号有企业正在添加为子账号，需要确认，发送Y同意，发送N拒绝;同时把短信记录到note表，子账号回复这条短信后，阿里云短信系统会调用接口把回复内容发送到接口，如果note回复为空，并且回复内容是Y(大小写均可)或N，那么根据收到的短信号查询note表，根据手机号找到子账号
    * 如果回复Y，查询目前子账号中已确认的有无这个手机号，如果有，给当前手机号回复：您已经是“某某”企业的子账号，不能重复添加到其他企业，然后退出；如果没有，把子账号状态改为CONFIRMED，生成随机密码，发送账号新建成功短信
    * 如果回复N，直接删除子账号，同时在userMessage中加一条消息：您添加的子账号(130xxxxxxxx)已拒绝邀请
    * 如果子账号超过24小时未回复，编写一个线程删除这样的记录，同时在userMessage添加一条消息：您添加的子账号(130xxxxxxxx)超过24小时未回复，已被删除
    * 参数：name,phone,department,remark
    * 返回：
        * OK:新建成功
        * DATA_EXIST:子账号已存在，不能重复添加。子账号手机号已存在，不能重复添加添加相同的手机号  
23. 用户编辑子账号：/user/account/update,POST
      * 参数：id(子账号id),name, department, remark
      * 返回：
        * OK:编辑成功
        * FAIL:子账号不属于当前用户,session中的user和参数中的userId不一致，这个处理防止其他用户误操作
24. 删除子账号：/user/account/delete,POST
    * 删除子账号，相当于企业删除员工一样；删除前循环监测所有项目中有无协同内容，删除之
    * 参数：id(子账号id)
    * 返回：     
        * OK:删除成功
        * FAIL: 子账号不属于该企业，不能删除
25. 修改用户昵称，/user/updateUserName,POST        
   * 水利云用户注册时填写的userName是昵称，都是英文字符和数字，用于在未实名时显示用户名称，在新建项目时使用昵称建立项目编号
   * 长度不超过36字符
   * 参数：userName:xxx
   * 返回：     
       * OK:修改成功
       * PARAMETER_ERROR: 用户名包含中文和特殊字符,或长度超过36个字符
26. 修改安布雷拉水文监测用户公司名称,/user/abll/updateUserName, POST    
   * 安布雷拉水文监测用户注册时填写的userName是公司名称，数字英文中文均可，用在界面显示公司名称
   * 长度不超过36字符
   * 保证有user:abll角色
   * 参数：userName:xxx
   * 返回：     
       * OK:修改成功
       * PARAMETER_ERROR: 用户名包含特殊字符,或长度超过36个字符
       * UNAUTHORIZED: 不是安布雷拉水文监测用户
27. 安布雷拉水文监测用户注册，/user/abll/register, POST
   * 安布雷拉水文监测用户注册时填写的userName是公司名称，英文中文均可，用在界面显示公司名称
   * 注册完拥有user:abll角色
   * 长度不超过36字符
   * 参数：
       * userName:公司名称
       * password:密码
       * verification:验证码
   * 返回：
       * OK:注册成功
       * INVALID:请重新获取验证码
       * FAIL:验证码错误     
28. 安布雷拉水文监测用户web端登录：/user/abll/web/login POST
    * 取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 判断用户角色，只有拥有user:abll角色的用户才能登录
    * 参数：
        * code：登录凭证，手机号或邮箱
        * password:密码
        * cookie：cookie
    * 返回：
        * OK(用户对象):登录成功
        * FAIL：密码错误
        * EXIST:用户不存在
        * OTHER:需要验证码
        * UNKNOWN:用户已锁定
        * UNAUTHORIZED： 不是安布雷拉用户，无访问权限
29. web端验证码登录：/user/abll/web/loginByVerify POST
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 判断用户角色，只有拥有user:abll角色的用户才能登录
    * 参数：password:密码；verification:验证码
    * 返回：
        * OK(用户对象):登录成功
        * FAIL：密码或验证码错误
        * UNKNOWN:用户已锁定
        * INVALID:请重新获取验证码
        * UNAUTHORIZED： 不是安布雷拉用户，无访问权限                           
       
```
    /** 真实姓名 */
    private String name;
    /** 密码 */
    private String password;
    /** 联系电话 */
    private String phone;
    /** 邮箱 */
    private String email;
    /** 所属部门 */
    private String department;
    /** 备注 */
    private String remark;
    /** 是否锁定 */
    private Boolean isLocked;
    /** 锁定日期 */
    private Date lockedDate;
    /** 最后登录日期 */
    private Date loginDate;
    /** 最后登录IP */
    private String loginIp;
    /** 连续登录失败次数 */
    private Integer loginFailureCount;
    /** 用户角色 */
    private String roles;
    /** 子账号的登录方式 */
    private String loginType;
    /** 子账号状态 */
    private Status status;

    /** 所属user */
    private User> user;
    /** 子账号消息 */
    private List<AccountMessage> accountMessages = new ArrayList<>();

    /** 子账号状态 */
    public enum Status {
        // 待确认
        AWAITING,
        // 已确认
        CONFIRMED
    }
```


```
	/**
	 * 用户消息类型
	 */
	public enum Type {
		/** 购买套餐，包含购买，续费，升级 */
		BUY_PACKAGE,
		/** 购买测站，包含购买，续费 */
		BUY_STATION,
		/** 分享项目 */
		SHARE_PROJECT,
		/** 分享测站 */
		SHARE_STATION,
		/** 子账号 */
		ACCOUNT,
		/** 认证 */
		CERTIFY,
		/** 反馈回复 */
		FEEDBACK
	}
  /**
   * 子账号消息类型
   */
  public enum Type {
      /** 项目协同 */
      COOPERATE_PROJECT,
      /** 反馈回复 */p
      FEEDBACK,
      /** 测站协同 */
      COOPERATE_STATION
  }
```



