cmake_minimum_required(VERSION 2.8.3)
project(futurakart_msgs)

find_package(catkin REQUIRED COMPONENTS
  std_msgs
  message_generation
)

add_message_files(
  FILES
  MotorDrive.msg
  MotorFeedback.msg
  Status.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(CATKIN_DEPENDS std_msgs message_runtime)
