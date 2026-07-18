ROS2 入门 HelloWorld 学习日志
基本信息
学习日期：2026-07-18
环境：Oracle VirtualBox Ubuntu 24.04 + VSCode + ROS2 Humble
仓库用途：ROS2 学习笔记归档
一、今日学习内容
1. ROS2 工作空间基础流程
目录结构规范
工作空间：ros2_ws
功能包存放目录：ros2_ws/src/
Python 功能包内部双层文件夹：demo_pkg/demo_pkg/，存放 .py 源码与 __init__.py
标准操作命令流程
bash
运行
# 1. 进入工作空间根目录
cd ~/ros_code/ros2_ws
# 2. 清理旧编译缓存（排错关键）
rm -rf build install log
# 3. 编译指定功能包
colcon build --packages-select demo_pkg
# 4. 加载工作空间环境（每次编译/新开终端必执行）
source install/setup.bash
# 5. 运行节点
ros2 run 包名 节点别名
环境加载说明
source install/setup.bash 仅对当前终端窗口生效，关闭终端失效；
不执行该命令，ROS2 无法识别自定义功能包与节点。
2. 编写 Python 版 HelloWorld 节点
标准完整代码 hello_node.py
python
运行
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node

class HelloNode(Node):
    def __init__(self):
        # 父类初始化，设置节点名称
        super().__init__('hello_node')
        # 日志打印输出
        self.get_logger().info("Hello World!")

# 【重点】main函数必须顶格，与class同级，不能缩进在类内部
def main(args=None):
    rclpy.init(args=args)
    node = HelloNode()
    rclpy.spin(node)  # 循环阻塞，维持节点运行
    node.destroy_node()
    rclpy.shutdown()

# 程序入口判定
if __name__ == '__main__':
    main()
代码分层讲解
导入 ROS2 基础库；
自定义节点类继承 Node，构造函数完成初始化与日志输出；
main() 函数：ROS2 节点标准入口，完成初始化、创建节点、循环运行、资源释放；
if __name__ == '__main__'：Python 脚本直接运行时调用 main 函数。
3. setup.py 节点注册配置
核心字段 entry_points 用于绑定终端启动命令与代码内 main 函数：
python
运行
entry_points={
    'console_scripts': [
        # 格式：终端启动别名 = 包名.文件名:main函数
        'hello = demo_pkg.hello_node:main',
    ],
},
hello：运行命令 ros2 run demo_pkg hello 中的节点别名；
demo_pkg.hello_node：包名。源码文件名（不带 .py）；
:main：绑定代码中顶层 def main() 函数。
4. 节点运行状态查看指令
前置要求：保持一个终端运行节点不关闭，新开第二个终端执行查看命令；
新开终端操作：
bash
运行
cd ~/ros_code/ros2_ws
source install/setup.bash
# 查看所有正在运行的节点
ros2 node list
# 查看单个节点详细信息（话题/服务/参数）
ros2 node info /hello_node
二、遇到的全部报错问题与完整解决方案
问题 1：No executable found
报错场景
执行 ros2 run demo_pkg hello_node 提示无可用可执行程序
根因
setup.py 中 entry_points 缩进层级错误，与上方参数字段未对齐，编译未注册节点；
旧编译缓存 build/install/log 残留，新配置未覆盖旧文件；
内层源码文件夹缺失空文件 __init__.py，Python 模块加载失败。
解决步骤
修正 setup.py 缩进，保证 entry_points={} 和 extras_require={} 左对齐；
清空全部缓存：rm -rf build install log；
创建模块初始化文件：touch src/demo_pkg/demo_pkg/__init__.py；
重新编译 + source 环境后运行。
问题 2：AttributeError: module 'demo_pkg.hello_node' has no attribute 'main'
报错场景
节点注册成功，但运行时报找不到 main 函数
根因
代码缩进错误：def main() 被缩进写在 class HelloNode 类内部，变成类方法，不是顶层全局函数，ROS2 无法调用。
解决步骤
将 def main()、if __name__ == '__main__': 两行代码完全顶格，和 class 类同级；
本地单独测试脚本验证：cd src/demo_pkg/demo_pkg && python3 hello_node.py；
清空缓存重新编译加载环境。
问题 3：执行 ros2 pkg executables demo_pkg 输出旧别名 hello，和 setup.py 配置不匹配
根因
未完整删除编译缓存，系统持续读取上一版 setup.py 生成的旧脚本，新配置不生效。
解决
完整执行清理、重编译流程：
bash
运行
rm -rf build install log
colcon build --packages-select demo_pkg
source install/setup.bash
问题 4：新开终端 ros2 node list 看不到运行的节点
根因
新终端未执行 source install/setup.bash，未加载工作空间环境；
运行节点的终端已经闪退、节点停止运行；
两个终端不在同一个工作空间目录。
解决
新开终端先执行：
bash
运行
cd ~/ros_code/ros2_ws
source install/setup.bash
问题 5：编译警告 Ament prefix path ... doesn't exist
根因
环境变量残留了已删除的旧 install 目录路径。
解决
执行 rm -rf build install log 全量清理缓存后重新编译，警告自动消除。
三、避坑总结（高频踩坑点）
缩进是 Python+setup.py 第一大坑
main() 不能写在 class 内部；
setup.py 的 entry_points 必须和同级参数字段左对齐，缩进错误直接注册失败。
修改代码 /setup.py 后必须三步骤
删缓存 → 重新编译 → source 刷新环境，缺一不可。
终端环境隔离
每个新打开的终端都要手动 source install/setup.bash，否则 ROS2 识别不到自定义包。
目录强制规范
Python 功能包内层必须存在 __init__.py 空文件，否则模块加载异常。
运行命令和 setup.py 绑定
ros2 run 包名 xxx 里的 xxx 必须和 entry_points 里注册的别名完全一致。