title: "kvm user interface"
date: 2015-03-20 10:23:27
tags:
---

最近准备开始研读KVM的代码实现，先从KVM用户层接口开始看起。

<!-- more -->

### kvm device ioctl interface

KVM内核提供的用户层接口以`/dev/kvm` 的`ioctl` 接口为主，可以在文件`virt/kvm/kvm_main.c`中看到。

核心的函数为`kvm_dev_ioctl` ，用户层工具如果需要创建虚拟机实例，必须要交互的接口是`/dev/kvm`。

*  KVM_GET_API_VERSION

该接口用于获取KVM的接口版本，用于用户层工具和内核交互时的版本检查，接口返回版本号`KVM_API_VERSION`。

* KVM_CREATE_VM

该接口用于创建一个虚拟机实例，会返回一个文件描述符`fd`，所有跟本虚拟机相关的内核接口均需要基于这个`fd`来进行`ioctl`操作。

* KVM_CHECK_EXTENSION

检查KVM的能力支持，由用户传递参数，检查内核是否支持。参数为`KVM_CAP_`前缀的能力值。能力值列表在`include/uapi/linux/kvm.h`中。

* KVM_GET_VCPU_MMAP_SIZE

获取VCPU的上下文大小，创建一个vcpu后，同样会返回一个文件描述符`fd`，可以mmap该`fd`，作为和内核层交互数据的接口。

* 体系结构相关的ioctl接口

这里X86、ARM等处理器均有不同实现，不一一描述。


### kvm create vm ioctl接口分析


在创建一个vm实例时，首先需要调用ioctl的`KVM_CREATE_VM`接口。调用形式如下： 

```
kvm->vm_fd = ioctl(kvm->sys_fd, KVM_CREATE_VM, KVM_VM_TYPE);
```

其中`KVM_VM_TYPE`是根据体系结构区分设置的整数值，`x86`为0。

在`kvm_dev_ioctl_create_vm`接口中，先创建`kvm`数据结构，利用`kvm_create_vm`，然后创建`fd`，利用`anon_inode_getfd`接口。


