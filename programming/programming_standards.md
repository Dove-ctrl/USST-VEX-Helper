## C++编写规范
在开发前，还有一些准备工作要做。如果你编写代码的经验很少，那么我强烈建议你认真阅读这一部分。
- ***为什么要强调编写规范？***
  不规范的编写会给代码维护工作带来很多麻烦，工整清晰的代码编写，能大大提高可读性和开发效率，几乎可以完全规避各种低级错误。
- ***为什么要维护代码？***
  关于机器人程序的编写实际上是一个项目，而非一个简单的自动脚本，因此其中可能出现一些逻辑错误，也可能需要添加或删减一些功能等情况，这个时候就需要团队成员维护代码，去解决这些问题。

1. **缩进和空格**
   1. 缩进对齐，属于同一个代码块的代码对齐，不同的错开。  
        Do this. :heavy_check_mark:
        ```c++
        while(true){
            angle = Encoder.position();
            if(angle >= 90){
                Motor.spin(100 , pct);
                wait(500 , msec);
                Motor.stop();
            }
            Encoder.reset();
        }

        ```
        Don't do this. :x:
        ```c++
        while(true){
        angle = Encoder.position();
                if(angle >= 90){
                    Motor.spin(100 , pct);
            wait(500 , msec);
                Motor.stop();
                    }
            Encoder.reset();
                }
        ```
    2. 行宽过长时换行对齐。  
        Do this. :heavy_check_mark:
        ```c++
        left_velocity = (
            rpm_to_mm_s(LF.velocity(velocityUnits::rpm) * GEAR_RATIO , DRIVE_WHEEL_R * 2) +
            rpm_to_mm_s(LMF.velocity(velocityUnits::rpm) * GEAR_RATIO , DRIVE_WHEEL_R * 2) +
            rpm_to_mm_s(LMB.velocity(velocityUnits::rpm) * GEAR_RATIO , DRIVE_WHEEL_R * 2) +
            rpm_to_mm_s(LB.velocity(velocityUnits::rpm) * GEAR_RATIO , DRIVE_WHEEL_R * 2)
        ) / 4;

        ```
        Don't do this. :x:
        ```c++
        left_velocity = ( rpm_to_mm_s(LF.velocity(velocityUnits::rpm) * GEAR_RATIO , DRIVE_WHEEL_R * 2) + rpm_to_mm_s(LMF.velocity(velocityUnits::rpm) * GEAR_RATIO , DRIVE_WHEEL_R * 2) + rpm_to_mm_s(LMB.velocity(velocityUnits::rpm) * GEAR_RATIO , DRIVE_WHEEL_R * 2) + rpm_to_mm_s(LB.velocity(velocityUnits::rpm) * GEAR_RATIO , DRIVE_WHEEL_R * 2) ) / 4;
        ```
    3. 符号，对象之间用空格隔开。  
        Do this. :heavy_check_mark:
        ```c++
        point.x = uu * p0.x + 2 * u * t * p1.x + tt * p2.x;

        PurePursuit(QuadraticBezierPathPlan(p0 , p1 , p2 , 200) , 1600);
        ```
        Don't do this. :x:
        ```c++
        point.x=uu*p0.x+2*u*t*p1.x+tt*p2.x;
        
        PurePursuit(QuadraticBezierPathPlan(p0,p1,p2,200),1600);
        ```
    4. 括号风格。  
        三种里面任选，但是最好同一个项目统一风格
        ```c++
        if(true){/* code */}

        while(true)
        {
            /* code */
        }

        do{
            /* code */
        }while(true)
        ```
2. **命名规范**
    我们会给：类（结构体）、函数、常量、全局变量、静态变量、成员变量、临时变量（参数）等进行命名，总体原则：**清晰、简洁、避免缩写（除非广泛接受）**。  
    Do this. :heavy_check_mark:
    ```c++
    int object_num = 0; //代表物体的数目，使用单词object和常用缩写num（number）
    float driveVoltage = 1000; //代表驱动电压，使用单词drive和voltage
    int right_motor_velocity = 100; //右电机速度
    int left_motor_velocity = 100; //左电机速度
    const int MAX_VELOCITY = 200; //代表最大速度，常量名称全部大写来做区分

    void ControllerBtnCheck(); //代表遥控器按钮状态检查，使用单词controller，check和常用缩写btn（button）
    void MotorVelocitySet(int right, int left); //设置电机速度
    void MotorVelocityGet(int &right, int &left); //获取电机速度

    class chassis{ //底盘类
    public:
        chassis();
    };
    class odometry{ //里程计类
    public:
        odometry();
    };
    ```
    Don't do this. :x:
    ```c++
    //含义完全不清楚
    int a = 1; 
    int abc = 2;
    int aabbcc = 3;

    //拼音缩写
    int woshinidie = 2; 

    //拼音首字母缩写
    float wjsynkbd(int nybsldw); 

    double this_is_encoder_value = 0.135; //名字太长，代码显得冗长，不方便快速浏览

    int MV = 12000; //本意是Motor Voltage，但是没有人用这个缩写，这是非公认的。

    float forward();//forward是VEX API里的东西，虽然编译可能不会报错，但是会影响阅读者的判断。
    ```

3. **注释**
   - ***为什么要写注释？***
    对于刚开始接触编程的人来说，勤写注释能帮助他更好的理解自己在做什么。注释还能更清晰地展现代码的结构，方便后续的维护和修改。因此，养成写注释的习惯一定不是坏事。
    1. 普通注释。  
        对变量，函数或对象的注释，一般两种方式：要么写在上面一行，要么写在这行的结尾，需要注意的是，写在上面一行时需要空一行，和其他代码分开来。同理，写在结尾需要打上空格再写注释。还有，注释需要已尽量简洁的话语表达含义，不可太长。  
        ```c++
        void exegesis(){
            int number = 1; //变量

            //函数
            exegesis();

            /* 
                让机器人移动指定距离
                参数为距离
            */
            MoveDistance(double dist);

            //如果按钮被按下，就做...否则...
            if(is_pressing()){
                /* code */
            }else{
                /* code */
            }
        }
        ```
    2. Doxygen注释
        对某个定义的注释，这里建议用Doxygen注释，右键变量或函数名可以看到 **生成Doxygen注释** 的选项，当你在项目的任何地方，只要鼠标光标移到这个被Doxygen注释过的函数上时，会显示这个函数的详细注释，非常方便。
        ```c++
        /// @brief 这是一个变量
        float value = 1.234;

        /// @brief 这是一个示例函数
        /// @param i 形参i
        void example(int i);
        ```
:apple:当然，还有其他代码编写规范，这里就只介绍常用的一些，如果你能遵守这些规范，对于VEX机器人程序编写来说足够了，网络上关于这部分的资料相当多，感兴趣的朋友可以自己查阅。
***
#### :door: [回到编程](/programming/programming.md)