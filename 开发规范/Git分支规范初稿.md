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



### 三、开发过程分支规范、示例：

![补丁分支](/Users/lm104/Desktop/补丁分支.png)





### 四、线上问题修复分支规范、示例：

![补丁分支](/Users/lm104/Desktop/补丁分支.png)