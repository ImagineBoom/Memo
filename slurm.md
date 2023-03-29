[TOC]

## 安装部署

todo

## 使用总览

| 命令     | 含义                             | 简单示例                                    |
| -------- | -------------------------------- | ------------------------------------------- |
| sbatch   | 批量提交作业命令，后面跟脚本文件 | sbatch xxx.sh                               |
| squeue   | 查看目前提交作业的信息           | squeue(可显示作业号、作业状态等)            |
| salloc   | 抢占计算资源命令                 | salloc -p kshctest -N 1 -n 32               |
| scontrol | 查看正在计算作业信息             | scontrol show job jobid                     |
| scancel  | 取消作业                         | scancel JOBID                               |
| sacct    | 查看历史作业                     | sacct -j jobid -X -o elapsed,state,nodelist |
| sinfo    | 查看节点和分区信息               | sinfo -lNa                                  |
| srun     | 运行并行作业                     | srun -c24 XXX                               |

- 请不要在登录节点上不通过作业调度管理系统直接运行作业，以免影响其余用户的正常使用；

- [x] ### sinfo

```sh
sinfo               #查看所有分区状态
sinfo -a            #查看所有分区状态
sinfo -N            #查看节点状态
sinfo -n node-name  #查看指定节点状态
sinfo --help        #查看sinfo的说明
```

| 输出字段  | 解释                                                         |
| --------- | ------------------------------------------------------------ |
| PARTITION | 分区名（队列名）                                             |
| AVAIL     | 队列是否可用，up（可用）、inact（不可用）                    |
| TIMELIMIT | 该队列作业运行时间限制，infinite（不限时），30:00 表示30分钟，如果是2-00:00:00表示2天 |
| NODES     | 节点数                                                       |
| STATE     | 节点状态，drain(故障)、alloc(已被分配)、idle(可用)、down(下线)、mix(部分占用） |
| NODELIST  | 节点列表                                                     |

- [x] ### squeue

```sh
squeue									#查看运行中的作业列表
squeue -l  							#查看列表细节信息
squeue -u<USERNAME>
squeue -j<JOBID>
```

| 输出字段         | 解释                                                    |
| ---------------- | ------------------------------------------------------- |
| JOBID            | 作业号                                                  |
| PARTITION        | 作业运行的队列名                                        |
| NAME             | 作业名可自定义                                          |
| USER             | 该作业所属账号名                                        |
| ST/STATE         | 作业状态，R(运行中)、PD(排队中)、CG(将完成)、CD(已完成) |
| NODES            | 作业所使用节点数                                        |
| NODELIST(REASON) | 作业所使用节点列表                                      |



- [x] ### srun/sbatch/salloc

| 输入参数                  | 解释                                                         |
| ------------------------- | ------------------------------------------------------------ |
| --help                    | 显示帮助信息                                                 |
| -N, --nodes=N             | 指定节点数                                                   |
| -n, --ntasks=ntasks       | 指定总进程数；不使用cpus-per-task，可理解为进程数即为核数    |
| --ntask-per-node=N        | 指定每个节点进程数/核数，使用-n参数后变为每个节点最多运行的进程数 |
| -c, --cpu-per-task=NCPUs  | 指定每个进程使用核数，不指定默认为1                          |
| -p, --partion=debug       | 指定分区                                                     |
| -w, --nodelist=node[1,2]  | 指定提交作业到node1和node2节点                               |
| -x, --exclude=node[3,5-6] | 提交作业时排除node3和node5-6节点                             |
| -o, --output=filename     | 指定标准输出到filename文件                                   |
| -e, --error=filename      | 指定重定向错误输出到filename文件                             |
| -J, --job-name=jobname    | 指定作业名为jobname                                          |
| -t, --time=dd-hh:mm:ss    | 限制运行time分钟                                             |
| --mail-type=type          | notify on state change: BEGIN, END, FAIL or ALL              |
| --mail-user=user          | who to send email notification for job state changes         |
| --mem=MB                  | minimum amount of real memory                                |
| --mem-per-cpu=MB          | maximum amount of real memory per allocated cpu required by the job<br/>--mem >= --mem-per-cpu if --mem is specified. |
| -q, --qos=qos             | 服务质量                                                     |

- [x] ### srun

```sh
srun -J JOBNAME -p debug -N 2 -c 1 -n 32 --ntasks-per-node=16 -w node[3,4] -x node[1,5-6] --time=dd-hh:mm:ss --output=file_name --error=file_name --mail-user=address --mail-type=ALL mpirun -n 64 ./iPic3D ./inputfile/test.inp

```



- [x] ### sbatch

计算开始后，工作目录中会生成以 slurm 开头的.out 文件为输出文件（不指定输出的话）

- touch run.sh

```sh
#!/bin/bash                     %指定运行shell
#提交单个作业
#SBATCH --job-name=JOBNAME      %指定作业名称
#SBATCH --partition=debug       %指定分区
#SBATCH --nodes=2               %指定节点数量
#SBATCH --cpus-per-task=1       %指定每个进程使用核数，不指定默认为1
#SBATCH -n 32										%指定总进程数；不使用cpus-per-task，可理解为进程数即为核数
#SBATCH --ntasks-per-node=16    %指定每个节点进程数/核数,使用-n参数（优先级更高），变为每个节点最多运行的任务数
#SBATCH --nodelist=node[3,4]    %指定优先使用节点
#SBATCH --exclude=node[1,5-6]   %指定避免使用节点
#SBATCH --time=dd-hh:mm:ss      %作业最大运行时长，参考格式填写
#SBATCH --output=file_name      %指定输出文件输出
#SBATCH --error=file_name       %指定错误文件输出
#SBATCH --mail-type=ALL         %邮件提醒,可选:END,FAIL,ALL
#SBATCH --mail-user=address     %通知邮箱地址

source /public/home/user/.bashrc   #导入环境变量文件

mpirun -n 32 ./iPic3D ./inputfiles/test.inp #运行命令
```

- sbatch run.sh

- [x] ### salloc

todo

- [x] scancel

| 输入参数     | 解释                       |
| ------------ | -------------------------- |
| [JOBID]      | 终止作业JOBID              |
| -n [jobname] | 终止作业名为jobname的作业  |
| -p [name]    | 终止队列名为name的作业     |
| -t PENDING   | 终止正在排队的作业         |
| -w fa0101    | 终止运行在fa0104节点的作业 |

- [x] ### scontrol

```sh
scontrol show job JOBID         					#查看作业的详细信息
scontrol show node              					#查看所有节点详细信息
scontrol show node node-name    					#查看指定节点详细信息
scontrol show node | grep CPU 					  #查看各节点cpu状态
scontrol show node node-name | grep CPU 	#查看指定节点cpu状态
```

```sh
# 在任务开始前却发现作业的属性写错了（例如提交错了分区，修改名字），取消了重新排队似乎很不划算。如果作业恰好 没在运行，我们是可以通过 scontrol 命令来更新作业的属性

scontrol update jobid=JOBID ... #...为下面参数
reqnodelist=<nodes>
reqcores=<count>
name=<name>
nodelist=<nodes>
excnodelist=<nodes>
numcpus=<min_count-max_count>
numnodes=<min_count-max_count>
numtasks=<count>
starttime=yyyy-mm-dd
partition=<name>
timelimit=d-h:m:s
mincpusnode=<count>
minmemorycpu=<megabytes>
minmemorynode=<megabytes>
```



- Refs:

1. https://blog.csdn.net/tangjiahao10/article/details/126281267
2. https://www.hpccube.com/doc/1.0.6/30000/general-handbook/User-Guide/slurm.html?h=slurm
3. https://zhuanlan.zhihu.com/p/356415669

