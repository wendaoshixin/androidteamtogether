# **Android CodeReview**



**代码质量分等级**：

> 可以编译通过->可以正常运行->可以测试通过->容易阅读->容易扩展，维护

**CodeReview的好处**：

> 通过Code Review，对于同样的功能实现，有经验的工程师可以给经验尚浅的工程师提供合理的优化建议。经验尚浅的工程师可以通过阅读优质代码，快速学习相关技术运用的最佳实践。

**CodeReview的落地执行**：

- **小伙伴的Code自查**: 考虑通过代码扫描工具，AndroidStudio的自带的Link或者第三方插件(例如，[阿里的P3C插件](https://github.com/alibaba/p3c)）,来处理低级的问题/BUG。
- **指定组内小伙伴审查合并**：考虑通过代码审查工具(gerrit或者gitlab)，进行强制CodeReView。通过merge request来review代码，不符合规范或者存在问题的代码，进行打回。例如，[基于GitLab的Code Review教程](https://ken.io/note/gitlab-code-review-tutorial)。

