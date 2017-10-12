## 分享接口设计
### 一. 项目分享设计
>
1. 多项目多用户共享：ProjectController中，POST,路径/project/share，参数：Long[] projectIds,欲协同的项目，示例[12,34,24];Long[] userIds，多用户，示例[12,34,24]；成功返回OK，失败返回FAIL,已分享的不再重复分享
2. 取消分享：ProjectController中，POST,路径/project/unShare，参数：Long prjectId，欲取消分享的项目，Long[] userIds，多用户，示例[12,34,24]

### 二. 项目协同设计
>
1. 多项目多人多编辑协同：ProjectController中，POST,路径/project/cooperate/edit，参数：Long[] projectIds,欲协同的项目，示例[12,34,24];Long[] userIds，多用户，示例[12,34,24]；types，多编辑权限，使用字符串方式，用逗号隔开，示例("VISIT_INVITE_ELEMENT", "VISIT_INVITE_FILE"),成功返回OK，失败返回FAIL,已分享的不再重复分享
2. 取消多人多编辑协同：ProjectController中，POST,路径/project/cooperate/unEdit，参数：Long prjectId，欲取消协同的项目，String types，使用字符串方式，用逗号隔开，示例("VISIT_INVITE_ELEMENT", "VISIT_INVITE_FILE")
3. 多项目多人查看协同：ProjectController中，POST,路径/project/cooperate/view，参数：Long[] projectIds,欲协同的项目，示例[12,34,24];Long[] userIds，多用户,示例[12,34,24]；types，成功返回OK，失败返回FAIL,已分享的不再重复分享
4. 取消多人查看协同：ProjectController中，POST,路径/project/cooperate/unView，参数：Long prjectId，欲取消协同的项目,Long[] userIds，多用户,示例[12,34,24]

### 三. 测站共享设计 StationController
>
1. 多测站多用户共享：/station/shares POST
    * 参数：String stationIds,欲分享的仪表，用逗号隔开，示例("12,34,24");String userId,分享的目标用户,用逗号隔开，示例("12,34,24")
    * OK,成功返回
2. 取消测站分享：/station/unShare POST
    * 参数：Long sensorId，欲取消分享的仪表，String userIds，用逗号隔开，示例("12,34,24")
3. 后台修改内容,station实体类添加share字段，在实体Station包下增加Share,ShareVisit类，和project分享的实体类Share,ShareVisit类似



