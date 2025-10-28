## 开发平台
这里使用的VEX机器人程序的开发平台为 **Visual Studio Code**（vscode）加上 **VEX官方扩展** ，语言为 **C++** ，当然也有其他开发选择（例如PROS扩展，官方的VEX CODE PRO V5），但是技巧是通用的，这里只使用这种方式做演示。
- [Visual Studio Code](https://code.visualstudio.com/) ( <<< 官网链接 )
![vscode](/attachment/picture/vscode.png)
    （使用vscode的好处就是有AI代码补全功能，有时候会省事不少）
- VEX官方扩展
![vex ex](/attachment/picture/vex%20ex.png)
    :high_brightness: 建议VEX官方扩展安装 **0.6.0版本** ，因为后续版本貌似终端打印功能失效了，这不利于读取遥测数据和分析机器人状态。
    :high_brightness: 第一次直接安装0.6.0可能会失败，这个时候可以先装最新的版本，然后卸载重装0.6.0或是直接退回0.6.0版本。
***

## 创建新项目
假设你已经配置好了开发环境，那么可以开始这一部分。
步骤如下：
- 打开vscode，点击左边栏的vex扩展
- 选择new project
- 选择V5
- 选择C/C++
- 选择图示模板
  ![template](/attachment/picture/template.png)
- 给你的项目命名，并选择储存路径
  :warning:这里需要注意的是，命名只能用英文字母，下划线_，短横杠-，加减号+-，也不能带空格，否则机器人主控读取不到名称。储存路径最好也不要带有中文，否则可能会出现意外的bug，而且强烈建议新建一个文件夹专门存放你所有的项目，因为默认给的路径会把文件藏得很深，不好找。
***

## 认识项目文件
这一部分帮助你认识一个项目包含哪些文件，以及这些文件有什么作用。准确来说，这一部分及其重要，因为后续的项目文件结构，都会基于这个基础来进行。这里只帮助你认识它们，具体怎么开始编程，会在下一部分介绍。
:high_brightness:建议在开始前把所有文件里面的所有注释都删除掉，这样结构更加清晰。

1. **robot-config.h**
   ```c++
    using namespace vex;

    extern brain Brain;

    void vexcodeInit(void); 
   ```
   这个文件是机器人配置文件之一，主要是 **声明** 机器人的所有电气元件，以及一些全局常量可以放在这里。此外，还包含一个初始化函数，这个后面会提及。

2. **vex.h**
   ```c++
    #include <math.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    #include "v5.h"
    #include "v5_vcs.h"

    #include "robot-config.h"
    ```
    上面这段代码包含VEX API，也就是编程时会用到的函数都会在里面，不需要做修改，这里不过多介绍。


    ```c++
    #define waitUntil(condition)
        do {                                                            \
            wait(5, msec);                                              \
        } while (!(condition))                                          \
    #define repeat(iterations)
        for (int iterator = 0; iterator < iterations; iterator++)       \
   ```
   上面这段代码是官方提供的小工具，可以理解为和if，while一样的控制语句。第一个 `waitUntil(condition)` 表示 **持续等待直到某条件达成** ，这个方法可以阻塞程序进行，避免错过某条件的达成就继续进行下去。 `repeat(iterations)` 表示 **重复执行某行为** ，这个类似于for循环，实现方式也是for循环，这个方法可以理解为帮你省去了for循环繁琐的语法，而你只需要关注重复的内容，以及重复几次。

3. **main.cpp**
   这是所有代码最终执行的地方，也就是main函数所在文件。
   下面说说这个文件的代码结构：
   - **pre_auto函数**
    ```c++
    void pre_auton(void) {
        vexcodeInit();
    }
    ```
    这个函数是最先执行的函数，几乎是程序一启动就会开始执行，因此你可以在这里进行机器人系统的初始化，包括但不限于：惯导的校准，编码器的置零，算法的初始化。
   - **autonomous函数**
    ```c++
    void autonomous(void) {
  
    }
    ```
    这个函数是自动时段函数，比赛的自动时段会执行这个函数，在这里面编写你的自动程序。
   - **usercontrol函数**
    ```c++
    void usercontrol(void) {
  
    }
    ```
    这个函数是手动时段函数，比赛的手动时段会执行这个函数，在这里面编写你的手动程序。当非比赛时间，遥控器没有接入场控就进入程序时，默认是执行手动函数。
   - **main函数**
    ```c++
    int main() {
        Competition.autonomous(autonomous);
        Competition.drivercontrol(usercontrol);

        pre_auton();

        while (true) {
            wait(100, msec);
        }
    }
    ```
    `Competition.autonomous(autonomous);`和`Competition.drivercontrol(usercontrol);`作用是把自动函数和手动函数挂在后台，等待场控命令，并不会直接运行函数里面的内容，因此main函数很快就会执行完毕，但是又不能退出，否则程序就终止了，所以最下面会有一个无限循环，来保证main函数不会退出。
    :high_brightness:最好不要把所有代码都往main.cpp里面塞，因为会显得很杂乱无章，后期修改起来异常困难，更好的做法是，这里只有一些基本的控制逻辑结构，至于具体怎么控制的，放到其他文件的函数里，main.cpp只需引用这些文件即可。
    :warning:不建议 **不了解场控运行机制的人** 去修改这个文件的结构和代码，尤其是main函数，因为及其容易导致程序运行过程中产生严重bug，且阅读了这一部分 **不代表就了解场控运行机制** 。

4. **robot-config.cpp**
   ```c++
    #include "vex.h"

    using namespace vex;

    brain Brain;

    void vexcodeInit(void) {
    
    }
   ```
   这个文件是机器人的另一个配置文件，主要是 **定义** 机器人的所有电气元件。然后这里也定义了初始化函数`vexcodeInit()`，在这个函数里面可以去初始化你的传感器和算法，它会在`pre_auton()`里面执行。
***

#### :door: [回到编程](/programming/programming.md)