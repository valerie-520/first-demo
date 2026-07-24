# 一、ROS环境+命令
## 1.核心概念
roscore：ROS 总通信中枢，所有程序必须连它
工作空间：catkin_ws/src 放所有代码包
编译：catkin_make
## 2.命令
roscore                          # 开核心
rosrun 包名 节点名                # 单独运行单个程序
roslaunch 包名 xxx.launch        # 一键启动多机器人/仿真（比赛核心）
rostopic list                    # 查看所有话题
rostopic echo /话题名            # 打印话题数据（看球坐标、速度）
rostopic pub /话题 消息类型 参数  # 手动发指令调试机器人
rosnode list / rosnode info 节点  # 查运行程序
rosbag record -a -O game.bag     # 录制整场比赛
rosbag play game.bag             # 回放对局
rviz                             # 可视化球场、TF坐标
## 3.实操
### 1.创建空工作空间
mkdir -p ~/catkin_ws/src && cd ~/catkin_ws && catkin_make
source devel/setup.bash
### 2.运行小乌龟demo，把上面所有命令全部实操一遍
验收：能独立录bag、回放、用rostopic控制小乌龟移动
# 二、话题发布订阅（python为主）
## 1.核心模型：发布者（发数据）————订阅者（收数据）
比赛用途：机器人发速度、球坐标、全场状态
## 2.最简模板
1.发布者（发速度）
import rospy
from geometry_msgs.msg import Twist

rospy.init_node("vel_pub")
pub = rospy.Publisher("/bot1/cmd_vel", Twist, queue_size=10)
rate = rospy.Rate(10)

while not rospy.is_shutdown():
    msg = Twist()
    msg.linear.x = 0.2
    msg.angular.z = 0
    pub.publish(msg)
    rate.sleep()
2.订阅者（接受球坐标）
import rospy
from geometry_msgs.msg import Point

def callback(data):
    rospy.loginfo("球坐标 x=%f y=%f", data.x, data.y)

rospy.init_node("ball_sub")
sub = rospy.Subscriber("/ball_pos", Point, callback)
rospy.spin()    
# 三、Launch文件+多机器人命名空间
## 1.launch三大核心标签
node:启动节点
param:加载参数
group ns="bot1":命名空间隔离bot1~bot5,话题自动加前缀
## 2.比赛标准多机器人模板
<launch>
  <!-- 一号机器人所有节点装进bot1命名空间 -->
  <group ns="bot1">
    <node pkg="robot" type="perception.py" name="percep" output="screen"/>
    <node pkg="robot" type="decision.py" name="dec" output="screen"/>
  </group>
  <group ns="bot2">
    <node pkg="robot" type="perception.py" name="percep" output="screen"/>
  </group>
</launch>

# 四、参数服务器yaml+ROS计时器（防犯规核心）
## 1.参数用法
yaml写参数（踢球力度、禁区10秒阈值），launch加载，代码读取
yaml示例：
kick_power: 6.0
zone_max_time: 10.0
代码读取：
power = rospy.get_param("/bot1/kick_power")
## 2.Timer计时器（规则强制）
用来统计机器人在禁区停留时间、带球时长
def timer_cb(event):
    global stay_time
    stay_time += 0.1
timer = rospy.Timer(rospy.Duration(0.1), timer_cb)
验收：写计时器，满10秒打印“超时，离开禁区”
# 五、TF2坐标变换
核心目标：机身坐标系<->全局球场坐标系互相转换
## 1.广播TF（发布自身位置）
## 2.监听TF（查询球相对自己的位置）
import tf2_ros
from geometry_msgs.msg import TransformStamped

tfBuffer = tf2_ros.Buffer()
listener = tf2_ros.TransformListener(tfBuffer)
try:
    trans = tfBuffer.lookupTransform("world", "bot1_base", rospy.Time(0), rospy.Duration(0.1))
except:
    passimport tf2_ros
from geometry_msgs.msg import TransformStamped

tfBuffer = tf2_ros.Buffer()
listener = tf2_ros.TransformListener(tfBuffer)
try:
    trans = tfBuffer.lookupTransform("world", "bot1_base", rospy.Time(0), rospy.Duration(0.1))
except:
    pass
# 六、Grazebo仿真交互，读取全场物体坐标（感知数据源）
## 唯一关键话题/grazebo/model_states
Gazebo会把机器人、足球、球门全部坐标打包发出来，是你感知层输入
## 代码逻辑：
订阅话题->遍历索引模型名称->提取ball、bot1、bot2坐标
# 七、ROS服务Service（踢球、裁判单次指令）
话题是持续数据流，服务是一问一答单词操作，用于踢球、复位球
## 服务端（踢球执行）
import rospy
from std_srvs.srv import Trigger, TriggerResponse

def kick_handle(req):
    # 执行踢球动作
    return TriggerResponse(success=True, message="kick done")

rospy.init_node("kick_server")
srv = rospy.Service("/bot1/kick", Trigger, kick_handle)
rospy.spin()
## 客户端（决策层调用踢球）
rospy.wait_for_service("/bot1/kick")
cli = rospy.ServiceProxy("/bot1/kick", Trigger)
res = cli.call()