```c++
class odometry{
private:
    //Configuration
    const double MOUNTING_ANGLE = 45;
    const double TRACK_WHEEL_R = 26.972; //mm

public:
    Pose odom_pose; 
    double odom_liner_velocity;

    odometry():
        odom_pose({0.0 , 0.0 , 0.0}),
        odom_liner_velocity(0.0),
        odom_angular_velocity(0.0)
    {}

    void Update();
};

class chassis{
private:
    chassis():
        robot_pose({0.0 , 0.0 , 0.0}),
        yaw(0.0),
        left_voltage(0.0), right_voltage(0.0)
    {
        left_v_pid.set_coefficient(7 , 0 , 0);
        right_v_pid.set_coefficient(7 , 0 , 0);
        left_v_pid.set_error_tol(1);
        right_v_pid.set_error_tol(1);
        left_v_pid.set_D_Tol(1);
        right_v_pid.set_D_Tol(1);
        left_v_pid.set_I_Max(2000);
        right_v_pid.set_I_Max(2000);

        turn_pid.set_coefficient(300 , 0 , 700);
        turn_pid.set_error_tol(5);
        turn_pid.set_D_Tol(0.2);
        turn_pid.set_I_Max(5000);
        turn_pid.set_arrived_times(2);
    }
    chassis(const chassis &_chassis) = delete;
    const chassis &operator=(const chassis &_chassis) = delete;

    //Physical configuration
    static const uint32_t CHASSIS_MOTOR_NUMBER = 6;
    const double WHEEL_BASE = 330.2; //mm
    const double DRIVE_WHEEL_R = 41.275; //mm
    const double GEAR_RATIO = 1;

    //Controller configuration
    const double MAX_VOLTAGE = 12000; // mV
    const double K_OP_LINER = 120;
    const double K_OP_ANGULAR = 90;
    const double MAX_CURVATURE = 0.7; //曲率达到该值时降速效果最大
    const double MIN_SPEED_FACTOR = 0.5; //最低速度占比，曲率极大时速度不低于基础速度的百分比
    const double LOOKAHEAD_DISTANCE = 300.0; //mm，基础前视距离
    const double LOOKAHEAD_DISTANCE_FACTOR = 0.05; //动态前视距离系数
    const double POSITION_THRESHOLD = 100.0; //mm，终点阈值
    const double SEGMENTS_SLOW_FACTOR = 0.8; //大于插值点数的该百分比就开始减速
    const double SLOW_FACTOR = 0.7; //减速系数
    const double TIMEOUT = 1250;

    //Parameters
    double left_velocity , right_velocity; // mm/s

    double liner_velocity; // mm/s
    double angular_velocity; // rad/s

    //Pose parameters
    double yaw; // degree
    Pose robot_pose;

    //Controller
    double left_voltage , right_voltage;
    incremental_PID left_v_pid;
    incremental_PID right_v_pid;
    position_PID turn_pid;

public:
    static chassis& GetInstance(){
        static chassis Chassis;
        return Chassis;
    }

    void ChassisRoute();

    //Odometry
    odometry Odometry;
    
    //Drive
    void PoseDrive();
    void VelocityDrive();
    void MovementDrive();
    static void ChassisDrive(){
        thread odometry_drive(
            [](){
                chassis::GetInstance().Odometry.Update();
            }
        );
        while(true){
            chassis::GetInstance().MovementDrive();
            chassis::GetInstance().PoseDrive();
            chassis::GetInstance().VelocityDrive();
            wait(10,msec);
        }
    }

    //Control
    void SetBrake(brakeType btype);
    void SetBrakeType(brakeType btype);

    double ComputeTargetSpeed(double baseSpeed, double curvature);
    void PurePursuit(const std::vector<Point>& controlPoints, int segments , double base_speed);
    void Turn(double direction);
    void PushFor(double time , double power);

    //Output
    double GetYaw();
    Pose GetPose();
};

void chassis::MovementDrive(){ //底盘移动驱动，自动时段由程序决定左右电压，手动时段由遥控器决定左右电压
    if(eos::COMPETITION->isAutonomous() || eos::AUTODEBUG){

        //nothing to do

    }else if(eos::COMPETITION->isDriverControl() && !eos::AUTODEBUG){

        left_voltage = LinerAxis()*K_OP_LINER + AngularAxis()*K_OP_ANGULAR;
        right_voltage = LinerAxis()*K_OP_LINER - AngularAxis()*K_OP_ANGULAR;
        
    }
    
    LF.spin(fwd , left_voltage , voltageUnits::mV);
    LM.spin(fwd , left_voltage , voltageUnits::mV);
    LB.spin(fwd , left_voltage , voltageUnits::mV);

    RF.spin(fwd , right_voltage , voltageUnits::mV);
    RM.spin(fwd , right_voltage , voltageUnits::mV);
    RB.spin(fwd , right_voltage , voltageUnits::mV);
}

void chassis::VelocityDrive(){ //速度更新驱动
    left_velocity = rpm_to_mm_s(
        (
            LF.velocity(velocityUnits::rpm) + 
            LM.velocity(velocityUnits::rpm) + 
            LB.velocity(velocityUnits::rpm)
        )/(CHASSIS_MOTOR_NUMBER/2) * GEAR_RATIO,
        DRIVE_WHEEL_R * 2.0
    );
    right_velocity = rpm_to_mm_s(
        (
            RF.velocity(velocityUnits::rpm) + 
            RM.velocity(velocityUnits::rpm) + 
            RB.velocity(velocityUnits::rpm)
        )/(CHASSIS_MOTOR_NUMBER/2) * GEAR_RATIO,
        DRIVE_WHEEL_R * 2.0
    );

    liner_velocity = Odometry.odom_liner_velocity;
}

void chassis::PoseDrive(){ //位姿更新驱动
    robot_pose.x = Odometry.odom_pose.x;
    robot_pose.y = Odometry.odom_pose.y;
    robot_pose.theta = Odometry.odom_pose.theta;
    yaw = robot_pose.theta * RAD_TO_DEG;
    if(yaw > 0){
        yaw = fmod(yaw, 360.0);
    }else if(yaw < 0){
        yaw = 360 - fmod(fabs(yaw), 360.0);
    }else{
        yaw = 0;
    }
}

void odometry::Update(){
    double imu_rotation = 0.0;
    double pre_encoderL = 0.0 , pre_encoderR = 0.0;
    while(true){
        imu_rotation = sign(Inertial.rotation()) * fabs(Inertial.rotation() + Inertial.rotation(rev) * IMU_COMPENSATION);
        odom_pose.theta = imu_rotation * DEG_TO_RAD;

        double encoderL_position = EncoderL.position(rotationUnits::deg) * DEG_TO_RAD * TRACK_WHEEL_R;
        double encoderR_position = EncoderR.position(rotationUnits::deg) * DEG_TO_RAD * TRACK_WHEEL_R;
        odom_liner_velocity = rpm_to_mm_s(
            EncoderL.velocity(velocityUnits::rpm) * Cos(MOUNTING_ANGLE) + EncoderR.velocity(velocityUnits::rpm) * Sin(MOUNTING_ANGLE),
            TRACK_WHEEL_R * 2.0
        );
        double d_encoderL = encoderL_position - pre_encoderL;
        double d_encoderR = encoderR_position - pre_encoderR;
        pre_encoderL = encoderL_position;
        pre_encoderR = encoderR_position;

        double dx_local = d_encoderR * Cos(MOUNTING_ANGLE) - d_encoderL * Sin(MOUNTING_ANGLE);
        double dy_local = d_encoderR * Sin(MOUNTING_ANGLE) + d_encoderL * Cos(MOUNTING_ANGLE);
        double global_dx = dx_local * cos(odom_pose.theta) + dy_local * sin(odom_pose.theta);
        double global_dy = -dx_local * sin(odom_pose.theta) + dy_local * cos(odom_pose.theta);
    
        odom_pose.x += global_dx;
        odom_pose.y += global_dy;

        wait(5,msec);
    }
}

double chassis::ComputeTargetSpeed(double baseSpeed, double curvature) {
    double factor = 1.0 - std::min(std::fabs(curvature) / MAX_CURVATURE, 1.0) * (1.0 - MIN_SPEED_FACTOR);
    return baseSpeed * factor;
}

void chassis::Turn(double direction){
    double total_time = eos::SystemTime(msec);
    double delta_direction = 0.0;
    eos::TerminalPrint("# Turn:" , WHITE);
    eos::TerminalPrintWithData("   target_yaw(deg) = " , fabs(direction) , BLUE);
    eos::TerminalPrintWithData("   start_yaw(deg) = " , yaw , YELLOW);

    if(direction >= 0){
        delta_direction = (fabs(direction) > yaw ? (fabs(direction) - yaw) : (360 + fabs(direction) - yaw));
    }else{
        delta_direction = (fabs(direction) < yaw ? (yaw - fabs(direction)) : (360 + yaw - fabs(direction)));
    }

    double target_theta = robot_pose.theta * RAD_TO_DEG + sign(direction) * delta_direction;
    
    turn_pid.set_target(target_theta);
    while(!turn_pid.is_arrived() && (eos::SystemTime(msec) - total_time) <= TIMEOUT){
        turn_pid.update(robot_pose.theta * RAD_TO_DEG);
        left_voltage = max_value_limit(MAX_VOLTAGE , turn_pid.get_output());
        right_voltage = -max_value_limit(MAX_VOLTAGE , turn_pid.get_output());
        wait(50,msec);
    }
    SetBrake(brake);
    turn_pid.reset();
    total_time = eos::SystemTime(msec) - total_time;
    eos::TerminalPrintWithData("   end_yaw(deg) = " , yaw , YELLOW);
    eos::TerminalPrintWithData("   time(msec) = " , total_time , MAGENTA);
    eos::TerminalEndl();
}

void chassis::PurePursuit(const std::vector<Point>& controlPoints, int segments , double base_speed) {
    std::vector<Point> path = PathPlan(controlPoints, segments);
    size_t target_index = 0;
    double total_time = eos::SystemTime(msec);
    double max_speed = 0.0;
    eos::TerminalPrint("# PurePursuit:" , WHITE);
    eos::TerminalPrintWithData("   target_speed(mm/s) = " , base_speed , GREEN);
    eos::TerminalPrintWithData("   start_yaw(deg) = " , yaw , YELLOW);
    eos::TerminalPrintWithData("   start_point_x = " , robot_pose.x , BLUE);
    eos::TerminalPrintWithData("   start_point_y = " , robot_pose.y , BLUE);

    while(true) {
        max_speed = sign(liner_velocity) * std::max(max_speed, fabs(liner_velocity));

        double robot_x = robot_pose.x;
        double robot_y = robot_pose.y;
        double robot_theta = robot_pose.theta;

        double dynamic_lookahead = LOOKAHEAD_DISTANCE + LOOKAHEAD_DISTANCE_FACTOR * liner_velocity;
        
        //检查是否到达终点
        double dist_to_end = point_distance(path.back(), {robot_x, robot_y});
        if(dist_to_end < POSITION_THRESHOLD) {
            chassis::GetInstance().SetBrake(brake);
            break;
        }

        //保证目标点向前更新
        while (target_index < path.size()-1) {
            double dist_to_target = point_distance(path[target_index], {robot_x, robot_y});
            if (dist_to_target <= dynamic_lookahead) {
                target_index++;
            } else {
                break;
            }
        }
        Point target_point = path[target_index];

        //计算全局差值
        double dx = target_point.x - robot_x;
        double dy = target_point.y - robot_y;

        //计算曲率
        double local_x = dx * cos(robot_theta) - dy * sin(robot_theta);
        double local_y = dx * sin(robot_theta) + dy * cos(robot_theta);
        double L = point_distance({0, 0}, {local_x, local_y});
        double curvature = (L >= 1e-5 ? 2.0 * local_x / (L * L) : 0.0);

        //生成左右速度目标值
        double target_speed = ComputeTargetSpeed(base_speed, curvature);
        if(target_index + 1 >= segments * SEGMENTS_SLOW_FACTOR){
            target_speed = target_speed * SLOW_FACTOR;
        }
        double omega = target_speed * curvature;
        double leftTargetSpeed = target_speed + (omega * WHEEL_BASE / 2.0);
        double rightTargetSpeed = target_speed - (omega * WHEEL_BASE / 2.0);

        //计算输出
        left_v_pid.set_target(leftTargetSpeed);
        right_v_pid.set_target(rightTargetSpeed);
        left_v_pid.update(left_velocity);
        right_v_pid.update(right_velocity);
        left_voltage = left_v_pid.get_output();
        right_voltage = right_v_pid.get_output();

        wait(10,msec);
    }

    chassis::GetInstance().SetBrake(brake);
    total_time = eos::SystemTime(msec) - total_time;
    eos::TerminalPrintWithData("   max_speed(mm/s) = " , max_speed , GREEN);
    eos::TerminalPrintWithData("   end_yaw(deg) = " , yaw , YELLOW);
    eos::TerminalPrintWithData("   end_point_x = " , robot_pose.x , BLUE);
    eos::TerminalPrintWithData("   end_point_y = " , robot_pose.y , BLUE);
    eos::TerminalPrintWithData("   time(msec) = " , total_time , MAGENTA);
    eos::TerminalEndl();
}

void chassis::PushFor(double time , double power) {
    double total_time = eos::SystemTime(msec);
    eos::TerminalPrint("# PushFor:" , WHITE);
    eos::TerminalPrintWithData("   target_time(msec) = " , time , GREEN);
    eos::TerminalPrintWithData("   power(pct) = " , power , YELLOW);

    left_voltage = right_voltage = MAX_VOLTAGE * power / 100.0;

    wait(time,msec);

    chassis::GetInstance().SetBrake(brake);
    total_time = eos::SystemTime(msec) - total_time;
    eos::TerminalPrintWithData("   time(msec) = " , total_time , MAGENTA);
    eos::TerminalEndl();
}
```
***
#### :door: [回到编程](/programming/programming.md)