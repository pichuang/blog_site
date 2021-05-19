---
layout: post
title: Kubernetes 部署在虛機好還是裸機好?
author: Phil Huang
toc: true
tags:
  - container
categories:
  - container
date: 2020-08-04 00:31:45
udpated: 2020-08-04 00:31:45
---

在一篇 [Kubernetes 部署在虛機好還是裸機好?][1]，我有提到一個論點是 `Container are Linux`，有客戶問我說如何證明這件事? 過程蠻簡單的，我們可以透過一個小實驗來表示，我這邊使用大家比較常用的 Docker 來進行小實驗

<!--more-->

## 運行一個 Docker Container 在系統上

```bash
# Pull Container Image
$ docker pull quay.io/app-sre/nginx:1.9.2
1.9.2: Pulling from app-sre/nginx
4d2e9ae40c41: Already exists
a3ed95caeb02: Already exists
ada158fbf84d: Already exists
cfd60577ca05: Already exists
f72b86395ecd: Already exists
406a1e871229: Already exists
391515714543: Already exists
Digest: sha256:e9ce80ba3441259c5df465bf3dc94610abdd0d65d24965723f8bd0249dad113a
Status: Image is up to date for quay.io/app-sre/nginx:1.9.2
quay.io/app-sre/nginx:1.9.2

# 啟動 nginx
$ docker run -p 80:80 -d quay.io/app-sre/nginx:1.9.2
d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc

# 檢查服務
$ curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...omit...

# 檢查 Log
$ docker logs d37f6e55ac
172.17.0.1 - - [04/Aug/2020:01:14:38 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
```

## Docker 做了些什麼? Linux Namespace 操作大簡化

其實上面的動作是 Docker 透過簡化的指令，創造了多種來自 `Linux Namespace` 的能力並且提供了給這個 Container 裡面的服務，下面就是可以解釋 Container are Linux 的一個實際技術例子

```bash
# 從 docker inspect 獲得 Process ID
$ docker inspect --format '{{printf "Pid:%d" .State.Pid}}' d37f6e55ac
Pid:2863

# 從 ps 獲得 Process ID
$ ps -ef |grep nginx
root      2863  2846  0 09:13 ?        00:00:00 nginx: master process nginx -g daemon off;
104       2893  2863  0 09:13 ?        00:00:00 nginx: worker process
root      3348  1555  0 09:24 pts/0    00:00:00 grep --color=always nginx
```

各位看官可以了解到無論是從 Docker 角度或者是 Linux 系統角度，都可以獲得到 nginx 所使用的 PID (Process ID) namespace 都是 *2863*

如果更細節下去看的話，你會發現 Linux Namesapce 所提供的核心能力，都跟這個 PID *2863* 連接在一起提供一個核心級的資源隔離能力，

```bash
# /proc/<PID>/ns
$ ls -l /proc/2863/ns
total 0
lrwxrwxrwx. 1 root root 0 Aug  4 09:22 ipc -> ipc:[4026532431]
lrwxrwxrwx. 1 root root 0 Aug  4 09:22 mnt -> mnt:[4026532425]
lrwxrwxrwx. 1 root root 0 Aug  4 09:13 net -> net:[4026532434]
lrwxrwxrwx. 1 root root 0 Aug  4 09:22 pid -> pid:[4026532432]
lrwxrwxrwx. 1 root root 0 Aug  4 09:27 user -> user:[4026531837]
lrwxrwxrwx. 1 root root 0 Aug  4 09:22 uts -> uts:[4026532430]
```

以 Docker 的角度來看的話，一個容器啟動的順序是: `Containerd > Container-shim > runtime-runc > Target container`，詳細技術可以參考 [淺談 Container 實現原理, 以 Docker 為例(II) - Hwchiu][4]

```bash
$ ps -aef --forest |grep container -A 1
root     30430     1  0 08:31 ?        00:00:07 /usr/bin/containerd
root      2846 30430  0 09:13 ?        00:00:00  \_ containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root      2863  2846  0 09:13 ?        00:00:00      \_ nginx: master process nginx -g daemon off;
---
...omit...
```

## cgroups 也可以使用?

> cgroup = control group based traffic control filter

關於 cgroups 的理解，我覺得這篇文章寫得不錯 [理解Docker（4）：Docker 容器使用 cgroups 限制资源使用][2] 一文

預設狀態下，Docker 會在 `/sys/fs/cgroup/*/docker` 目錄之下產生以 Container ID 為名字的資料夾放置各式參數

```bash
# 列出所有跟 docker 有關係的 cgroup 資料夾
$ find /sys/fs/cgroup/* -name docker -type d
/sys/fs/cgroup/blkio/docker
/sys/fs/cgroup/cpu,cpuacct/docker
/sys/fs/cgroup/cpuset/docker
/sys/fs/cgroup/devices/docker
/sys/fs/cgroup/freezer/docker
/sys/fs/cgroup/hugetlb/docker
/sys/fs/cgroup/memory/docker
/sys/fs/cgroup/net_cls,net_prio/docker
/sys/fs/cgroup/perf_event/docker
/sys/fs/cgroup/pids/docker
/sys/fs/cgroup/systemd/docker
```

如果換個角度，以 Container ID 下去對所有 cgroups 去做搜尋，會發現以下這些 cgroups 資源你都可以管理

```bash
$ cd /sys/fs/cgroup
$ find -iname d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./devices/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./memory/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./perf_event/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./freezer/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./net_cls,net_prio/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./pids/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./blkio/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./cpuset/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./hugetlb/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./cpu,cpuacct/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
./systemd/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc

# 隨便選一個 `cpu,cpuacct` 觀察
$ ls -l ./cpu,cpuacct/docker/d37f6e55ac959cbdc82ef9de6969696f77b7804106ba310638cbf2c0988d5bdc
total 0
-rw-r--r--. 1 root root 0 Aug  4 09:13 cgroup.clone_children
--w--w--w-. 1 root root 0 Aug  4 09:13 cgroup.event_control
-rw-r--r--. 1 root root 0 Aug  4 09:23 cgroup.procs
-rw-r--r--. 1 root root 0 Aug  4 09:13 cpu.cfs_period_us
-rw-r--r--. 1 root root 0 Aug  4 09:13 cpu.cfs_quota_us
-rw-r--r--. 1 root root 0 Aug  4 09:13 cpu.rt_period_us
-rw-r--r--. 1 root root 0 Aug  4 09:13 cpu.rt_runtime_us
-rw-r--r--. 1 root root 0 Aug  4 09:13 cpu.shares
-r--r--r--. 1 root root 0 Aug  4 09:13 cpu.stat
-r--r--r--. 1 root root 0 Aug  4 09:13 cpuacct.stat
-rw-r--r--. 1 root root 0 Aug  4 09:13 cpuacct.usage
-r--r--r--. 1 root root 0 Aug  4 09:13 cpuacct.usage_percpu
-rw-r--r--. 1 root root 0 Aug  4 09:13 notify_on_release
-rw-r--r--. 1 root root 0 Aug  4 09:13 tasks
```

當然除非你只是暫時想看看到效果，個人並不是很建議你直接對這些參數做調整，實務上你應該要使用 docker 提供的參數 [Runtime options with Memory, CPUs, and GPUs - Docker][3] 去調整，而且也比較好理解其資源控制意義

## Environment
- Red Hat Enterprise Linux 7.8
- Docker version 19.03.12, build 48a66213fe

## References
- [Kubernetes 部署在虛機好還是裸機好?][1]
- [理解Docker（4）：Docker 容器使用 cgroups 限制资源使用][2]
- [Runtime options with Memory, CPUs, and GPUs - Docker][3]
- [淺談 Container 實現原理, 以 Docker 為例(II) - Hwchiu][4]

[1]: https://blog.pichuang.com.tw/20200713-bm-and-vm-container-deployment-consideration/#Containers-are-Linux
[2]: https://www.cnblogs.com/sammyliu/p/5886833.html
[3]: https://docs.docker.com/config/containers/resource_constraints/
[4]: https://www.hwchiu.com/container-design-ii.html