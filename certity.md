

## 认证接口设计


<div align="center">
   <img src="http://112.124.104.190:10001/soft/wiki/uploads/bccd50cf206ac37ef851ea03cadda633/certify.png" width="680px" />
</div>

###  一. 用户信息说明
>
1. 用户注册时有昵称、手机号、密码，用户信息里动态添加属性有，身份认证状态，企业认证状态，绑定微信的昵称，微信openid，**当前套餐对象{套餐名，套餐状态} ??**
2. 用户可以绑定邮箱，邮箱验证码确认，认证后的邮箱可以登录，也可以发送验证码
3. 用户真实姓名在身份认证中产生，用户还没有通过身份认证时，用户真实姓名保证和昵称一致
4. 用户身份认证时，由用户提交认证，填写真实姓名和身份证号,自动审核时调用阿里云的身份认证接口，获得认证结果
5. 企业认证时，可以填写企业信息， 阿里云的身份认证接口，获得认证结果,并发送短信，如果绑定邮箱，同时发送邮件
6. 用户头像不用base64保存，头像由前台保存到阿里云，路径是qqsl/user/{userid}/avatar.jpg，头像路径不用保存到数据库
7. 身份证正反面，企业营业执照也是由前台保存到阿里云，路径是qqsl/user/{userid}/identity1.jpg,qqsl/user/{userid}/identity2.jpg,qqsl/user/{userid}/licence.jpg
8. 动态权限问题：用户设定simple,identity,company,hydrology角色，前台方便显示界面；管理员设定simple,super角色；子账号设定simple角色
9. 有验证码的不用再发手机号或邮箱！！

### 二. userController 用户控制层
>
1.  注册时发送手机验证码:/user/phone/getRegistVerify
    * 参数：phone:手机号
    * 返回：OK:发送成功,FIAL:手机号不合法，EXIST：手机号已被使用
2.  修改密保手机发送验证码：/user/phone/getUpdateVeridy
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
    * 参数：userName:昵称，password:密码，验证码：verification:验证码
    * 返回：OK:注册成功，INVALID:验证码过期，FAIL:验证码错误  
8. 手机找回密码:/user/phone/getbackPassord，忘记密码时找回密码
    * 参数：verification:验证码;passord:重设的密码; 
    * 返回：OK:修改成功，INVALID:验证码过期，FAIL:验证码错误
9. email找回密码:/user/email/getbackPassord，忘记密码时找回密码
    * 参数：verification:验证码;passord:重设的密码;   
    * 返回：OK:修改成功，INVALID:验证码过期，FAIL:验证码错误 
10. 修改密码:在基本资料的修改密码处点击保存时调用， /user/updatePassword POST
    * 参数：oldPassword:当前密码，newPassword：新密码
    * 返回：OK:修改成功，UNKNOWN:当前密码错误
11. 修改密保手机：在基本资料的密保手机处点击保存时调用， /user/updatePhone POST
    * 参数：verification:验证码
    * 返回：OK:修改成功，INVALID:验证码过期，FAIL:验证码错误 
12. 绑定邮箱：在基本资料里的绑定邮箱处点击保存时调用，/user/updateEmail POST
    * 参数：verification:验证码
    * 返回：OK:修改成功，INVALID:验证码过期，FAIL:验证码错误 
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
    * 参数：code：登录凭证，手机号或邮箱；password:密码；cookie：cookie
    * 返回：OK(用户对象):登录成功；FAIL：密码错误；EXIST:用户不存在;OTHER:需要验证码，UNKNOWN:用户已锁定
17. 移动端登录：/user/phone/login POST
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码；如果是email，通过email取得用户，验证用户密码,微信第一次登录也是这个
    * 参数：code：登录凭证，手机号或邮箱；password:密码
    * 返回：OK(用户对象):登录成功；FAIL：密码错误；EXIST:用户不存在,UNKNOWN:用户已锁定
18. 微信公众号登录：/user/wechat/login POST
    * 微信公众号登录，微信号和用户站好会绑定
    * 参数：code：登录凭证，手机号或邮箱；password:密码
    * 返回：OK(用户对象):登录成功；FAIL：密码错误；EXIST:用户不存在,UNKNOWN:用户已锁定
19. 微信公众号自动登录：/user/wechat/autoLogin POST
    * 参数：code：微信公众号回传的标识码
    * 返回：OK(用户对象):登录成功；EXIST:用户不存在
20.web端验证码登录：/user/web/loginByVerify POST
    * 用户登录支持手机号和邮箱两种方式，如果登录凭证是手机号，通过手机号取得用户，验证用户密码，cookie；如果是email，通过email取得用户，验证用户密码，cookie
    * 参数：password:密码；verification:验证码
    * 返回：OK(用户对象):登录成功；FAIL：密码或验证码错误；UNKNOWN:用户已锁定；INVALID:验证码过期
21. 判断输入的用户密码是否正确：/user/checkPassword,POST
    * 进入账号管理时，要求输入密码，保证安全
    * 参数：password:密码
    * 返回：OK:密码正确；FAIL：密码错误
 
### 三. certifyController 认证控制层
>
1. 个人认证处理流程：从阿里云读取身份证正反面图片，转换成base64，得到姓名，身份证号和过期时间，和认证对象里的对比一致性;查询认证对象，将name、identityId传递到阿里云判断真实姓名和身份证号一致性；如果认证通过，将认证对象的认证状态改为通过,不通过，经认证对象改为不通过。使用定时任务，每周执行两次，循环对认证中的对象执行此认证逻辑
2. 企业认证处理流程：从阿里云读取企业营业执照照片，转换成base64，得到法人姓名，营业执照号码和过期时间，和认证对象里的对比一致性;查询认证对象，将法人姓名、营业执照号传递到阿里云判断法人和营业执照合法性；如果认证通过，将认证对象的认证状态改为通过,不通过，经认证对象改为不通过。使用定时任务，每周执行两次，循环对认证中的对象执行此认证逻辑
3. 管理员登陆后可以触发个人或企业认证流程，以上认证结果通过短信、邮件和用户信息(平台内userMessage)通知用户
4. 身份认证提交：在实名认证处，/certify/personalCertify POST
    * 身份证正反面图片由前台上传到阿里云
    * 参数：name(string)：真实姓名，identityId(string):身份证号
    * 返回：OK:提交成功，FAIL：提交失败，EXIST：已通过认证，不可再次提交，OHTER：营业执照已认证
5. 获取用户实名认证信息:/certify/getPersonalCertify,GET
    * 参数：
    * 返回：OK(实名认证信息)，如果没认证过，返回对象的状态是未认证CertifyStatus.UNAUTHEN
6. 企业认证提交：在企业认证处，/certify/companyCertify, POST
    * 企业营业执照由前台上传到阿里云
    * 参数：legal(string)：法人姓名，companyName:企业名, companyAddress：企业地址，companyPhone：企业电话,companylicence(string):营业执照号码
    * 返回：OK:提交成功，FAIL：提交失败，EXIST：已通过认证，不可再次提交；NO_ALLOW:没有通过实名认证，不允许企业认证,OHTER：营业执照已认证
7. 获取用户企业认证信息：/certify/getCompanyCertify, GET
    * 参数：
    * 返回：OK(实名认证信息)，如果没认证过，返回状态是未认证
8. 管理员获取认证列表：/certify/admin/lists,GET
    * 参数：
    * 返回：OK(实名认证列表)
9. 管理员获取认证信息：/certify/admin/certify/{id}, GET
    * 参数：id：认证对象id
    * 返回：OK(实名认证信息)，如果没认证过，返回状态是未认证；EXIST:认证对象不存在
9. 管理员触发认证流程：/certify/admin/startCertify, POST
    * 参数：
    * 返回：OK：认证完成



