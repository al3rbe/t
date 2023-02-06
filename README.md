#include <Encoder.h>
#include <ros.h>
#include <geometry_msgs/Twist.h>


#define ENCODER_PULSES_PER_REVOLUTION 360
#define WHEEL_DIAMETER 0.067
#define WHEEL_CIRCUMFERENCE 0.2091
#define WHEEL_RADIUS 0.0335

ros::NodeHandle nh;
Encoder leftEncoder(18, 19);
Encoder rightEncoder(20, 21);

geometry_msgs/Twist twist_msg;
ros::Publisher cmd_vel("cmd_vel", &twist_msg);

unsigned long left_encoder_pulses;
unsigned long right_encoder_pulses;
double left_velocity;
double right_velocity;
double linear_velocity;
double angular_velocity;

void setup() {
  nh.initNode();
  nh.advertise(cmd_vel);
}

void loop() {
  left_encoder_pulses = leftEncoder.read();
  right_encoder_pulses = rightEncoder.read();

  left_velocity = (left_encoder_pulses / ENCODER_PULSES_PER_REVOLUTION) * WHEEL_CIRCUMFERENCE;
  right_velocity = (right_encoder_pulses / ENCODER_PULSES_PER_REVOLUTION) * WHEEL_CIRCUMFERENCE;

  linear_velocity = (left_velocity + right_velocity) / 2;
  angular_velocity = (right_velocity - left_velocity) / (2 * WHEEL_RADIUS);

  twist_msg.linear.x = linear_velocity;
  twist_msg.angular.z = angular_velocity;

  cmd_vel.publish(&twist_msg);
  nh.spinOnce();
  delay(100);
}
