# WSL

The Windows Subsystem for Linux if powerfull, but tricky. If you're using Docker Desktop, the trickiest part is running high-consuming rendering application (within a container) using your favourite GPU. Otherwise, even simple programs like Rviz! will overload your CPU.

After some investigations we found a solution.

## ROS1

```dockerfile
FROM osrf/ros:noetic-desktop

ARG DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

RUN apt update -y && apt upgrade -y && apt -y install \
    libxext-dev \
    libx11-dev \
    libglvnd-dev \
    libglx-dev \
    libgl1-mesa-dev \
    libgl1-mesa-glx \
    libgl1-mesa-dri \
    libegl1-mesa-dev \
    libgles2-mesa-dev \
    freeglut3-dev \
    mesa-utils \
    mesa-utils-extra

RUN apt install -y software-properties-common && add-apt-repository ppa:kisak/turtle -y && apt update && apt upgrade -y

ENV MESA_D3D12_DEFAULT_ADAPTER_NAME=NVIDIA
ENV LD_LIBRARY_PATH=/usr/lib/wsl/lib
ENV LIBVA_DRIVER_NAME=d3d12
```

and test with

    docker run --device /dev/dxg --device /dev/dri/card0 --device /dev/dri/renderD128 --env DISPLAY=$DISPLAY --env WAYLAND_DISPLAY=$WAYLAND_DISPLAY --env PULSE_SERVER=$PULSE_SERVER --volume /tmp/.X11-unix:/tmp/.X11-unix --volume /mnt/wslg:/mnt/wslg --volume /usr/lib/wsl:/usr/lib/wsl -dit --gpus=all --name noetic wsl2-opengl:noetic

## ROS2

Way simpler

    docker run --device /dev/dxg --device /dev/dri/card0 --device /dev/dri/renderD128 --env DISPLAY=$DISPLAY --env WAYLAND_DISPLAY=$WAYLAND_DISPLAY --env PULSE_SERVER=$PULSE_SERVER --env MESA_D3D12_DEFAULT_ADAPTER_NAME=NVIDIA --env LD_LIBRARY_PATH=/usr/lib/wsl/lib --env LIBVA_DRIVER_NAME=d3d12 --volume /tmp/.X11-unix:/tmp/.X11-unix --volume /mnt/wslg:/mnt/wslg --volume /usr/lib/wsl:/usr/lib/wsl -dit --gpus=all --name jazzy osrf/ros:jazzy-desktop

## Clock

If you encounter any issue with ROS time, which keeps resetting, look at this [repo](https://github.com/matthewnourse/polite-hwclock-hctosys). Of course you need to install the service within the WSL distribution that serves as Docker Desktop backend.

## Networking

If you need to contact some windows applications or some physical device via TCP, you'll need to open the `WSL Settings` app. Set the network mode to `Mirrored` and enable the host address loopback option. Then restart WSL.
