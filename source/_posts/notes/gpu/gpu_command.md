---
title: gpu command
tags: [GPU, nvidia-smi, Monitoring]
excerpt: GPU管理和监控命令大全，包括nvidia-smi详细参数说明、GPU状态监控、计算模式设置、功耗限制、时钟频率锁定、进程查询等实用命令和配置方法。
---

## GPU 命令
```
nvidia-smi --format=csv,noheader,nounits --query-gpu=timestamp,index,memory.total,memory.used,memory.free,utilization.gpu,utilization.memory -lms 500
```

```
nsys profile -t cuda,nvtx,cublas,cublas-verbose,cusparse,cusparse-verbose,cudnn --stats=true --cuda-memory-usage true --force-overwrite true --gpu-metrics-device=1 --gpu-metrics-frequency=10  -o pp-yolo-int8 python benchmark.py --model_dir=quant_int8/models_re/yolov5s_quant --config_file config.yaml --backend_type=paddle --batch_size=1 --enable_gpu=true --gpu_id=1 --enable_trt=true --paddle_model_file "model.pdmodel" --paddle_params_file "model.pdiparams" --precision=int8
```

## nvidia-smi使用
<details>
<summary>nvidia-smi详细使用说明</summary>
https://docs.markhh.com/pages/dev/nvidia-smi/

```shell
    -pm,  --persistence-mode=   设置持久性模式: 0/DISABLED, 1/ENABLED
    -e,   --ecc-config=         切换 ECC 支持: 0/DISABLED, 1/ENABLED
    -p,   --reset-ecc-errors=   重置 ECC 错误计数: 0/VOLATILE, 1/AGGREGATE
    -c,   --compute-mode=       为计算应用程序设置模式:
                                0/DEFAULT, 1/EXCLUSIVE_PROCESS,
                                2/PROHIBITED
    -dm,  --driver-model=
    -fdm, --force-driver-model= 强制启用或禁用 TCC 驱动程序模型。 
                               
          --gom=                设置 GPU 操作模式:
                                0/ALL_ON, 1/COMPUTE, 2/LOW_DP
    -r    --gpu-reset           触发 GPU 的重置。
                                可用于在需要重启机器的情况下重置 GPU 硬件状态。
                                如果发生双位 ECC 错误，通常很有用。
                                重置操作不能保证在所有情况下都有效，应谨慎使用。
    -vm   --virt-mode=          切换 GPU 虚拟化模式:
                                将 GPU 虚拟化模式设置为 3/VGPU 或 4/VSGA。
                                GPU 的虚拟化模式只能在它运行在管理程序上时设置。
    -lgc  --lock-gpu-clocks=    将指定的一对 <minGpuClock,maxGpuClock> 时钟
                                频率取值范围（例如 1500,1500），设置为需锁定
                                的 GPU 时钟频率的范围（以 MHz 为单位）。 
                                无论 GPU 上是否存在正在运行的应用程序，设置此项
                                将取代applications-clocks并生效。
                                输入也可以是一个单一的期望时钟值
                                （例如 <GpuClockValue>）。
    -rgc  --reset-gpu-clocks    重置 GPU 时钟频率到默认值。
    -lmc  --lock-memory-clocks=     将指定的一对 <minMemClock,maxMemClock> 时钟
                                    频率取值范围（例如 5100,5100) ，设置为需锁定
                                    的 GPU 内存时钟频率的范围（以 MHz 为单位）。
                                    输入也可以是一个单一的期望时钟值
                                    （例如<MemClockValue>).
    -rmc  --reset-memory-clocks    重置 GPU 内存时钟频率到默认值。
    -ac   --applications-clocks= 将指定的一对 <memory,graphics> 时钟
                                    频率值（例如 2000,800) ，设置为 GPU 上
                                    运行的应用程序的memory和graphics时钟
                                    频率值（以 MHz 为单位）。
    -rac  --reset-applications-clocks     重置应用程序时钟到默认值。
    -pl   --power-limit=        以瓦特为单位指定最大电源管理限制。
    -cc   --cuda-clocks=        覆盖或恢复默认 CUDA 时钟频率。
                                在覆盖模式下，GPU 在运行 CUDA 应用程序时
                                时钟频率更高。 仅适用于从 Volta 系列开始的
                                受支持设备。 需要管理员权限。
                                0/RESTORE_DEFAULT, 1/OVERRIDE
    -am   --accounting-mode=    启用或禁用记帐模式: 0/DISABLED, 1/ENABLED
    -caa  --clear-accounted-apps    清除缓冲区中所有已记账的进程pid信息。
          --auto-boost-default= 将默认自动增强策略设置为 0/DISABLED 
                                或 1/ENABLED，仅在最后一个提升客户端
                                退出后强制执行更改。
          --auto-boost-permission=  允许非管理员/root 对自动增强模式进行控制：
                                0/UNRESTRICTED, 1/RESTRICTED
    -mig  --multi-instance-gpu= 启用或禁用多实例 GPU: 0/DISABLED, 1/ENABLED
                                需要 root 权限.
    -gtt  --gpu-target-temp=    以摄氏度为单位设置 GPU 的 GPU 目标温度。
                                需要管理员权限
```
</details>

### 设置GPU计算模式
`nvidia-smi -i gpu_id -c mode`
0/Default：表示每个设备允许多个上下文。
1/Exclusive_Thread：已弃用，改用 Exclusive_Process
2/Prohibited：表示每台设备不允许使用任何上下文（无计算应用程序）。
3/Exclusive_Process： 表示每个设备只允许一个上下文，一次可从多个线程使用。

### GPU 状态重置
`nvidia-smi -r`
一般用于GPU重启后

## 常见GPU使用命令
### GPU重置

### 查询限制功耗
nvidia-smi --query-gpu=power.limit --format=csv,noheader,nounits
sudo nvidia-smi -pl {power_limit}

### 锁频及持久化模式
sudo nvidia-smi -pm 1
nvidia-smi -q -d CLOCK
sudo nvidia-smi -lgc 2100,2100

### 查看哪些进程在使用GPU
`fuser -v /dev/nvidia*`
`lsof /dev/nvidia*`
