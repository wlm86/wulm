---
layout: post
title: libvirt添加API实现
category: VIRTUAL
tags: VIRTUAL
description:  segfault探究
---
#  libvirt添加API实现 


### 1.**需求**

   
### 2.**libvirt cgroup相关代码**
1. 创建vm中的cgroup创建概览
    ```
cmdCreate
 virDomainCreateXML
  qemuDomainCreateXML
   qemuProcessStart
    qemuProcessLaunch
     qemuSetupCgroup(driver, vm, nnicindexes, nicindexes)
      qemuInitCgroup
       virSystemdMakeMachineName
       virCgroupNewMachine
        virCgroupNewMachineSystemd
        virCgroupNewMachineManual
         virCgroupNewPartition
          virCgroupMakeGroup 真正创建cgroup目录结构的函数。
     qemuProcessSetupEmulator 设置cpu模拟器相关
     qemuProcessWaitForMonitor
     qemuSetupCpusetMems
     qemuProcessSetupVcpus 设置vm的vcpu对应的线程号等
     qemuProcessSetupIOThreads
    ```
2. 示例
  https://listman.redhat.com/archives/libvir-list/2016-June/msg01283.html
