---
description: Some of the problems you could run into when using the agent, along with solutions
---


# NVIDIA

## Failed to initialize NVML: Unknown Error
This error applies to any utilities/libraries that use NVML: `pytorch`, `nvidia-smi`, `pynvml` etc. 
It frequently shows up unexpectedly and prevents applications from using the GPU until it is fixed. 

### Quick solution
If the error only appears inside the container, you can quickly fix it by restarting it.
If the error also appears on the host after running `nvidia-smi` you can fix it by using the `reboot` command on the host.

### Proper solution
1. If the error has not yet appeared, you can check if your system is affected by this problem.
    - run agent docker on the host PC;
    - run `sudo systemctl daemon-reload` on the host;
    - execute `nvidia-smi` into the agent container and catch `NVML initialization error`

2. Set the parameter `"exec-opts": ["native.cgroupdriver=cgroupfs"]` in the `/etc/docker/daemon.json` file.
```bash
~$ cat /etc/docker/daemon.json 
```
```json
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    },
    "exec-opts": ["native.cgroupdriver=cgroupfs"]
}
```

3. Restart Docker with `sudo systemctl restart docker`

You can also try other [official NVIDIA fixes](https://github.com/lurk-lab/gh-actions-runner/pull/9) to solve this problem for a specific docker container or plunge into this problem by reading [this discussion](https://github.com/NVIDIA/nvidia-docker/issues/1671) or [this official description](https://github.com/NVIDIA/nvidia-docker/issues/1730).


## CUDA Out Of Memory Error
This error could appear in any training apps.

### Solution

1. Check the amount of free GPU memory by running `nvidia-smi` command in your machine terminal - it will give you an understanding of how much GPU memory is it necessary to free in order to train your machine learning model

2. Stop unnecessary app sessions in Supervisely:
    *START button → App Sessions → stop all unnecessary app sessions by clicking on Stop button in front of every undesired app session*

3. Stop unnecessary processes in your machine terminal by running `sudo kill <put_your_process_id_here>`

4. Select a lighter machine learning model (check "Memory" column in a model table - there is information about how much GPU memory will this model require to train).

![MMsegmentation required memory](https://github.com/supervisely/developer-portal/assets/87002239/5c31d56d-185a-4f3b-9307-2da0d70a35a3)

   If this information is not provided, use a simple rule: the higher the model in the table, the lighter it is.

![The lightest YOLOv8 model](https://github.com/supervisely/developer-portal/assets/87002239/a4381712-1d89-4f16-b8a5-bd18fcb6a167)

5. Reduce batch size or model input resolution
   <table>
   <tr>
   <td><figure><img src="https://github.com/supervisely/developer-portal/assets/87002239/d5d3b1ad-836f-493d-8e1c-19f0300b50f0" alt="mmseg"></figure></td>
   <td><figure><img src="https://github.com/supervisely/developer-portal/assets/87002239/d65bc286-5b3e-40f9-8200-c91e8753e6e9" alt="mmdet"></figure></td>
   </tr>
   <tr>
   <td>MMsegmentation image resolution/batch size</td>
   <td>MMdetection v3 image resolution/batch size</td>
   </tr>
   </table>


#### Additional: stop a process via docker.

1. run `docker ps` - it will return a big table with all docker containers running on this machine
2. run `docker stop <put_your_container_id_here>`

## Can't start the docker container. Trying to use another runtime.

This message indicates that there was a problem using the Nvidia runtime, most likely the Nvidia driver failed after an automatic kernel update - by default, this feature is enabled on Ubuntu. However, it often fails because the driver cannot be unloaded while it is in use.<br>

### Solution

#### Fast solution

The simplest way to fix this problem is to `reboot` the machine. After the reboot, the Nvidia driver will be reloaded and the problem will be fixed.<br>
But, if you don't want to reboot the machine, use the second solution.

#### Proper solution

In case you receive: `nvidia version mismatch` you can fix it without rebooting or reinstalling the driver using these commands:

```bash
kill -9 $(lsof /dev/nvidia* | awk '{print $2}')
sleep 5
modprobe --remove nvidia_uvm nvidia_drm nvidia_modeset nvidia drm_kms_helper drm
modprobe nvidia_uvm nvidia_drm nvidia_modeset nvidia drm_kms_helper drm
```

The first command will kill all processes that use the Nvidia driver, and the last two will unload and reload the driver.<br>
You might need to redeploy your agent on the machine after running this command.<br>

### Additional: disable automatic kernel updates.
You can also run this command to disable automatic kernel updates:
```bash
sudo apt remove unattended-upgrades
```
If the commands above don't work for you (some process is auto restarting preventing the driver from properly unload), you can simply reboot the machine.