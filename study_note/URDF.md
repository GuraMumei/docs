# URDF

1. urdf是机器人的描述文件，gazebo中加载的仿真机器人是根据urdf加载的
2. 发布tf的时候也要通过urdf来得到坐标系名称、位置以及不同坐标系之间的关系



## 1. URDF文件

定义一个长方体
```
<geometry>
  <box size="1.0 0.5 0.2"/>
</geometry>
```
定义一个圆柱体
```
<geometry>
  <cylinder length="1.0" radius="0.3"/>
</geometry>
```
定义一个球体
```
<geometry>
  <sphere radius="0.5"/>
</geometry>
```
定义一个三维模型
```
<geometry>
  <mesh filename="package://my_robot/meshes/my_model.stl" scale="0.001 0.001 0.001"/>
</geometry>
```
## 2. xacro基本使用

1.  **xacro:propetry** (解决属性封装问题) 

eg：<xacro:propetry name="wheel_radius" value=0.1 />；

可以通过改变value的值，改变这个name的值，这样就可以一次改变下面代码中所有用到这个变量的地方，而不需要一个一个改。

2. **xacro:macro** (定义宏，解决代码复用问题，类似定义一个函数)

```xaml
<xacro:macro name="wheel_func" params="wheel_name flag">
    ...
```

3. **xacro:宏 参数** （调用宏）

4. .xacro不能被直接使用，要转换为.urdf

```xaml
进入xacro所在目录，执行命令：rosrun xacro xacro xxx.xacro > xxx.urdf，可以把xacro解析为urdf
```

## 3. xacro语法详解

### 1. xmlns

根标签robot 必须包含命名空间声明：xmlns：xacro="http://wiki.ros.org/xacro"

### 2. 属性

1. 封装urdf中一些字段，比如车的尺寸，轮子半径这些

2. 定义：

   eg: <xacro:property name="pi" value="3.14" />

3. 调用：${属性}

   eg: < xxx(标签) name="${pi}" />

4. 算术运算：

   eg: <xxx(标签) result="${pi/2}" />

### 3. 宏

1. 宏类似函数实现，提高代码复用率

2. 定义

```xaml
<xacro:macro name="函数名" params="参数列表（多参数之间使用空格分隔）">

...

</xacro:macro>
```

3. 调用

```
<xacro:宏名称 参数=xxx />
```


### 4. 文件包含

1. 语法：

```
<xacro:include filename="文件名" />
```

## 4. xacro完整使用

1. 编写xacro文件
2. xacro文件集成到launch文件


eg:

### 1.先将xacro文件转换为urdf文件，然后集成urdf文件。
在目标xacro文件目录下打开终端，输入命令
```rosrun xacro xacro xxx_filename.xacro > xxx_filename.urdf```
然后就可在launch文件中加载
```<launch>
    <param name="robot_description" textfile="$(find package_name)/file_path/file_name.urdf" />
	<!-- 其他内容 -->
</launch>
```
### 2.直接在launch文件中以xacro的形式加载。

```
<launch>
    <param name="robot_description" command="$(find xacro)/xacro $(find package_name)/file_path/file_name.xacro" />
	<!-- 其他内容 -->
</launch>
```

---
上述两种方法，加载urdf文件时为 textfile，加载xacro文件时为 command，需注意。

## 4.标签

1. about tag "<safety_controller>"
定义在joint下的标签
   - eg：

   ```xml
   <safety_controller k_velocity="0.1"
                      k_position="100"
                      soft_lower_limit="${pitch_lower_limit+threshold}"
                      soft_upper_limit="${pitch_upper_limit-threshold}"/>
   ```

   - limit标签下的velocity是关节速度的限制，是通过控制力矩的指令来强制执行的，当关节速度越接近limit，那么扭矩会越来越小，如果joint超过了这个速度，会有一个反向的扭矩；

     k_velocity决定了力矩受到的限制的大小(类似p控制器)
     公式如下：施加在关节上的力矩 = -k_velocity * (v_real - v_limit)

   - 同理：
     越接近软限位速度越小，超过软限位会有一个反向的关节速度指令
     k_position决定关节速度的限制(也是类似p控制器)
     公式如下：关节的速度 = -k_position * (p_real - soft_upper_limit)

## 5. note of rm_description

1. <xacro:arg />：定义一个变量
2. <xacro:if > ... </ xacro:if>：相当于if判断

## 6. Transmission
eg:

```
<transmission name="trans_trigger_joint">
        <type>transmission_interface/SimpleTransmission</type>
        <actuator name="trigger_joint_motor">
            <mechanicalReduction>-2.5</mechanicalReduction>
        </actuator>
        <joint name="trigger_joint">
            <hardwareInterface>hardware_interface/EffortJointInterface</hardwareInterface>
            <offset>0.3</offset>
        </joint>
</transmission>
 

1. <transmisson name="xxx">:指定transmission的名字

2. <type>...</type>:指定transmission的类型

3. <actuator name="xxx">:指定transmission连接的驱动器，驱动器名称由name=“xxx”指定

4. <mechanicalReduction>...</mechanicalReduction>:<actuator>的子组件，指定驱动器跟关节的减速比

5. <hardwareInterface>...<hardwareInterface>: 指定支持的关节空间硬件接口；

   transmission加载到gazebo时，这个tag的值EffortJointInterface；

   transimission加载到RobotHW时，值为hardware_interface/EffortJointInterface
6. <joint name="xxx"></joint> :指定的关节
7. <offset>...</offset>：补偿量、限位

   - 如果没有设置offset，开校准控制器校准后，电机会转到一个位置卡住，然后把那个位置设为零点
   - 如果有这个offset，那么会把原本的零点加上一个offset后的位置设置为新的零点
```
**Transmission决定了关节和执行器的映射关系**
## 7. gazebo
eg:  
```
<gazebo reference="xxx">
    <material>Gazebo/Red</material>
</gazebo>
```

定义在gazebo中的颜色
