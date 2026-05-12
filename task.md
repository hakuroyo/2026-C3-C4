1、使用 Gazebo Classic 仿真深度摄像头获得 RGB-D 图像（每 4 帧取 1 个图像）

2、RGB-D 图像经去雾算法处理

3、然后转化为点云图，以`std_msgs/pointcloud2`类型 publish 到`pointcloud`topic 。

启动方式用`launch.py`方式启动。启动时应当唤起：Gazebo 、OpenCV 处理前的实时图像、OpenCV 去雾算法处理后的实时图像、rviz2

Gazebo 加载的`.world`文件里包含：雾、一个正方体、一个负责输出图像的深度摄像头。

rviz2 启动时应已经订阅好`pointcloud`topic（`std_msgs/pointcloud2`类型）