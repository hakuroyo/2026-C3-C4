FROM --platform=linux/amd64 ros:humble

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
    ros-humble-desktop \
    xvfb \
    x11-utils \
    x11vnc \
    novnc \
    websockify \
    openbox \
    xterm \
    wmctrl \
    xdotool \
    mesa-utils \
    libgl1-mesa-dri \
    libgl1-mesa-glx \
    dbus-x11 \
    libopencv-dev \
    python3-opencv \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV DISPLAY=:1
ENV LIBGL_ALWAYS_SOFTWARE=1
ENV LIBGL_DRI3_DISABLE=1
ENV QT_X11_NO_MITSHM=1
ENV QT_QPA_PLATFORM=xcb
ENV QT_OPENGL=software
ENV QT_QUICK_BACKEND=software
ENV XDG_RUNTIME_DIR=/tmp/runtime-rosdev
ENV GAZEBO_MASTER_URI=http://127.0.0.1:11345
ENV GAZEBO_IP=127.0.0.1
ENV IGN_IP=127.0.0.1
EXPOSE 1111

RUN printf '%s\n' \
    '#!/usr/bin/env bash' \
    'set -e' \
    '' \
    'export DISPLAY="${DISPLAY:-:1}"' \
    '' \
    'if ! pgrep -f "Xvfb ${DISPLAY}" >/dev/null 2>&1; then' \
    '    Xvfb "$DISPLAY" -screen 0 "${VNC_RESOLUTION:-1600x1000x24}" -dpi 96 -ac +extension GLX +render -noreset >/tmp/xvfb.log 2>&1 &' \
    'fi' \
    '' \
    'for _ in $(seq 1 50); do' \
    '    xdpyinfo -display "$DISPLAY" >/dev/null 2>&1 && break' \
    '    sleep 0.1' \
    'done' \
    '' \
    'if ! pgrep -x openbox >/dev/null 2>&1; then' \
    '    openbox >/tmp/openbox.log 2>&1 &' \
    'fi' \
    '' \
    'if [ "${START_XTERM:-1}" = "1" ] && ! pgrep -f "xterm -title noVNC" >/dev/null 2>&1; then' \
    '    xterm -title noVNC -geometry 100x28+24+24 >/tmp/xterm.log 2>&1 &' \
    'fi' \
    '' \
    'if ! pgrep -f "x11vnc.*${DISPLAY}" >/dev/null 2>&1; then' \
    '    x11vnc -display "$DISPLAY" -rfbport 5900 -forever -shared -nopw -quiet >/tmp/x11vnc.log 2>&1 &' \
    'fi' \
    '' \
    'if ! pgrep -f "websockify.*1111" >/dev/null 2>&1; then' \
    '    websockify --web=/usr/share/novnc/ 0.0.0.0:1111 localhost:5900 >/tmp/novnc.log 2>&1 &' \
    'fi' \
    '' \
    'exec "$@"' \
    > /usr/local/bin/start-novnc \
    && chmod +x /usr/local/bin/start-novnc \
    && printf '%s\n' \
    '<!doctype html>' \
    '<meta http-equiv="refresh" content="0; url=/vnc.html?autoconnect=true&resize=scale&reconnect=true">' \
    '<a href="/vnc.html?autoconnect=true&resize=scale&reconnect=true">Open noVNC</a>' \
    > /usr/share/novnc/index.html

RUN printf '%s\n' \
    '#!/usr/bin/env bash' \
    'set -e' \
    '' \
    'export DISPLAY="${DISPLAY:-:1}"' \
    'export LIBGL_ALWAYS_SOFTWARE="${LIBGL_ALWAYS_SOFTWARE:-1}"' \
    'export LIBGL_DRI3_DISABLE="${LIBGL_DRI3_DISABLE:-1}"' \
    'export QT_X11_NO_MITSHM="${QT_X11_NO_MITSHM:-1}"' \
    'export QT_QPA_PLATFORM="${QT_QPA_PLATFORM:-xcb}"' \
    'export QT_OPENGL="${QT_OPENGL:-software}"' \
    'export QT_QUICK_BACKEND="${QT_QUICK_BACKEND:-software}"' \
    'export XDG_RUNTIME_DIR="${XDG_RUNTIME_DIR:-/tmp/runtime-rosdev}"' \
    'export GAZEBO_MASTER_URI="${GAZEBO_MASTER_URI:-http://127.0.0.1:11345}"' \
    'export GAZEBO_IP="${GAZEBO_IP:-127.0.0.1}"' \
    'export IGN_IP="${IGN_IP:-127.0.0.1}"' \
    'mkdir -p "$XDG_RUNTIME_DIR"' \
    'chmod 700 "$XDG_RUNTIME_DIR"' \
    '' \
    'gzserver --verbose "$@" > /tmp/gzserver.log 2>&1 &' \
    'gzserver_pid="$!"' \
    'sleep 1' \
    'dbus-run-session -- gzclient --verbose > /tmp/gzclient.log 2>&1 &' \
    'gzclient_pid="$!"' \
    '' \
    'for _ in $(seq 1 80); do' \
    '    window_ids="$(xdotool search --pid "$gzclient_pid" 2>/dev/null || true)"' \
    '    [ -n "$window_ids" ] && break' \
    '    window_ids="$(xdotool search --class gzclient 2>/dev/null || true)"' \
    '    [ -n "$window_ids" ] && break' \
    '    window_ids="$(wmctrl -lx | awk '"'"'BEGIN{IGNORECASE=1} /(gzclient|gazebo)/ && $0 !~ /Qt Selection Owner/ {print $1}'"'"' || true)"' \
    '    [ -n "$window_ids" ] && break' \
    '    sleep 0.25' \
    'done' \
    '' \
    'if [ -n "${window_ids:-}" ]; then' \
    '    for window_id in $window_ids; do' \
    '        title="$(xdotool getwindowname "$window_id" 2>/dev/null || true)"' \
    '        case "$title" in *"Qt Selection Owner"*) continue ;; esac' \
    '        wmctrl -i -r "$window_id" -b remove,hidden,maximized_vert,maximized_horz 2>/dev/null || true' \
    '        wmctrl -i -r "$window_id" -e 0,40,40,1180,720 2>/dev/null || true' \
    '        wmctrl -i -a "$window_id" 2>/dev/null || true' \
    '    done' \
    'else' \
    '    echo "No gzclient window found on $DISPLAY" >&2' \
    '    echo "Run list-x-windows for the current X window tree." >&2' \
    'fi' \
    '' \
    'wait "$gzclient_pid"' \
    'kill "$gzserver_pid" >/dev/null 2>&1 || true' \
    > /usr/local/bin/gazebo-novnc \
    && chmod +x /usr/local/bin/gazebo-novnc

RUN printf '%s\n' \
    '#!/usr/bin/env bash' \
    'set -e' \
    '' \
    'export DISPLAY="${DISPLAY:-:1}"' \
    'window_ids="$(xdotool search --class gzclient 2>/dev/null || true)"' \
    'if [ -z "$window_ids" ]; then' \
    '    window_ids="$(wmctrl -lx | awk '"'"'BEGIN{IGNORECASE=1} /(gzclient|gazebo)/ && $0 !~ /Qt Selection Owner/ {print $1}'"'"' || true)"' \
    'fi' \
    'if [ -z "$window_ids" ]; then' \
    '    echo "No gzclient/Gazebo windows found on $DISPLAY" >&2' \
    '    echo "Run list-x-windows to inspect the current X window tree." >&2' \
    '    exit 1' \
    'fi' \
    'for window_id in $window_ids; do' \
    '    title="$(xdotool getwindowname "$window_id" 2>/dev/null || true)"' \
    '    case "$title" in *"Qt Selection Owner"*) continue ;; esac' \
    '    wmctrl -i -r "$window_id" -b remove,hidden,maximized_vert,maximized_horz || true' \
    '    wmctrl -i -r "$window_id" -e 0,40,40,1180,720 || true' \
    '    wmctrl -i -a "$window_id" || true' \
    'done' \
    > /usr/local/bin/show-gazebo-windows \
    && chmod +x /usr/local/bin/show-gazebo-windows

RUN printf '%s\n' \
    '#!/usr/bin/env bash' \
    'set -e' \
    '' \
    'export DISPLAY="${DISPLAY:-:1}"' \
    'for _ in $(seq 1 80); do' \
    '    window_ids="$(xdotool search --pid "$1" 2>/dev/null || true)"' \
    '    [ -n "$window_ids" ] && break' \
    '    window_ids="$(xdotool search --class gzclient 2>/dev/null || true)"' \
    '    [ -n "$window_ids" ] && break' \
    '    window_ids="$(wmctrl -lx | awk '"'"'BEGIN{IGNORECASE=1} /(gzclient|gazebo)/ && $0 !~ /Qt Selection Owner/ {print $1}'"'"' || true)"' \
    '    [ -n "$window_ids" ] && break' \
    '    sleep 0.25' \
    'done' \
    '' \
    'if [ -n "${window_ids:-}" ]; then' \
    '    for window_id in $window_ids; do' \
    '        title="$(xdotool getwindowname "$window_id" 2>/dev/null || true)"' \
    '        case "$title" in *"Qt Selection Owner"*) continue ;; esac' \
    '        wmctrl -i -r "$window_id" -b remove,hidden,maximized_vert,maximized_horz 2>/dev/null || true' \
    '        wmctrl -i -r "$window_id" -e 0,40,40,1180,720 2>/dev/null || true' \
    '        wmctrl -i -a "$window_id" 2>/dev/null || true' \
    '    done' \
    'fi' \
    > /usr/local/bin/place-gzclient-window \
    && chmod +x /usr/local/bin/place-gzclient-window

RUN printf '%s\n' \
    '#!/usr/bin/env bash' \
    'set -e' \
    '' \
    'export DISPLAY="${DISPLAY:-:1}"' \
    'echo "Processes:"' \
    'ps -eo pid,ppid,comm,args | grep -E "gzserver|gzclient|gazebo|Xvfb|openbox|x11vnc|websockify" | grep -v grep || true' \
    'echo' \
    'echo "wmctrl windows:"' \
    'wmctrl -lx || true' \
    'echo' \
    'echo "gzclient log:"' \
    'tail -n 120 /tmp/gzclient.log 2>/dev/null || true' \
    'echo' \
    'echo "gzserver log:"' \
    'tail -n 80 /tmp/gzserver.log 2>/dev/null || true' \
    'echo' \
    'echo "xwininfo tree:"' \
    'xwininfo -root -tree | sed -n "1,120p"' \
    > /usr/local/bin/list-x-windows \
    && chmod +x /usr/local/bin/list-x-windows

# 添加 OSRF Gazebo 源
RUN curl -sSL http://packages.osrfoundation.org/gazebo.gpg | apt-key add -

RUN echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable \
$(lsb_release -cs) main" > /etc/apt/sources.list.d/gazebo-stable.list

# 安装 Gazebo Classic + ROS 接口
RUN apt-get update && apt-get install -y \
    gazebo \
    ros-humble-gazebo-ros-pkgs \
    ros-humble-gazebo-plugins

RUN dpkg-divert --rename --add /usr/bin/gazebo \
    && printf '%s\n' \
    '#!/usr/bin/env bash' \
    'set -e' \
    '' \
    'if [ "${GAZEBO_USE_ORIGINAL:-0}" = "1" ]; then' \
    '    exec /usr/bin/gazebo.distrib "$@"' \
    'fi' \
    '' \
    'start-novnc true' \
    'exec gazebo-novnc "$@"' \
    > /usr/bin/gazebo \
    && chmod +x /usr/bin/gazebo

RUN dpkg-divert --rename --add /usr/bin/gzclient \
    && printf '%s\n' \
    '#!/usr/bin/env bash' \
    'set -e' \
    '' \
    'if [ "${GZCLIENT_USE_ORIGINAL:-0}" = "1" ]; then' \
    '    exec /usr/bin/gzclient.distrib "$@"' \
    'fi' \
    '' \
    'export DISPLAY="${DISPLAY:-:1}"' \
    'export LIBGL_ALWAYS_SOFTWARE="${LIBGL_ALWAYS_SOFTWARE:-1}"' \
    'export LIBGL_DRI3_DISABLE="${LIBGL_DRI3_DISABLE:-1}"' \
    'export QT_X11_NO_MITSHM="${QT_X11_NO_MITSHM:-1}"' \
    'export QT_QPA_PLATFORM="${QT_QPA_PLATFORM:-xcb}"' \
    'export QT_OPENGL="${QT_OPENGL:-software}"' \
    'export QT_QUICK_BACKEND="${QT_QUICK_BACKEND:-software}"' \
    'export XDG_RUNTIME_DIR="${XDG_RUNTIME_DIR:-/tmp/runtime-rosdev}"' \
    'mkdir -p "$XDG_RUNTIME_DIR"' \
    'chmod 700 "$XDG_RUNTIME_DIR"' \
    '' \
    'start-novnc true' \
    'dbus-run-session -- /usr/bin/gzclient.distrib "$@" &' \
    'gzclient_pid="$!"' \
    'place-gzclient-window "$gzclient_pid" &' \
    'wait "$gzclient_pid"' \
    > /usr/bin/gzclient \
    && chmod +x /usr/bin/gzclient

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
    && colcon build --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
    && cd /
WORKDIR /home/$USERNAME/ros2_ws

# Copy configuration files
RUN echo 'source /opt/ros/'$ROS_DISTRO'/setup.bash' >> /home/$USERNAME/.bashrc \
    && echo 'source /home/'$USERNAME'/ros2_ws/install/setup.bash' >> /home/$USERNAME/.bashrc

# Setup entrypoint
COPY ./ros_entrypoint.sh /
ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["start-novnc", "bash"]
