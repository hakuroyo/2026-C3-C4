# C3 C4
## Dev Container

对宿主机系统几乎无要求，只需要能正常运行 Docker，并能拉取所需镜像。

### VS Code

1. 安装 Docker 和 VS Code 的 Dev Containers 插件。
2. 用 VS Code 打开本仓库，点击提示中的 **Reopen in Container**。
3. 进入容器后，源码位于 `/home/rosdev/ros2_ws/src`，工作区位于 `/home/rosdev/ros2_ws`。

### CLion

1. 安装 Docker，并在 CLion 中打开本仓库。
2. 使用 Dev Containers 打开 `.devcontainer/devcontainer.json`。
3. 等待镜像构建完成后，在容器内进行开发、构建和调试。

容器启动后会自动开启 noVNC GUI，端口为 `1111`；需要查看 Gazebo 等图形界面时，在浏览器打开转发的 noVNC 地址即可。

运行 Gazebo（三种方式）
```shell
gazebo
gazebo-novnc
ros2 launch gazebo_ros gazebo.launch.py
```
需要加载世界时：
```shell
gazebo world:=src/test.world
gazebo-novnc --pause src/test.world
ros2 launch gazebo_ros gazebo.launch.py --pause src/test.world
```
注意：除`src`目录下的文件以外其它目录下的文件在 rebuild container 时不会保留。