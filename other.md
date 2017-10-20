## 其他操作接口设计
### ossController接口
>
1. 取得文件访问url：getFileUrl，GET
    * 返回的文件url有效期30分钟
    * 参数：key:文件路径,bucketName:bucket名
    * 返回：OK：内容是{url：访问地址}，如果文件不存在没有内容

### 全景和兴趣点接口,interestController
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


