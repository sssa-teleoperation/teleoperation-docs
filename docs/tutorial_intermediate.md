# Tutorials (Intermediate)

Before going on, let's go back to the `docker run hello-world` command. What `docker run` does is create a container from the `hello-world` image. You can create as many containers (completely unrelated) as you want from the same image. 

**Note:** We suggest you the [vscode docker](https://code.visualstudio.com/docs/containers/overview) extension to [explore](https://code.visualstudio.com/docs/containers/overview#_docker-explorer) all containers and images on your system.

You should also have noticed that `docker run` will *download* (or rather `pull`) the `hello-world` image from the web (actually from some known repositories, like `apt` does) if it was not found locally. Try one by one the commands below.
```bash
docker rmi hello-world #will erase the local image
docker pull hello-world #only downloads the latest version of the image.
docker run hello-world #start a container with a random fancy name
docker run --name pippo hello-world #start a container named pippo
docker run --name pluto --rm hello-world #start a container named pippo, automatically remove it on stop (--rm)
docker container ls #list all running container
```

## Dockerfiles

In you went through the [ROS tutorial](tutorial.md/#ros), you probably installed some `apt` packages to proceed, or maybe you manually *downloaded* some code from `git`.

The point is, if you loose the container for some reason, will you also loose all the installations you made, all the files you downloaded, within the container? The answer is "yes".

That's the reason why we use a `Dockerfile` to keep track of most of the installations. Let's consider the [learn the C++ basics](tutorial.md/#c) example.
```bash
docker run -d -t --name ubuntu ubuntu #start a basic Linux container: -d sends it in the background (detach), -t keeps it running
docker exec -it ubuntu bash -c 'g++ --version' #gives error, g++ is not installed yet
docker exec -it ubuntu bash -c 'apt update && apt install g++ -y' #install the C++ compiler, you will need it!
```
We can keep track of the `g++` installation with a `Dockerfile`. Create it from vscode and save it in a dedicated folder
```dockerfile
FROM ubuntu
RUN apt update && apt install g++ -y
```
How it works? Save it and use it to build a specialized Docker Image
```bash
docker build -f /path/to/your/folder/Dockerfile -t my-image
```
so that if you start a container from the newly created image you will find `g++` already there
```bash
docker run -d -t --name my-ubuntu my-image
docker exec -it ubuntu bash -c 'g++ --version' #no error, prints the compiler version
docker run -it --rm my-image g++ --version #test the new image in a single line
```

## Docker Compose

Docker Containers can be composed together to form a modular architecture. For instance, consider a practical case on our teleoperation platform.

You are developing a super cool ROS package which turns a target 6D pose for the robot into target joint angles for the robotic arm. The vendor, Universal Robots in our case, provides us the ROS driver package, which will listen for your joint angle commands (via a `ros2 topic`) and will take care to send them to the robot.

The point is, are you sure that you want to test your new algorithm directly on real (very expensive) robots? Wouldn't you like to see first how the math that you implemented behaves? Maybe you'll find some bugs, so it's a good practice you test your codes on simulated robots first. Remember, simulated arms are free! 

Here's where modularity helps you! Let's say that your ROS package is installed along with the UR driver into a dedicated container, let's call it *drivers*. You only need to start a second container which runs the simulator to start debugging. 

Moreover, imagine that a colleague of yours has a container, let's name it *devices*, which turns mocap data into target 6D poses for the robots. You can just compose *devices* and *drivers* together and the teleoperation system is ready to go!

Whenever you're ready to test with real robots, just stop the simulator container and connect the *drivers* with your real robots.

### ROS

First, learn how it works in a [minimal ROS architecture](https://docs.ros.org/en/jazzy/How-To-Guides/Run-2-nodes-in-single-or-separate-docker-containers.html#run-two-nodes-in-two-separate-docker-containers) example.

Try to add a custom service (i.e. a container supported by a specific image) to the compose architecture using the [Dockerfile](#dockerfiles)
```yml
version: '2'

services:
    talker:
        image: osrf/ros:jazzy-desktop
        command: ros2 run demo_nodes_cpp talker
    listener:
        image: osrf/ros:jazzy-desktop
        command: ros2 run demo_nodes_cpp listener
        depends_on: talker
    my_service:
        build:
            dockerfile: /path/to/your/folder/Dockerfile
        command: g++ --version
```
``` bash
docker compose build my_service #build only my_service
docker compose up my_service #run only my_service
```
or 
``` bash
docker compose build
docker compose up
```
**Note:** Docker will take a lot of memory (disk) to store images and containers. Periodically remove unused images. Moreover clear the builder cache with `docker builder prune`. Cleaning the cache means that new builds will start almost from scratch, which means they eill take more time.
