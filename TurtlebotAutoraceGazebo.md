## Criando ambiente com ROS Kinetic

Vamos iniciar instalando o Ubuntu 16.04, com a ISO disponível em https://releases.ubuntu.com/16.04/ubuntu-16.04.7-desktop-amd64.iso

Vamos construir uma VM do VirtualBox com todo o ambiente configurado.

### Instalação do ROS

A primeira coisa a fazer é adicionar os pacotes do ROS aos repositórios APT do sistema.

```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

sudo apt-get install curl

curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

Depois disso, vamos atualizar a lista de pacotes e instalar a versão completa do ROS Kinetic Desktop

```
sudo apt-get update

sudo apt-get install -y ros-kinetic-desktop-full
```

Também precisamos adicionar o setup do ambiente ROS ao .bashrc, para que os pacotes estejam carregados no ambiente, mesmo ao abrir um novo terminal.

```
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Atualização do Gazebo

O pacote de simulação "Gazebo", que acompanha o ROS Kinetic por padrão, utiliza uma versão antiga que está quebrada, para que possamos utilizar adequadamente este ambiente de simulação, vamos atualizar o pacote para a versão 7.16.

Vamos adicionar o repositório APT do Gazebo, atualizar nossa lista de pacotes e fazer upgrade do gazebo atualmente instalado.

```
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'

wget https://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -

sudo apt-get update

sudo apt-get remove -y gazebo7 
sudo apt-get install -y gazebo7 ros-kinetic-gazebo-ros-pkgs
```

### Pacotes do Turtlebot 

Vamos instalar as dependencias adicionais para o funcionamento do Turtlebot

```
sudo apt-get install -y ros-kinetic-joy ros-kinetic-teleop-twist-joy ros-kinetic-teleop-twist-keyboard ros-kinetic-laser-proc ros-kinetic-rgbd-launch ros-kinetic-depthimage-to-laserscan ros-kinetic-rosserial-arduino ros-kinetic-rosserial-python ros-kinetic-rosserial-server ros-kinetic-rosserial-client ros-kinetic-rosserial-msgs ros-kinetic-amcl ros-kinetic-map-server ros-kinetic-move-base ros-kinetic-urdf ros-kinetic-xacro ros-kinetic-compressed-image-transport ros-kinetic-rqt-image-view ros-kinetic-gmapping ros-kinetic-navigation ros-kinetic-interactive-markers
```

Agora iremos criar nosso ambiente de trabalho catkin

```
mkdir -p ~/catkin_ws/src/
cd ~/catkin_ws/src/

sudo apt-get install git -y
git clone -b kinetic-devel https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
git clone -b kinetic-devel https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone -b kinetic-devel https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
git clone -b kinetic-devel https://github.com/ROBOTIS-GIT/turtlebot3_autorace.git
git clone https://github.com/falfab/turtlebot3_autorace_simulation

catkin_make

echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### Rodando o projeto 

Cada um dos blocos abaixo deve ser executado em um novo terminal, seguindo esta sequência

```
export SVGA_VGPU10=0
roslaunch turtlebot3_gazebo turtlebot3_autorace.launch
```

```
roslaunch turtlebot3_gazebo turtlebot3_autorace_mission.launch
```

```
export GAZEBO_MODE=true
export AUTO_IN_CALIB=action
roslaunch turtlebot3_autorace_camera turtlebot3_autorace_intrinsic_camera_calibration.launch
```

```
export AUTO_EX_CALIB=action
export AUTO_DT_CALIB=action
export TURTLEBOT3_MODEL=burger
roslaunch turtlebot3_autorace_core turtlebot3_autorace_core.launch
```

```
rostopic pub -1 /core/decided_mode std_msgs/UInt8 "data: 2"
```