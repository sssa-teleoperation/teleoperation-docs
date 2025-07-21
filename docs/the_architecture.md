# THE Architecture

The development of the following architecture was supported by the Tuscany Health Ecosystem (THE) project and it hosted on the [sssa-teleoperation](https://github.com/sssa-teleoperation) GitHub account.

## Dry run

Let's try to prepare your host OS for the teleoperation architecture
```bash
mkdir teleoperation && cd teleoperation
git clone git@github.com:sssa-teleoperation/docker.git
cd docker && docker compose up
```
If everything set up properly (it will take about 10 minutes) you can try to see what's running on
```bash
docker exec ros2_tools /rviz2
```