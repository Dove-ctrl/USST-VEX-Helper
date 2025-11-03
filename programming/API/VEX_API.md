## VEX API食用指南
在开始开发项目前，你还需要学习一些VEX API的调用，后续所有获取机器人传感器数据，控制机器人移动，发送或显示遥测数据的代码，本质上都是基于VEX API进行的。

### 什么是API？
API（Application Programming Interface，应用程序编程接口）是一组预定义的规则和工具，允许不同的软件系统或组件之间进行交互和通信。简单来说，它定义了如何调用某个功能、传递什么数据以及返回什么结果，隐藏了底层实现细节，开发者只需关注如何使用接口。API可以类比理解为一家餐厅：

- :hocho:厨房 = 后端系统（实现功能的核心代码）。
- :orange_book:菜单 = API（列出可点的菜品和描述）。
- :cat:服务员 = API的调用过程（传递你的请求并返回结果）。
  
你不需要知道厨房如何做菜，只需通过菜单点餐，就能得到想要的菜品。例如，如果你想要电机旋转，只需要调用能让电机旋转的接口就可以了。

***

### 目录
1. 执行器：
- [电机](/programming/API/motor.md)
- [气动电磁阀](/programming/API/solenoid_valve.md)

2. 控制器：
- [主控](/programming/API/brain.md)

3. 传感器：

***
#### :door: [回到编程](/programming/programming.md)