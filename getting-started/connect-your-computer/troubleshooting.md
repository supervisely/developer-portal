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

You can also try other [official NVIDIA fix](https://github.com/lurk-lab/gh-actions-runner/pull/9) to solve this problem for a specific docker container or plunge into this problem by reading [this discussion](https://github.com/NVIDIA/nvidia-docker/issues/1671) or [this official description](https://github.com/NVIDIA/nvidia-docker/issues/1730).