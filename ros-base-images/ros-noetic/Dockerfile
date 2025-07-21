#FROM osrf/ros:noetic-desktop-full
FROM ghcr.io/sloretz/ros:noetic-desktop-full
# 定义用户和用户组
ARG USERNAME=Nagisa
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG GROUP_NAME=wheel
ENV DEBIAN_FRONTEND=noninteractive

# 步骤1：创建专用密钥目录，直接下载ROS密钥并独立存储（跳过apt-key）
RUN mkdir -p /etc/apt/keyrings \
    # 下载ROS密钥并保存为独立文件（使用国内镜像加速）
    && curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/ros/ros.key | gpg --dearmor -o /etc/apt/keyrings/ros.gpg \
    # 设置安全权限（避免权限问题导致验证失败）
    && chmod 644 /etc/apt/keyrings/ros.gpg \
    # 添加密钥指纹验证（防止篡改或下载失败）
    && (gpg --show-keys /etc/apt/keyrings/ros.gpg | grep -q "F42ED6FBAB17C654" || (echo "❌ 密钥指纹验证失败！"; exit 1)) \
    # 更新ROS仓库配置：使用国内镜像源 + 显式绑定独立密钥文件
    && echo "deb [signed-by=/etc/apt/keyrings/ros.gpg] https://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros.list \
    # 更新仓库并安装所需软件（清理缓存减小镜像体积）
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        net-tools nautilus bash-completion clangd git \
        gedit gdb nano software-properties-common wget curl vim \
    && rm -rf /var/lib/apt/lists/*  # 立即清理APT缓存

RUN groupadd --gid $USER_GID ${GROUP_NAME} \
    && useradd --uid $USER_UID --gid $USER_GID -m -s /bin/bash $USERNAME \
    # 配置密码
    && echo "$USERNAME:password" | chpasswd \
    # 允许用户使用 sudo
    # && usermod -aG sudo $USERNAME \
    # && echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers \
    # 允许wheel组的用户使用 sudo
    && usermod -aG ${GROUP_NAME} $USERNAME \
    && echo "%${GROUP_NAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers \
    # 使新用户的 `.bashrc` 文件生效
    && chown $USERNAME:$GROUP_NAME /home/$USERNAME/.bashrc
USER root
RUN  add-apt-repository ppa:fish-shell/release-3 -y \
    &&  apt-get update &&  apt-get install -y fish fzf 
COPY packages/fish /home/${USERNAME}/.config/fish
WORKDIR /home/${USERNAME}
USER ${USERNAME}
RUN mkdir packages && cd packages && git clone https://github.com/Livox-SDK/Livox-SDK.git && cd Livox-SDK/build \
    && cmake .. && make -j8 && sudo make install \
    && echo 'export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH' >> /home/${USERNAME}/.bashrc 
#安装雷达ros驱动
RUN . /opt/ros/noetic/setup.sh \
    && cd packages && mkdir ws_livox && cd ws_livox && mkdir src && cd src \
    && git clone https://github.com/Livox-SDK/livox_ros_driver.git && cd .. && catkin_make \
    && echo 'source ~/packages/ws_livox/devel/setup.bash' >> /home/Elaina/.bashrc \
    && git config --global core.filemode false
