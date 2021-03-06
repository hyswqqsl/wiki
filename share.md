<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [分享接口设计](#%E5%88%86%E4%BA%AB%E6%8E%A5%E5%8F%A3%E8%AE%BE%E8%AE%A1)
  - [一. 项目分享设计](#%E4%B8%80-%E9%A1%B9%E7%9B%AE%E5%88%86%E4%BA%AB%E8%AE%BE%E8%AE%A1)
  - [二. 项目协同设计,ProjectController](#%E4%BA%8C-%E9%A1%B9%E7%9B%AE%E5%8D%8F%E5%90%8C%E8%AE%BE%E8%AE%A1projectcontroller)
  - [三. 测站共享设计 StationController](#%E4%B8%89-%E6%B5%8B%E7%AB%99%E5%85%B1%E4%BA%AB%E8%AE%BE%E8%AE%A1-stationcontroller)
  - [四. 测站协同设计 StationController](#%E5%9B%9B-%E6%B5%8B%E7%AB%99%E5%8D%8F%E5%90%8C%E8%AE%BE%E8%AE%A1-stationcontroller)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 分享接口设计
### 一. 项目分享设计
>
1. 多项目多用户共享：ProjectController中，POST,路径/project/share，参数：Long[] projectIds,欲协同的项目，示例[12,34,24];Long[] userIds，多用户，示例[12,34,24]；成功返回OK，失败返回FAIL,已分享的不再重复分享
2. 取消分享：ProjectController中，POST,路径/project/unShare，参数：Long prjectId，欲取消分享的项目，Long[] userIds，多用户，示例[12,34,24]

### 二. 项目协同设计,ProjectController
>
1. 将一个项目的多个权限协同给一个子账号:POST,/project/cooperateMul
    已协同的不再重复 
    * 参数：
        * projectId:欲协同的项目
        * types:多编辑权限，使用字符串方式，用逗号隔开，示例("VISIT_INVITE_ELEMENT", "VISIT_INVITE_FILE")
        * accountId:欲协同的子账号
    *返回：
        * 成功返回OK，
        * 如果不是自己的项目，返回DATA_REFUSE
        * 失败返回FAIL
2. 将多个项目的一个权限协同给一个子账号:POST,/project/coorperateSim
    已协同的不再重复
    * 参数：
        * projectIds:欲协同的多个项目
        * type:编辑权限，传递字符串
        * accountId:欲协同的子账号
    *  返回:
        * 成功返回OK，
        * 如果不是自己的项目，返回DATA_REFUSE 
        * 失败返回FAIL
3. 取消子账号编辑协同：POST,/project/unCooperate
    * 参数：
        * projectId，欲取消协同的项目
        * types，使用字符串方式，用逗号隔开，示例("VISIT_INVITE_ELEMENT", "VISIT_INVITE_FILE")
   *  返回:
        * 成功返回OK
        * 如果不是自己的项目，返回DATA_REFUSE 
        * 失败返回FAIL
4. 取消子账号查看权限：POST,/project/unView
    * 参数：
        * prjectId，欲取消协同的项目
        * accountIds，多个子账号id,示例[12,34,24]
      *  返回:
        * 成功返回OK
        * 如果不是自己的项目，返回DATA_REFUSE 
        * 失败返回FAIL 


### 三. 测站共享设计 StationController
>
1. 多测站多用户共享：/station/shares POST
    * 参数：String userIds,分享的目标用户,用逗号隔开，示例("12,34,24"),String stationIds,欲分享的仪表，用逗号隔开，示例("12,34,24");
    * OK,成功返回
2. 取消测站分享：/station/unShare POST
    * 参数：Long stationId，欲取消分享的仪表，String userIds，用逗号隔开，示例("12,34,24")
3. 后台修改内容,station实体类添加share字段，在实体Station包下增加Share,ShareVisit类，和project分享的实体类Share,ShareVisit类似

### 四. 测站协同设计 StationController
>
1. 将多个测站协同给多个子账号:POST,/station/cooperateMul
    测站协同目前只需要查看即可，已协同的不再重复 
    * 参数：
        * stationIds:欲协同的多个测站,示例("12,34,24")
        * accountIds:欲协同的多个子账号,示例("12,34,24")
    *  返回:
        * 成功返回OK，
        * 如果不是自己的测站，返回DATA_REFUSE 
        * 失败返回FAIL
2. 取消测站查看协同：POST,/station/unCooperate
    * 参数：
        * stationId，欲取消协同的测站
        * accountIds，欲取消查看协同的多个子账号,示例("12,34,24")
   *  返回:
        * 成功返回OK
        * 如果不是自己的测站，返回DATA_REFUSE 
        * 失败返回FAIL   
3. 子账号取得测站列表：GET, /station/account/lists
    * 参数：无     
    *  返回: 
        * 成功返回OK，返回测站列表，包含测站所有数据

