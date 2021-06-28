FROM ros:melodic

# These values will be overrided by `docker run --env <key>=<value>` command
ENV ROS_IP 127.0.0.1
ENV ROS_MASTER_URI http://127.0.0.1:11311

# Install some basic dependencies
RUN apt-get update && apt-get -y upgrade && apt-get -y install \
  curl nano ssh python-pip python3-pip \
  && rm -rf /var/lib/apt/lists/*

# Set root password
RUN echo 'root:root' | chpasswd

# Permit SSH root login
RUN sed -i 's/#*PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config

ARG FR_VERBOSE="false"
ARG FR_VERSION="latest"

# Override these variables when running the image
ENV FR_ACCOUNT="INSTALL_ONLY"
ENV FR_DEVICE="GENERIC_DEVICE"
ENV FR_TOKEN="INSTALL_ONLY_DEVICE_TOKEN"
ENV FR_SECRET=""

ENV FR_INSTALL_ELEMENTS="warn_no_fail,no_credentials,service_none,webrtc"
ENV FR_URL="https://api.freedomrobotics.ai/accounts/${FR_ACCOUNT}/devices/${FR_DEVICE}/installscript?\
mc_token=${FR_TOKEN}\
&code_version=${FR_VERSION}\
&install_elements=${FR_INSTALL_ELEMENTS}\
&auto_install_deps=true\
&ppa_is_allowed=true\
&verbose=${FR_VERBOSE}"

# Install Freedom agent
RUN curl -s "${FR_URL}" | python \
  && rm -rf /var/lib/apt/lists/* \
  && rm -rf /root/.cache/pip/* 


RUN mkdir -p /catkin_ws/src 

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-melodic-cv-bridge

RUN apt-get update && apt-get install -y --no-install-recommends \
    python-pip python3 python3-pip python3-setuptools \
    python3-cffi python3-numpy python-catkin-tools \
    gfortran python3-opencv python3-scipy && \
    rm -rf /var/lib/apt/lists/*

# Install catkin-tools
RUN apt-get update && apt-get install -y python-catkin-tools \
  && rm -rf /var/lib/apt/lists/*


RUN apt-get update && apt-get install -y --no-install-recommends python-opencv && \
    rm -rf /var/lib/apt/lists/* && mkdir -p /mars && \
    cd /catkin_ws/src && git clone https://github.com/canyagmur/erc_2020.git && \
    chmod +x /catkin_ws/src/erc_2020/rover_really_marsyard.py && chmod +x /catkin_ws/src/erc_2020/argparser.py

RUN apt-get update && rm /etc/ros/rosdep/sources.list.d/20-default.list && rosdep init && rosdep update && \
    rosdep install --from-paths /catkin_ws/src/ --ignore-src --rosdistro melodic -r -y && \
    rm -rf /var/lib/apt/lists/*

# Build packages
RUN /bin/bash -c '. /opt/ros/melodic/setup.bash; cd /catkin_ws; catkin build -DCMAKE_BUILD_TYPE=Release'

RUN echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc && \
    echo "source /catkin_ws/devel/setup.bash" >> ~/.bashrc

# Install Freedom scripts
RUN mkdir /freedom
COPY ./freedom_register.py /freedom/
COPY ./freedom_keep_alive.py /freedom/
COPY ./entrypoint.sh /freedom/
RUN  chmod +x /freedom/freedom_register.py /freedom/freedom_keep_alive.py /freedom/entrypoint.sh

ENTRYPOINT ["/freedom/entrypoint.sh"]
