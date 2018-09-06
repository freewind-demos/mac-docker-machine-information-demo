Mac "docker-machine" Information Demo
=====================================

在Mac上，由于操作系统不支持docker所需要的一些功能，所以要运行docker，必须先使用`docker-machine`相关的命令，在Mac上创建一个Linux虚拟机，在它里面执行docker命令。

这个docker machine实际上是运行于Virtual Box上的一个Linux。
由于它的存在，使得我们在Mac上使用docker有很多不便，特别是与路径相关的。
比如说，docker输出中有一些路径，我们在Mac本机上怎么也找不到，也不知道去哪里找。
我们必须了解关于Docker Machine更多的一些信息，才能帮助我们快速定位问题。

docker-machine创建的虚拟机在哪里
-----------------------

```
$ tree ~/.docker
```

输出：

```
tree ~/.docker/machine
/Users/freewind/.docker/machine
├── cache
│   └── boot2docker.iso
├── certs
│   ├── ca-key.pem
│   ├── ca.pem
│   ├── cert.pem
│   └── key.pem
└── machines
    └── default
        ├── boot2docker.iso
        ├── ca.pem
        ├── cert.pem
        ├── config.json
        ├── default
        │   ├── Logs
        │   │   └── VBox.log
        │   ├── default.vbox
        │   └── default.vbox-prev
        ├── disk.vmdk
        ├── id_rsa
        ├── id_rsa.pub
        ├── key.pem
        ├── server-key.pem
        └── server.pem

6 directories, 18 files
```

可以看到`machines`下有一个`default`，这就是我们创建的名为`default`的docker machine对应的文件。

查看目录大小
------

```
du -sh ~/.docker/machine
348M	/Users/freewind/.docker/machine
```

这个大小是不断增长的。每当我们的docker下载了新的image，它就会变大。

查看文件大小
------

```
$ cd ~/.docker/machine/machines/default
$ ls -ahlF
```

输出

```
drwx------  13 freewind  staff   442B Sep  6 19:42 ./
drwx------   3 freewind  staff   102B Sep  6 19:41 ../
-rw-r--r--   1 freewind  staff    48M Sep  6 19:41 boot2docker.iso
-rw-r--r--   1 freewind  staff   1.0K Sep  6 19:42 ca.pem
-rw-r--r--   1 freewind  staff   1.1K Sep  6 19:42 cert.pem
-rw-------   1 freewind  staff   2.8K Sep  6 19:42 config.json
drwx------   5 freewind  staff   170B Sep  6 19:41 default/
-rw-------   1 freewind  staff   252M Sep  6 20:53 disk.vmdk
-rw-------   1 freewind  staff   1.6K Sep  6 19:41 id_rsa
-rw-------   1 freewind  staff   381B Sep  6 19:41 id_rsa.pub
-rw-------   1 freewind  staff   1.6K Sep  6 19:42 key.pem
-rw-------   1 freewind  staff   1.6K Sep  6 19:42 server-key.pem
-rw-r--r--   1 freewind  staff   1.1K Sep  6 19:42 server.pem
```

可以看到最大的是`disk.vmdk`，它相当于是虚拟机的硬盘。
在mac下，我们使用`docker pull`命令下载的image，实际上是保存在它内部的，难怪我们找不到。

docker-machine环境信息
------------------

```
$ docker-machine env default
```

输出

```
set -gx DOCKER_TLS_VERIFY "1";
set -gx DOCKER_HOST "tcp://192.168.99.100:2376";
set -gx DOCKER_CERT_PATH "/Users/freewind/.docker/machine/machines/default";
set -gx DOCKER_MACHINE_NAME "default";
# Run this command to configure your shell:
# eval (docker-machine env default)
```

它既告诉了我们一些信息，又同时是将这些信息设置到shell环境变量中的脚本。

`docker-machine inspect`
------------------------

如果我们需要更详细的信息，可以运行：

```
$ docker-machine inspect
```

输出：

```
{
    "ConfigVersion": 3,
    "Driver": {
        "IPAddress": "192.168.99.100",
        "MachineName": "default",
        "SSHUser": "docker",
        "SSHPort": 52428,
        "SSHKeyPath": "/Users/freewind/.docker/machine/machines/default/id_rsa",
        "StorePath": "/Users/freewind/.docker/machine",
        "SwarmMaster": false,
        "SwarmHost": "tcp://0.0.0.0:3376",
        "SwarmDiscovery": "",
        "VBoxManager": {},
        "HostInterfaces": {},
        "CPU": 1,
        "Memory": 1024,
        "DiskSize": 20000,
        "NatNicType": "82540EM",
        "Boot2DockerURL": "",
        "Boot2DockerImportVM": "",
        "HostDNSResolver": false,
        "HostOnlyCIDR": "192.168.99.1/24",
        "HostOnlyNicType": "82540EM",
        "HostOnlyPromiscMode": "deny",
        "UIType": "headless",
        "HostOnlyNoDHCP": false,
        "NoShare": false,
        "DNSProxy": true,
        "NoVTXCheck": false,
        "ShareFolder": ""
    },
    "DriverName": "virtualbox",
    "HostOptions": {
        "Driver": "",
        "Memory": 0,
        "Disk": 0,
        "EngineOptions": {
            "ArbitraryFlags": [],
            "Dns": null,
            "GraphDir": "",
            "Env": [],
            "Ipv6": false,
            "InsecureRegistry": [],
            "Labels": [],
            "LogLevel": "",
            "StorageDriver": "",
            "SelinuxEnabled": false,
            "TlsVerify": true,
            "RegistryMirror": [],
            "InstallURL": "https://get.docker.com"
        },
        "SwarmOptions": {
            "IsSwarm": false,
            "Address": "",
            "Discovery": "",
            "Agent": false,
            "Master": false,
            "Host": "tcp://0.0.0.0:3376",
            "Image": "swarm:latest",
            "Strategy": "spread",
            "Heartbeat": 0,
            "Overcommit": 0,
            "ArbitraryFlags": [],
            "ArbitraryJoinFlags": [],
            "Env": null,
            "IsExperimental": false
        },
        "AuthOptions": {
            "CertDir": "/Users/freewind/.docker/machine/certs",
            "CaCertPath": "/Users/freewind/.docker/machine/certs/ca.pem",
            "CaPrivateKeyPath": "/Users/freewind/.docker/machine/certs/ca-key.pem",
            "CaCertRemotePath": "",
            "ServerCertPath": "/Users/freewind/.docker/machine/machines/default/server.pem",
            "ServerKeyPath": "/Users/freewind/.docker/machine/machines/default/server-key.pem",
            "ClientKeyPath": "/Users/freewind/.docker/machine/certs/key.pem",
            "ServerCertRemotePath": "",
            "ServerKeyRemotePath": "",
            "ClientCertPath": "/Users/freewind/.docker/machine/certs/cert.pem",
            "ServerCertSANs": [],
            "StorePath": "/Users/freewind/.docker/machine/machines/default"
        }
    },
    "Name": "default"
}
```

不过它没有告诉我们虚拟机内部文件信息，我们在后面看怎么办。

获取某个docker container的路径信息
-------------------------

```
$ docker ps -a

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
043f95189bf8        hello-world         "/hello"            44 minutes ago      Exited (0) 44 minutes ago                         jovial_galileo
```

```
$ docker inspect 043f95189bf8
[
    {
        "Id": "043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2",
        "Created": "2018-09-06T12:21:18.278659385Z",
        "Path": "/hello",
        "Args": [],
        "State": {
            "Status": "exited",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2018-09-06T12:21:18.517319168Z",
            "FinishedAt": "2018-09-06T12:21:18.508011783Z"
        },
        "Image": "sha256:2cb0d9787c4dd17ef9eb03e512923bc4db10add190d3f84af63b744e353a9b34",
        "ResolvConfPath": "/mnt/sda1/var/lib/docker/containers/043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2/resolv.conf",
        "HostnamePath": "/mnt/sda1/var/lib/docker/containers/043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2/hostname",
        "HostsPath": "/mnt/sda1/var/lib/docker/containers/043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2/hosts",
        "LogPath": "/mnt/sda1/var/lib/docker/containers/043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2/043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2-json.log",
        "Name": "/jovial_galileo",
        "RestartCount": 0,
        "Driver": "aufs",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "shareable",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/asound",
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": null,
            "Name": "aufs"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "043f95189bf8",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/hello"
            ],
            "ArgsEscaped": true,
            "Image": "hello-world",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "af5fab0f52fb50a469a0d7e10b5f01f863662ed24ff4dbceb967ac08f1df4968",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/af5fab0f52fb",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "ab49635d52626de5458fcea2a5341645a6baf1ed3b39e531e61f0e9853898180",
                    "EndpointID": "",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

可以看到，在前面它提到了自己的image路径为：

```
/mnt/sda1/var/lib/docker/containers/043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2
```

然而在mac本机，我们是找不到这个地址的，因为它存在于名为`default`的docker machine中。

登录docker machine
----------------

通过`docker-machine --help`我们能找到它所支持的很多命令，其中有一个是`ssh`，用于登录到虚拟机中。

```
$ docker-machine ssh default
```

输出

```
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 18.06.1-ce, build HEAD : c7e5c3e - Wed Aug 22 16:27:42 UTC 2018
Docker version 18.06.1-ce, build e68fc7a
docker@default:~$
```

寻找docker image保存路径
------------------

首先我们可以查看一下前面的image路径（需要用`sudo`）：

```
$ sudo ls /mnt/sda1/var/lib/docker/containers/043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2
```

输出：

```
043f95189bf8535a0169111d105fdd657326b2ef2afa9083a378be982befc7f2-json.log
checkpoints
config.v2.json
hostconfig.json
hostname
hosts
mounts
resolv.conf
resolv.conf.hash
```

果然是存在的。

同时，通过该路径，我们推断出docker containers应该保存在`/mnt/sda1/var/lib/docker/containers`中，查看一下它的大小：

```
$ sudo du -sh /mnt/sda1/var/lib/docker/containers
136.0K	/mnt/sda1/var/lib/docker/containers
```

太小了，看来不是它。

我们再看一下`/mnt/sda1/var/lib/docker`下有哪些文件：

```
$ sudo ls -a /mnt/sda1/var/lib/docker
.           builder     containers  overlay2    swarm       volumes
..          buildkit    image       plugins     tmp
aufs        containerd  network     runtimes    trust
```

它的总大小为：

```
$ sudo du -sh /mnt/sda1/var/lib/docker
205.6M	/mnt/sda1/var/lib/docker
```

由于寻找，发现`aufs`是最大的：

```
$ sudo du -sh /mnt/sda1/var/lib/docker/aufs
204.2M	/mnt/sda1/var/lib/docker/aufs
```

它下面到底有什么文件呢？

由于docker machine里面的linux是最小化的，所以没有`tree`命令，也没有办法安装（里面通过`busybox`提供了简单命令），所以我只能通过`find`命令列出所有文件，再修改一下，变成：

```
aufs/mnt
aufs/mnt/bfd8abd884d30a41c4aaad49b89fedff18995143a16b8a0b14dcd7ef6fb10cc9
aufs/mnt/c15bc3a1bc5388682efa1e98ae4f0055d91b023569d5786d2052d6ef07c2daa7
aufs/mnt/...
aufs/layers
aufs/layers/bfd8abd884d30a41c4aaad49b89fedff18995143a16b8a0b14dcd7ef6fb10cc9
aufs/layers/c15bc3a1bc5388682efa1e98ae4f0055d91b023569d5786d2052d6ef07c2daa7
aufs/layers/...
aufs/diff
aufs/diff/bfd8abd884d30a41c4aaad49b89fedff18995143a16b8a0b14dcd7ef6fb10cc9
aufs/diff/bfd8abd884d30a41c4aaad49b89fedff18995143a16b8a0b14dcd7ef6fb10cc9/...
aufs/diff/87c0e4638699401eb1c66ea5cbe453f98216e18abef26da7b3598ecac6e3871e
aufs/diff/87c0e4638699401eb1c66ea5cbe453f98216e18abef26da7b3598ecac6e3871e/usr
aufs/diff/87c0e4638699401eb1c66ea5cbe453f98216e18abef26da7b3598ecac6e3871e/usr/sbin
aufs/diff/87c0e4638699401eb1c66ea5cbe453f98216e18abef26da7b3598ecac6e3871e/usr/sbin/policy-rc.d
aufs/diff/87c0e4638699401eb1c66ea5cbe453f98216e18abef26da7b3598ecac6e3871e/sbin
aufs/diff/.........
```

这个输出有上万行，我只节选了典型的几个。

从中我们可以看到，docker image的保存并不是一个个单独的文件，而是一堆文件，通过`layers`, `mnt`以及大量的`diff`，使得不同的image之间可以尽可能的重用文件，有点像文件版本控制系统。

随着我们下载的docker image越来越多，这里的文件也会越来越多。

