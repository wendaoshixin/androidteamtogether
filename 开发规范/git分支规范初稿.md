# Git分支规范



### 一、主要分支、命名规范

1. **master**

   主分支（受保护）

2. **develop**

   开发分支（受保护）

   

3. **release**

   预发布分支，如 **release_1.0.0**

   可选择的临时性分支，用于发布之前的细小改动

   

4. **dev\_版本号\_merge** 

   某版本的公共分支，如 **dev_1.0.0_merge**
   该分支用于版本迭代中的`公共资源`、`公用代码` 的整合

   

5. **version_版本号**

   某版本的所有需求的集成分支，如 **version_1.0.0**
   该分支用于版本迭代中的所有需求特性分支的集成与测试

   

6. **dev\_版本号\_fix\_merge**

   某版本线上问题修复分支，如**dev_1.0.0_fix_merge**

   

7. **version\_版本号\_fix\_merge**

   某版本线上问题修复测试分支，如**version_1.0.0_fix**

   

8. **业务功能性分支命名**

   dev\_版本号\_功能名称、dev\_版本号\_fix\_功能名称

   如**dev_1.0.0_account**、**dev_1.0.0_fix_account**

   

9. **核心功能性分支命名：**

   dev\_版本号\_功能，如**dev_player_1.0.0**

   

### 二、提交规范：Commit#Type

1. **add** 
   新功能（feature）
2. **fix** 
   修补bug
3. **docs** 
   文档（documentation）,如 修改README、注释
4. **style** 
   UI样式改动，如调整颜色、更改圆角、字体
5. **refactor** 
   重构
6. **test** 
   增加测试
7. **chore** 
   构建过程或辅助工具的变动，修改版本号、混淆配置



### 三、开发过程分支规范：

![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/2563cf54a8fa86498925bf7d7606a774ef54be5a/开发规范/picture/git开发分支示例.jpg)

 * 操作流程说明
   1. 新建项目代码仓库，从master分支拉出develop，并将项目初始代码至develop分支
   2. 从develop分支拉出dev\_版本号\_merge，并从dev\_版本号\_merge拉出version_版本号和release\_版本号
   3. 开发人员从dev\_版本号\_merge拉出负责的功能分支，如dev\_版本号\_feature，并在dev\_版本号\_feature进行开发
   4. 开发过程中，如用到公共资源，需从dev\_版本号\_merge将资源合并到dev\_版本号\_feature.
   5. 添加公共资源时，切换到dev\_版本号\_merge进行添加，再切换回dev\_版本号\_feature分支进行合并
   6. 功能开发完成，需要自测，自测后，合入到version_版本号分支集成测试
   7. 测试阶段，Bug修复应在dev\_版本号\_feature分支上进行，修复后继续合入到version_版本号分支提测
   8. 测试完成，App发布后，将version_版本号合入到develop分支和master分支，并分别标记TAG.
   9. 版本迭代完成



### 四、线上问题修复分支示例：

![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/2563cf54a8fa86498925bf7d7606a774ef54be5a/开发规范/picture/git线上修复示例.jpg)

- 操作流程说明

1. 从develop的TAG节点拉出dev\_版本号\_fix_merge，并从dev\_版本号\_fix_merge拉出version_\_版本号\_fix_
2. 从dev\_版本号\_fix_merge，拉出要修复的功能分支dev\_版本号\_fix_feature
3. 问题修复后，合入version_\_版本号\_fix，进行提测
4. 测试完成发布后，将version_\_版本号\_fix合入develop和master分支，并分别标记对应的TAG，如 DEV_TAG\_版本号\_fix
5. develop分支将修复的提交合入正在开发的dev\_版本号\_merge分支