```c++
class PID {
protected:
    double P, I, D;
    double kp, ki, kd;
    double target, error_tol;
    double I_Max, D_Tol;
    double output;
    double pre_error;
    bool arrived;
    int arrived_times;
    int current_times;

public:
    PID():
        P(0), I(0), D(0),
        kp(0), ki(0), kd(0),
        target(0), error_tol(0.01),
        I_Max(100), D_Tol(0.01),
        output(0), pre_error(0)
    {}
    
    void set_coefficient(double _kp , double _ki , double _kd){
        kp = _kp;
        ki = _ki;
        kd = _kd;
    }
    void set_error_tol(double _error_tol){error_tol = _error_tol;}
    void set_target(double _target){target = _target;}

    double get_output(){return output;}
    double get_target(){return target;}

    void set_I_Max(double _I_Max){I_Max = _I_Max;};
    void set_D_Tol(double _D_Tol){D_Tol = _D_Tol;};

    void reset(){
        output = 0;
        arrived = false;
        current_times = 0;
        I = 0;
    };

    void set_arrived_times(double _arrived_times){arrived_times = _arrived_times;}
    bool is_arrived(){return arrived;}

    void update(double input) override {
        double error = target - input;
    
        if(fabs(error) <= error_tol){
            if(current_times == arrived_times){
                arrived = true;
            }
            current_times++;
        }
        
        P = error;

        I += error;
        if(fabs(I) > I_Max) {
            I = sign(I) * I_Max;
        }
        
        D = error - pre_error;
        if(fabs(D) < D_Tol){
            D = 0;
        }

        pre_error = error;
        
        output = (P * kp) + (I * ki) + (D * kd);
    }
};
```

***
#### :door: [回到编程](/programming/programming.md)