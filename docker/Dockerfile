################################################################i
# Dockerfile for Arena-Rosnav

# URL: https://github.com/eliastreis/arena-rosnav-3D

# Based on Ubuntu 18.04 + ROS-Melodic-Desktop-Full Version

#################################################################

# This image includes additional meta-packages such for desktop installations than offical image
FROM osrf/ros:melodic-desktop-full

# Change the default shell from /bin/sh to /bin/bash
SHELL ["/bin/bash","-c"]

# 1. Install additinal pkgs
RUN apt-get -y update && apt-get install -y \
    gnutls-bin \
    vim \
    git \
    original-awk \
    python3-pip \
    libqt4-dev \
    libopencv-dev \
    liblua5.2-dev \
    screen \
    python3.6 \
    python3.6-dev \
    libpython3.6-dev \
    python3-scipy \
    python3-catkin-pkg-modules \
    python3-rospkg-modules \
    python3-empy \
    python3-setuptools \
    ros-melodic-navigation \
    ros-melodic-teb-local-planner \
    ros-melodic-mpc-local-planner \
    ros-melodic-nlopt \
    libarmadillo-dev \
&& echo $'\n\
source /opt/ros/melodic/setup.sh' >> /root/.bashrc
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y tzdata \
&& DEBIAN_FRONTEND=noninteractive apt-get install -y keyboard-configuration
RUN apt-get install -y software-properties-common
RUN apt-get install wget

# 2. Prepare virtual environment
RUN pip3 install --upgrade pip \
&& pip3 install virtualenv  virtualenvwrapper\
&& cd /root \
&& mkdir .python_env \
&& echo $'\n\
export WORKON_HOME=/root/.python_env \n\
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3 \n\
export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv \n\
export PYTHONPATH=/root/catkin_ws/src/arena-rosnav-3D:${PYTHONPATH} \n\
export TURTLEBOT3_MODEL=burger \n\
source /usr/local/bin/virtualenvwrapper.sh' >> /root/.bashrc

ENV WORKON_HOME=/root/.python_env \
    VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3 \
    VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv
    
# 3. Create python virtualenv and install dependencies
RUN . /usr/local/bin/virtualenvwrapper.sh \
&& mkvirtualenv --python=python3.6 rosnav \ 
&& workon rosnav \
&& pip3 install --upgrade pip \
&& pip3 install --extra-index-url https://rospypi.github.io/simple/ rospy rosbag tf tf2_ros --ignore-installed \
&& pip3 install pyyaml catkin_pkg netifaces pathlib \
&& pip3 install filelock

# 4. Install Arena-Rosnav repo and compile
RUN git clone https://github.com/Ann-Rachel/arena-rosnav-3D /root/catkin_ws/src/arena-rosnav-3D

RUN . /usr/local/bin/virtualenvwrapper.sh \
&& . /opt/ros/melodic/setup.sh \
&& workon rosnav \
&& pip3 install --upgrade pip \
&& pip3 install gym \
&& pip3 install PyQt5 \
&& pip3 install scipy \
&& cd /root/catkin_ws/src/arena-rosnav-3D \
&& git checkout \
&& rosws update

RUN . /opt/ros/melodic/setup.sh \
&& cd /root/catkin_ws/src \
&& git clone https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git \
&& git clone https://github.com/ROBOTIS-GIT/turtlebot3.git \
&& git clone https://github.com/rst-tu-dortmund/mpc_local_planner \
&& cd ~/catkin_ws && catkin_make -j1

RUN . /opt/ros/melodic/setup.sh \
&& cd /root/catkin_ws \
&& catkin_make -j1 -DCMAKE_BUILD_TYPE=Release -DPYTHON_EXECUTABLE=/usr/bin/python3 \
#&& catkin_make -DCMAKE_BUILD_TYPE=Release -DPYTHON_EXECUTABLE=/usr/bin/python3 \
&& echo $'source /root/catkin_ws/devel/setup.bash' >> /root/.bashrc

# 5. Install geometry2 pkg
RUN . /root/.bashrc \
&& . /opt/ros/melodic/setup.sh \
&& cd /root/catkin_ws/src/arena-rosnav-3D/ \
&& . geometry2_install.sh

# 6. start workon rosnav
RUN echo $'source /root/.python_env/rosnav/bin/activate' >> /root/.bashrc

# 7. Entrypoint
WORKDIR /root/catkin_ws/src/arena-rosnav-3D/
