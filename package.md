## 套餐接口设计
### 套餐过期
套餐过期后，所有操作都不能进行，如建立项目、绑定子账号、编辑项目要素、上传文件、上传工程布置、地图上绘图、编辑水文站、不能上传外业勘测数据，这些涉及到的操作都要判断套餐是否过期，这些操作后台上都加上判断套餐未过期标签IsExpire，返回EXPIRE类型的消息；前台根据用户下的套餐对象状态，也进行限制，不用到后台验证，提高效率，涉及到的接口有：
>
1. 建立项目，project/new POST
2. 编辑项目，project/update POST
3. 向项目的业主和负责人发送短信,project/sendMessage POST
4. 企业间分享项目，project/share POST
5. 将一个项目的多个权限分享给一个子账号,project/cooperateMul POST
6. 将多个项目的一个权限分享给一个子账号,project/cooperateSim POST
7. 向项目的业主和负责人发送短信,project/sendMessage POST
8. 是否允许上传接口:project/isAllowUpload GET,返回：ok 允许上传，no_allow 不允许上传
9. 保存单元下的复合单元列表,element/saveElementGroup POST
10. 添加要素数据,element/makeElementDataGroup POST
11. **用户邀请子账号,user/invite GET ?? 应为POST**
12. 解除公众号与水利云账号的绑定,user/unbind POST
13. 新建坐标线面,地图上绘图,field/createShape POST
14. 编辑坐标线面,地图上绘图,field/editShape POST
15. 新建建築物，地图上绘图,field/createBuild POST
16. 编辑建筑物,地图上绘图,field/editBuild POST
17. 移动端外业保存,移动端上传外业勘测数据，field/saveField POST
18. **坐标文件上传，oss/coordinateFile POST ?? 可以改到field/coordinateFile**
19. 添加仪表，station/addSensor POST
20. 添加摄像头，station/addCamera POST
21. 编辑仪表，station/editSensor POST
22. 编辑摄像头，station/editCamera POST
23. 编辑测站，station/editStation POST
24. 上传河道模型和水位流量关系曲线, station/uploadModel POST
25. 编辑测站参数，station/editParameter POST
26. 由于项目文件前台是直接上传至阿里云的，因此前台在上传前，先判断用户下套餐对象状态，如果是过期，拒绝上传；后台关于文件允许上传接口还是需要判断过期
27. 是否允许上传接口:project/isAllowUpload GET,返回：ok 允许上传，no_allow 不允许上传
28. 千寻连接接口，qxwz/getAccount接口
29. 是否有千寻连接权限：qxwz/isAllowQxwz,GET
30. 是否有bim协同权限：project/isAllowBim,GET


### 套餐限制

每种套餐都有限制，分为空间数、项目数、子账号数、全景数、千寻连接数，这些连接时都要判断是否超过限制，后台标签使用切面方式控制；由于前台的文件是由浏览器直接上传到阿里云的，因此文件上传、下载、删除需要先访问后台接口，判断是否允许上传，上传完文件再访问后台接口，告诉后台上传了多大的文件，让后台记录空间数变化；删除文件后也要访问后台接口，让后台记录空间数变化,同时记录日志；空间数只是限制阿里云空间大小，但是阿里云收费除了空间大小外，还有流量费，为了防止用户恶意操作浪费流量，还要限制流量使用，上传和下载流量最大是空间大小的10倍。

>
1. **向后台报告上传文件信息，project/reportUploadFileInfo POST ??**
    * 根据文件大小更改空间使用情况，根据文件名记录上传日志(本迭代暂不考虑)
    * 参数：projectId,项目id，fileNames,上传的一批文件名，多个用‘,’隔开，fileSize，上传的一批文件总大小
    * 返回：OK,附加数据是{fileSize:文件大小}，将文件大小回传给前台，FAIL，参数错误
2. **向后台报告下载文件信息，project/reportDownloadFileInfo POST ??**
    * 根据文件大小更改空间使用情况，根据文件名记录下载日志(本迭代暂不考虑)
    * 参数：projectId,项目id，fileNames,上传的一批文件名，多个用‘,’隔开，fileSize，上传的一批文件总大小
    * 返回：OK,附加数据是{fileSize:文件大小}，将文件大小回传给前台，FAIL，参数错误 
3. 是否允许上传接口:project/isAllowUpload GET
    * 不仅要判断空间数是否超限，判断上传流量是否超限，还要判断套餐是否过期，只要符合以上任一条件，即返回no_allow
    * 返回：ok 允许上传，no_allow 不允许上传
4. 是否允许下载接口:project/isAllowDownload GET
    * 判断上传流量是否超限
    * 返回：ok 允许下载，no_allow 不允许下载
5. 新建项目，project/new接口，加入判断项目数超限
6. 子账号数，user/invite接口，加入判断子账号数超限
7. 全景数，interest/savePanorama接口，加入判断全景数超限
8. 批量添加全景，interest/savePanoramas接口，加入判断全景数超限
9. 千寻连接接口，qxwz/getAccount接口，加入判断套餐是否过期，是否有千寻连接权限，判断连接数是否超限

