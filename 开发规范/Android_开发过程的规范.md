## **开发流程规范**

#### **1.项目开发总体流程**
  在整个Android产品的开发流程中，从需求开始到上线所经历的几个流程。
  
  - 根据产品详细需求做功能开发（产品如果没有给到的需求一定要问清楚，不能靠我们开发自己臆想，这个是开发过程中经常发生的事）
  - 按照UI和交互设计做页面与交互开发
  - 按照服务端接口文档，连调接口
  - 自测：根据测试用例进行功能点测试
  - 产品体验：产品体验，对比需求，防止与需求不服。
  - 提测：打包转测试
  - 产品验收：产品通过则上线

![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BC%80%E5%8F%91%E8%A7%84%E8%8C%83/picture/%E5%BC%80%E5%8F%91%E6%B5%81%E7%A8%8B.jpg)



#### **2.技术选型**

- 技术调研
   - 调研本项目需要的技术困难点、是否需要新技术、开源框架等技术
- 技术选型
   - 开发模式、网络框架、开源框架、图片压缩等
- 输出技术文档
- 技术评审
- 搭建项目框架
   - 组件化开发、模块化开发

![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BC%80%E5%8F%91%E8%A7%84%E8%8C%83/picture/%E6%8A%80%E6%9C%AF%E9%80%89%E5%9E%8B.jpg)

案例：[技术选型](http://192.168.10.254/!/#im/view/head/android/%E9%9B%B7%E8%AE%AF%E6%8A%80%E6%9C%AF%E9%80%89%E5%9E%8B(android).pdf)

#### **3.任务拆分**

- 需求分析、任务拆分
   - 根据产品原型做需求分析、任务拆分(功能点 单任务描述 子任务 评估时间 功能责任人)。做到功能点到人
- 实现逻辑、输出迭代版本文档
   - 根据需求把实现过程输出文档
- 评估时间
   - 根据需求分析、任务拆分与实现逻辑 难易程度评估实现时间
- 提交产品
   - 提交该迭代版本的时间线

  ![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BC%80%E5%8F%91%E8%A7%84%E8%8C%83/picture/%E4%BB%BB%E5%8A%A1%E6%8B%86%E5%88%86.jpg)
  
案例：
  [需求拆分](https://docs.google.com/spreadsheets/d/1bVdX6hQyi1KhssHXxlvZHVxLYeygA5Ra4AKbaixJqlI/edit#gid=1534735767)
  
  
#### **4.接口连调**

- 根据服务端任务的优先级做任务排期
  - 可整个迭代排期也可每周根据任务调整任务顺序
  - [案例](https://docs.google.com/spreadsheets/d/1bVdX6hQyi1KhssHXxlvZHVxLYeygA5Ra4AKbaixJqlI/edit#gid=1990953868)
- 后端提供接口文档
  - [接口文档](http://192.168.11.217:3000/project/32/interface/api/2532)
  - 接口路径、请求参数、返回数据等
  - 开发完的接口可实时的跟客户端同步
  
#### **5.方案变更流程**

- 给出变更理由
- 讨论出更合理的技术实现方案
- 产品与项目负责人知晓
- 文档变更并通知参与的所有人