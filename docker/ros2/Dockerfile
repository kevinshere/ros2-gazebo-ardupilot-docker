# Using Humble Desktop Full
FROM osrf/ros:humble-desktop-full

ARG user_id
ARG USER_NAME=dockeruser
ARG ROS_WS=/home/${USER_NAME}/dev_ws
ENV GZ_VERSION=harmonic

ENV DEBIAN_FRONTEND=noninteractive

# Use the variable with a $ sign
RUN if getent passwd ${user_id}; then \
    # If UID exists, rename the existing user to our USER_NAME
    existing_user=$(getent passwd ${user_id} | cut -d: -f1); \
    usermod -l ${USER_NAME} -m -d /home/${USER_NAME} ${existing_user}; \
    groupmod -n ${USER_NAME} $(getent group ${user_id} | cut -d: -f1); \
    else \
    # If UID doesn't exist, create it from scratch
    useradd -m --uid ${user_id} -s /bin/bash ${USER_NAME}; \
    fi \
    && echo "${USER_NAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USER_NAME} \
    && chmod 0440 /etc/sudoers.d/${USER_NAME}

# Install System & Gazebo Bridge Dependencies
RUN apt-get update && apt-get install -y \
    libx11-6 libx11-xcb1 libxcb-icccm4 libxcb-image0 libxcb-keysyms1 \
    libxcb-render-util0 libxcb-xinerama0 libxcb-xkb1 libxkbcommon-x11-0 \
    libxcb-cursor0 libxcb-util1 libxcb-xinput0 \
    qt6-base-dev libqt6gui6 libqt6widgets6 \
    libglu1-mesa libegl1 libgl1-mesa-dri libglx0 libopengl0 \
    lsb-release sudo tzdata git python3-pip python-is-python3 \
    default-jre bash-completion nano less xterm wget\
    # The following is to install gstreamer cli commands which can be used to provide a video feed
    gstreamer1.0-tools \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-libav \
    gstreamer1.0-gl \
    libopencv-dev \
    libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev \
    # ros-humble-ros-gz-* \
    && rm -rf /var/lib/apt/lists/*
# ros-humble-ros-gz-* use this option if you want a generic install instead of source, however, source allows for custom code modification

USER ${USER_NAME}
WORKDIR ${ROS_WS}
RUN mkdir -p src

# ROS 2 with ArduPilot
RUN vcs import --recursive --input https://raw.githubusercontent.com/ArduPilot/ardupilot/master/Tools/ros2/ros2.repos src \
    && sudo apt update \ 
    && rosdep update \ 
    && . /opt/ros/${ROS_DISTRO}/setup.sh \
    && rosdep install --from-paths src --ignore-src -r -y 

# Install ArduPilot Prerequisites
USER ${USER_NAME}
WORKDIR ${ROS_WS}/src/ardupilot
RUN SKIP_AP_EXT_ENV=0 SKIP_AP_GRAPHIC_ENV=0 SKIP_AP_COV_ENV=1 SKIP_AP_GIT_CHECK=1 \
    DO_AP_STM_ENV=1 AP_DOCKER_BUILD=1 \
    USER=${USER_NAME} \
    ./Tools/environment_install/install-prereqs-ubuntu.sh -y

# Build Micro-XRCE-DDS-Gen
# USER root
WORKDIR ${ROS_WS}
RUN git clone --recurse-submodules https://github.com/ardupilot/Micro-XRCE-DDS-Gen.git \
    && cd Micro-XRCE-DDS-Gen \
    && ./gradlew assemble 

ENV PATH="${PATH}:${ROS_WS}/Micro-XRCE-DDS-Gen/scripts"

# building the workspace
WORKDIR ${ROS_WS}
RUN . /opt/ros/${ROS_DISTRO}/setup.sh \
    && colcon build --packages-up-to ardupilot_dds_tests ardupilot_sitl

# Gazebo installation
RUN vcs import --recursive --input https://raw.githubusercontent.com/ArduPilot/ardupilot_gz/main/ros2_gz.repos src 

# Install Gazebo Harmonic Plugin Dependencies & Build Plugin for ArduPilot with Gazebo
USER root
RUN wget https://packages.osrfoundation.org/gazebo.gpg -O /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null \
    && apt update && apt install -y \
    libgz-sim8-dev \
    rapidjson-dev \
    && rm -rf /var/lib/apt/lists/*

RUN wget https://raw.githubusercontent.com/osrf/osrf-rosdep/master/gz/00-gazebo.list -O /etc/ros/rosdep/sources.list.d/00-gazebo.list \
    && rosdep update

USER ${USER_NAME}
WORKDIR ${ROS_WS}
RUN . /opt/ros/${ROS_DISTRO}/setup.sh \
    && sudo apt update \
    && rosdep update \
    && rosdep install --from-paths src --ignore-src -y \
    && colcon build --packages-up-to ardupilot_gz_bringup

# Configure Shell, Sourcing, and PATHS
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc \
    && echo "export PATH=\$PATH:${ROS_WS}/src/ardupilot/Tools/autotest" >> ~/.bashrc \
    && echo "export PATH=\$PATH:${ROS_WS}/src/Micro-XRCE-DDS-Gen/scripts" >> ~/.bashrc \
    && echo "export GZ_SIM_SYSTEM_PLUGIN_PATH=${ROS_WS}/install/ardupilot_gazebo/build:\${GZ_SIM_SYSTEM_PLUGIN_PATH}" >> ~/.bashrc \
    && echo "export GZ_SIM_RESOURCE_PATH=${ROS_WS}/src/ardupilot_gazebo/models:${ROS_WS}/src/ardupilot_gazebo/worlds:\${GZ_SIM_RESOURCE_PATH}" >> ~/.bashrc \
    && echo "export PATH=\"/home/${USER_NAME}/.local/bin:\$PATH\"" >> ~/.bashrc \
    && echo "if [ -f ${ROS_WS}/install/setup.bash ]; then source ${ROS_WS}/install/setup.bash; fi" >> ~/.bashrc

# The Simplified Alias
RUN echo 'alias launch_sitl="ros2 launch ardupilot_sitl sitl_dds_udp.launch.py transport:=udp4 synthetic_clock:=True wipe:=False model:=quad"' >> ~/.bashrc

WORKDIR ${ROS_WS}
USER ${USER_NAME}
ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["sleep", "infinity"]
