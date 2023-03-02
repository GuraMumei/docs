# moveit设置
## 1. 打开moveit_setup_assistant
```
rosrun moveit_setup_assistant moveit_setup_assistant
```

## 2. 内容配置
- 1. Self-Collision
- 配置自碰撞矩阵。
- 
- 2. Virtual Joints
- 通常，我们利用虚拟关节将机器人的根关节root joint和world坐标系关联起来
- 
- 3. Planning Groups
- 这一步可以将机器人的多个组成部分（links，joints）集成到一个组当中，运动规划会针对一组links或joints完成运动规划，在配置个过程中还可以选择运动学解析器（ kinematic solver）。
- 例如创建两个组，一个是机械臂部分，一个是夹具部分分别进行控制。
- 其中clain可以快速设置一串机器臂
- 
- 4. Robot Poses
- 这里可以设置一些固定的位置，比如说机器人的零点位置、初始位置等等，当然这些位置是用户根据场景自定义的，不一定要和机器人本身的零点位置、初始位置相同。这样做的好处就是我们使用MoveIt！的API编程时，可以直接调用这些位置。
- 
- 5. End-Effectors
- 机械臂在一些实用场景下，会安装夹具等终端结构，可以在这一步中添加。
- 
- 6. Passive Joints
- 机器人上的某些关节，可能在规划、控制过程中使用不到，可以先声明出来。
- 
- 7. Configuration Files
- 最后一步，生成配置文件，直接生成一个包。一般取名 robot_name_moveit_config。

---

# moveit c++ 编程
## 1. 头文件  

```
#include <moveit/move_group_interface/move_group.h>
#include <moveit/planning_scene_interface/planning_scene_interface.h>
#include <moveit_msgs/AttachedCollisionObject.h>
#include <moveit_msgs/CollisionObject.hnclude <moveit/move_group_interface/move_group.h>
```

## 2. API
```
 move_group_interface::MoveGroup group("group_name")  //初始化group并连接到在Planning Groups中设定的group_name
 ```
 
 *注：在MoveIt库的早期版本中，使用moveit::planning_interface::MoveGroupInterface来控制机器人运动，但是在后来的版本中，为了提高API的易用性，引入了move_group_interface::MoveGroup类，两个类的功能完全相同，只是名称不同。从MoveIt 1.1.0版本开始，move_group_interface::MoveGroup被作为推荐的API类，而moveit::planning_interface::MoveGroupInterface仍然被保留，以保证向后兼容性。*


```
  geometry_msgs::Pose target_pose1;
  target_pose1.orientation.w = ... ;
  target_pose1.orientation.x= ... ;
  target_pose1.orientation.y = ... ;
  target_pose1.orientation.z = ... ;
  target_pose1.position.x = ... ;
  target_pose1.position.y = ... ;
  target_pose1.position.z = ... ;
  group.setPoseTarget(target_pose1);  // 设置机器人终端的目标位置
```

```
  moveit::planning_interface::MoveGroup::Plan my_plan;  //进行运动规划，计算机器人移动到目标的运动轨迹，此时只是计算出轨迹，并不会控制机械臂运动
  bool success = group.plan(my_plan)  //是否成功规划
  if(success)
    group.execute(my_plan);  //让机械臂按照规划的轨迹开始运动。
```

```
moveit::planning_interface::PlanningSceneInterface planning_scene_interface; //使用 PlanningSceneInterface 类在“虚拟世界”场景中添加和删除碰撞对象
```

```
  moveit_msgs::OrientationConstraint ocm;
  ocm.link_name = "robot_link";
  ocm.header.frame_id = "base_link";
  ocm.orientation.w = 1.0;
  ocm.absolute_x_axis_tolerance = 0.1;
  ocm.absolute_y_axis_tolerance = 0.1;
  ocm.absolute_z_axis_tolerance = 0.1;
  ocm.weight = 1.0;  //定义约束

//这个路径约束规划是基于机器人的末端执行器（end effector）的方向和姿态（orientation）的约束。具体来说，这个约束规定了在机器人的末端执行器（链接名称为 "robot_link"）上，以 "base_link" 坐标系为参考坐标系，姿态的四元数（w=1.0 表示无旋转）应该在一个半径为 0.1 的球体内，并且在 x、y、z 轴方向的绝对容忍度也是 0.1。权重值为 1.0。

  moveit_msgs::Constraints test_constraints;
  test_constraints.orientation_constraints.push_back(ocm);
  group_name.setPathConstraints(test_constraints);
//添加到指定规划组
`‵‵

```
std::vector<double> joint_group_positions;
  current_state->copyJointGroupPositions(joint_model_group, joint_group_positions);  //获取当前关节位置
```

```
robot_state::RobotState start_state(*group_name.getCurrentState());
  geometry_msgs::Pose start_pose2;
  
  ...
  start_state.setFromIK(joint_model_group, start_pose2);  //通过逆向运动学（IK）求解
  move_group.setStartState(start_state);  //设置初始状态
```

```
 move_group.setPlanningTime(10.0);  //设规划时间
```


