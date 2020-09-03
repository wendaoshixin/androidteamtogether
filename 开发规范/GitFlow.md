# GitFlow开发流程规范



#### GitFlow流程介绍

###### master:

- 主分支 , 产品的功能全部实现后 , 最终在master分支对外发布
- 该分支为只读唯一分支 , 只能从其他分支(release/hotfix)合并 , 不能在此分支修改
- 另外所有在master分支的推送应该打标签做记录,方便追溯
- 例如release合并到master , 或hotfix合并到master

###### develop:

- 主开发分支 , 基于master分支克隆
- 包含所有要发布到下一个release的代码
- 该分支为只读唯一分支 , 只能从其他分支合并
- feature功能分支完成 , 合并到develop(不推送)
- develop拉取release分支 , 提测
- release/hotfix 分支上线完毕 , 合并到develop并推送

###### feature:

- 功能开发分支 , 基于develop分支克隆 , 主要用于新需求新功能的开发
- 功能开发完毕后合到develop分支
- feature分支可同时存在多个 , 用于团队中多个功能同时开发 , 属于临时分支 , 功能完成后可选删除

###### release:

- 发布前分支 , 基于feature分支合并到develop之后  , 从develop分支克隆
- 主要用于提交给测试人员进行功能测试 , 测试过程中发现的BUG在本分支进行修复 , 修复完成上线后合并到develop/master分支并推送(完成功能) , 打Tag
- 属于临时分支 , 功能上线后可选删除

###### hotfix:

- 补丁分支 , 基于master分支克隆 , 主要用于对线上的版本进行BUG修复
- 修复完毕后合并到develop/master分支并推送 , 打Tag
- 属于临时分支 , 补丁修复上线后可选删除
- 所有hotfix分支的修改会进入到下一个release

#### 工作流程

示例图（来源简书）：

![](/Users/lm233/Downloads/13183199-b45a2835bcd11234.webp)



###### 新功能开发：

* 1.从develop检出feature_version_name（例）分支，进行新功能开发

* 2.功能开发完成request merge，请求合并到develop（不严格要求，即未完成也允许请求合并）

* 3.每次提交前基于develop进行rebase（变基）操作

* 4.在分支上出现提示远程代码有更新时删除远程分支，按目前分支为标准

* 5.功能开发完成后删除本地及远程分支（或者规定在下个版本开始时删除）

> ###### rebase（变基）
> 
> 将当前的修改在最新的节点进行基点改变，相当于把当前的修改全部移动到最新的节点上进行的修改，可以在变基期间就在本地解决了冲突等问题，不需要在merge中去解决这些问题。

###### 修复BUG：

###### 方案1（标准方案）:

* 1.从develop中检出release_version_fixbug分支,进行bug修复

* 2.在该分支上进行测试，测试完成后合入develop和master中并标记Tag

> - 优点：操作方便，定义清晰
> 
> - 缺点：release分支代码稳定性不可控，不能快速的追溯到问题产生点。

###### 方案2（个人推荐）：

> 前提：release定义为预发布分支，只进行最终的回归测试

* 1.从develop中检出feature_version_name_fixbug分支,进行bug修复

* 2.每修复一个bug生成一次提交（【fixbug】detail）(可以快速的定位到问题代码)

* 3.修复完成后进行request merge

* 4.从develop进行前期的开发测试

* 5.开启最终的回归测试后启动release_version预发布分支，从develop检出，回归测试中出现bug只在release_version分支解决，最终merge到develop和master中，并打上Tag。

> 优点：release分支中的代码是有保证且相对稳定的，可以快速追溯到问题点（改动被控制）。develop分支严格控制在开发的整体流程中，master分支被严格保护。
> 
> 缺点：操作较繁琐。



### GitFlow操作流程

##### 创建git-flow仓库

示例图（Sourcetree - 快捷键：option+command+F）：

![](/Users/lm233/Downloads/clipboard3f0feae9c1aa1f65468e69c1129f1f4834537926.jpeg)

##### 新建相关分支:

示例图（Sourcetree - 快捷键：option+command+F）：

![](/Users/lm233/Downloads/clipboard372739f6c4a857af3227b0d7c3fee0f656f22ecd.jpeg)

分支示例图：

![](/Users/lm233/Downloads/clipboard473476e46fdc596830743a8a061b0fcad1b8ab4f.jpeg)



> 如果是使用命令行，控制检出命令就行
> 
> 如： **git checkout -b xxx**


