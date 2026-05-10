FROM --platform=linux/amd64 ros:humble-desktop

ARG USERNAME=rosdev
ARG UID=1000
ARG GID=$UID

# Install some dependencies packages
RUN apt update -q \
    && apt upgrade -q -y \
    && apt install -y --no-install-recommends \
    software-properties-common \
    python3-pip \
    xauth \
    libopencv-dev \
    python3-opencv \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 添加 OSRF Gazebo 源
RUN curl -sSL http://packages.osrfoundation.org/gazebo.gpg | apt-key add -

RUN echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable \
$(lsb_release -cs) main" > /etc/apt/sources.list.d/gazebo-stable.list

# 安装 Gazebo Classic + ROS 接口
RUN apt-get update && apt-get install -y \
    gazebo \
    ros-humble-gazebo-ros-pkgs \
    ros-humble-gazebo-plugins

# Create and switch to user
RUN groupadd -g $GID $USERNAME \
    && useradd -lm -u $UID -g $USERNAME -s /bin/bash $USERNAME \
    && echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
USER $USERNAME

# Create workspace so that user own this directory
RUN mkdir -p /home/$USERNAME/ros2_ws/src \
    && rosdep update \
    && cd /home/rosdev/ros2_ws \
    && rosdep install -i --from-path src --rosdistro humble -y \
    && colcon build \
    && cd /
WORKDIR /home/$USERNAME/ros2_ws

# Copy configuration files
RUN echo 'source /opt/ros/'$ROS_DISTRO'/setup.bash' >> /home/$USERNAME/.bashrc \
    && echo 'source /home/'$USERNAME'/ros2_ws/install/setup.bash' >> /home/$USERNAME/.bashrc

# Setup entrypoint
COPY ./ros_entrypoint.sh /
ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["bash"]
