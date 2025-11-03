## 主控
V5主控的功能很多，这里可能没法全部列举完，而且这些接口整合成一个可用的系统需要花些时间，但是不妨碍这些函数接口依然有探索价值。

***

#### 声明和定义
```c++
brain Brain;
```

***

#### 主控屏幕显示函数
```c++
Brain.Screen.clearLine(1);
Brain.Screen.clearScreen();
Brain.Screen.printAt(100 , 100 , "hello world");
Brain.Screen.printAt(100 , 100 , "x = &d" , chassis.x);
```
这些函数都是用于控制主控屏幕显示的函数，可以在屏幕上显示遥测数据，方便调试。

***
#### :door: [回到API](/programming/API/VEX_API.md)