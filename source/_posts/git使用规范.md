---
title: GIT使用规范
date: 2020-04-19 21:51:00
tags:
    - 规范
    - Git
categories:
    - 基础
    - 规范
---

- **每日进行开发工作之前必须更新代码，下班时提交代码。**
- **各员工需牢记各自的账户和密码，一定不得向他人透漏，严禁使用他人账户进行GIT各项操作**
- **文件提交时要求必须提交注释，注明相关修改信息，日志信息描述的越详细越好，让项目组其他成员在看到标注后不用详细看代码就能了解你所做的修改**
- **对提交的信息采用明晰的标注**

```ini
标注方法
    +)表示增加了功能 
    *) 表示对某些功能进行了更改 
    -) 表示删除了文件，或者对某些功能进行了裁剪，删除，屏蔽。 
    b) 表示修正了具体的某个bug
```

```ini
demo
+)  实现需求T2331
*)  优化FooUtil方法
-)  删除多余文件
b)  修复Bug1213
```
- **文件提交前必须先update代码再commit,如果遇到冲突,严禁删除原版本后直接上传新版本**
- **文件提交前必须检查是否有语法错误,php文件要使用php -l ［文件名］通过检测**
- **提前宣布修改计划。当你计划进行修改，需要影响到git里的许多文件时，先通过邮件或者当面通知其他开发者 例如，修改底层数据库模块时，有可能影响到业务逻辑层调用数据库模块的地方。这样其他开发者会有准备，也会对修改提出意见和建议**
- **原子性提交代码(即一个小功能提交一次)**
- **提交前请自行进行单元测试,严禁上传未测试的代码!**



