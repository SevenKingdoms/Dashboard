## Meeting Record

`2018/5/15 iter 3`

会议目标：开发人员汇报项目进度，确定API文档和数据库设计

### 议程

1. 汇报项目进度
2. 讨论API文档和数据库设计中的不足
3. 状态模型
4. 测试代码

### 结果

1. 目前编码进度

   已经完成小程序端和PC端的登录注册功能，API接口测试存在一些问题，导致登录注册出现bug

   结合用户的习惯，小程序端考虑采用自动注册，用户自主完善信息

   商家管理端由超级管理员给定商家账号密码，商家有修改账号密码的权限

   后端人员检查接口问题并修复

2. API文档和数据库设计


   根据领域模型和活动图，完善接下来的所有API设计，数据库设计遵循一定原则，尽量避免后期修改

3. 测试

   * 小程序端：小程序开发工具测试
   * 商家管理端：js脚本测试
   * 后端：postMan接口测试

4. 接下来几周任务

   * 按照迭代3的要求完善功能并测试
   * 设计好所有的API文档，把数据库建好
   * 尽可能多的实现程序功能，避免在期末考时爆炸