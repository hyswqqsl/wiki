## 青清水利第九次迭代
### 涉及接口

[套餐接口设计](http://www.qingqingshuili.net:10001/soft/wiki/wikis/package) | 
[水位站接口设计](http://www.qingqingshuili.net:10001/soft/wiki/wikis/station)

### 一. 用户权限
用户分为simple(默认),identity(实名认证),company(企业认证)，用户一旦注册，即拥有simple角色，一旦完成实名认证即拥有identity角色，完成企业认证即拥有company角色。**每月运行定时任务**对认证进行验证是否过期，如果过期将认证状态改为过期，同时移除用户的identity角色或company角色。前台可以利用角色控制显示：
> 
1. 拥有identity角色的，才能购买套餐，才能对套餐升级和续费
2. 拥有identity角色的，才能购买测站和续费
3. 拥有identity角色的，才能进行企业认证

后台可以去除IsPersonalCertify和IsCompanyCertify两个标签，采用shiro用户角色验证。

### 二. 项目特殊权限
qxwz(千寻位置),bim(bim协同)这两个权限附加在项目上，每次进行这两种操作时，都应该先向后台发请求
判断，当前操作的项目是否拥有权限。
>
1. 判断是够有千寻连接权限：qxwz/isAllowQxwz,GET，后台有IsFindCM标签，感觉不用定义标签
2. 判断是够有bim协同权限：project/isAllowBim,GET，后台有IsBim标签，感觉不用定义标签

### 三. 套餐过期
套餐过期后，所有操作都不能进行，如建立项目、绑定子账号、编辑项目要素、上传文件、上传工程布置、地图上绘图、编辑水文站、不能上传外业勘测数据，这些涉及到的操作都要判断套餐是否过期，这些操作后台上都加上判断套餐未过期标签**IsExpire**，返回EXPIRE类型的消息。

### 四. 套餐限制
每种套餐都有限制，分为空间数、项目数、子账号数、全景数、千寻连接数，这些连接时都要判断是否超过限制，后台标签使用切面方式控制；由于前台的文件是由浏览器直接上传到阿里云的，因此文件上传、下载、删除需要先访问后台接口，判断是否允许上传，上传完文件再访问后台接口，告诉后台上传了多大的文件，让后台记录空间数变化；删除文件后也要访问后台接口，让后台记录空间数变化,同时记录日志；空间数只是限制阿里云空间大小，但是阿里云收费除了空间大小外，还有流量费，为了防止用户恶意操作浪费流量，还要限制流量使用，上传和下载流量最大是空间大小的10倍。
>
1. 空间数
    * 后台标签:IsAllowUploadFile,判断了空间数是否超过限制，上传流量是否超过限制，感觉不用定义标签,IsAllowDownloadFile，判断了下载流量是否超限，感觉不用定义标签
    * 是否允许上传接口:project/isAllowUpload GET,返回：ok 允许上传，no_allow 不允许上传
    * 记录上传文件接口:project/uploadFile POST,参数：projectId，fileName，fileSize,返回：ok:记录上传成功
    * 是否允许下载接口:project/isAllowDownload GET,返回：ok 允许下载，no_allow 不允许下载
    * 记录下载文件接口:project/downloadFile POST，参数：projectId, fileName，fileSize,返回：ok:记录上传成功
    * 记录删除文件接口:projec/deleteFile POST,参数：projectId，fileName, fileSize,返回：ok：记录删除成功
2. 项目数
    * project/new接口，加入判断用户是否可以新建项目，后台使用的是IsAllowCreateProject标签，感觉不用定义标签
3. 子账号数
    * user/invite接口，加入判断用户是否可以邀请子账号，后台使用的IsAllowCreateAccount标签，感觉不用定义标签
4. 全景数
    * interest/savePanorama接口，加入判断用户是否可以新建全景，后台使用IsPano标签，感觉不用定义标签
    * interest/savePanoramas接口，这是批量添加全景，下次迭代一个全景里可以添加多个场景图片，所以和单个全景添加一样处理
5. 千寻连接数
    * qxwz/getAccount接口，加入判断用户是否有权限连接千寻，连接数是否超过限制，后台使用IsFindCM标签，感觉不用定义标签

### 五. 后台标签
1. 综上所述，后台标签大部分只用一次，就不需要定义了
2. 只有IsExpire和IsExpired，两个要在很多方法上使用，所以需要定义成标签，面向切面编程
### 六. 关于日志
日志类型，分为项目要素操作日志，项目文件操作日志，项目操作(新建，修改，删除)日志，子账号(邀请，解绑)日志，前景操作日志(新建，修改，删除),本次迭代暂不考虑。
用户消息，用户购买，续费，升级套餐成功，被分享查看项目，被取消查看项目，被分享查看测站，被取消查看测站,本次迭代暂不考虑。
子账号消息，被邀请成为子账号，被解绑子账号，被分享(查看，编辑)项目，被取消查看或编辑项目。
本次迭代,本次迭代暂不考虑。





