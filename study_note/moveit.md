# moveit设置
## 1. 打开moveit_setup_assistant
```
rosrun moveit_setup_assistant moveit_setup_assistant
```

## 2. 内容配置
- 1. Self-Collision
    配置自碰撞矩阵。
- 2. Virtual Joints
- 通常，我们利用虚拟关节将机器人的根关节root joint和world坐标系关联起来
- 3. Planning Groups
- 这一步可以将机器人的多个组成部分（links，joints）集成到一个组当中，运动规划会针对一组links或joints完成运动规划，在配置个过程中还可以选择运动学解析器（ kinematic solver）。
- 例如创建两个组，一个是机械臂部分，一个是夹具部分分别进行控制。
- 其中clain可以快速设置一串机器臂
- 4. Robot Poses
- 这里可以设置一些固定的位置，比如说机器人的零点位置、初始位置等等，当然这些位置是用户根据场景自定义的，不一定要和机器人本身的零点位置、初始位置相同。这样做的好处就是我们使用MoveIt！的API编程时，可以直接调用这些位置。
- 5. End-Effectors
- 机械臂在一些实用场景下，会安装夹具等终端结构，可以在这一步中添加。
- 6. Passive Joints
- 机器人上的某些关节，可能在规划、控制过程中使用不到，可以先声明出来。
- 7. Configuration Files
- 最后一步，生成配置文件，直接生成一个包。一般取名 robot_name_moveit_config。

---

# moveit c++ 编程
## 1.



