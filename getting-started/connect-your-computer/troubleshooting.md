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
4. Select a lighter machine learning model (check "Memory" column in a model table - there is information about how much GPU memory will this model require to train). If this information is not provided, use a simple rule: the higher the model in the table, the lighter it is.
![MMsegmentation required memory](https://github.com/supervisely/developer-portal/assets/87002239/48468b74-5d5b-4145-8782-6b18e3ee42e8)
![Lightest YOLOv8 model](https://github.com/supervisely/developer-portal/assets/87002239/d8f1a539-ea51-4e4e-b98c-dc85c420b543)


5. Reduce batch size
6. Reduce model input resolution

|![](https://github.com/supervisely/developer-portal/assets/87002239/9ee30cfa-4282-4cf5-be26-8b13fcfc8026)<br> MMsegmentation image resolution/batch size|![](https://github.com/supervisely/developer-portal/assets/87002239/d65bc286-5b3e-40f9-8200-c91e8753e6e9)<br>MMdetection v3 image resolution/batch size|
|:-:|:-:|

#### Additional: stop a process via docker.

1. run `docker ps` - it will return a big table with all docker containers running on this machine
2. run `docker stop <put_your_container_id_here>`