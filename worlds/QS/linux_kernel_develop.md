---
title: 内核驱动开发
description: 
published: false
date: 2025-05-26T03:09:07.190Z
tags: 
editor: markdown
dateCreated: 2025-05-26T03:03:55.063Z
---

参考链接1：https://blog.csdn.net/qq_53144843/article/details/126652621

## 一、驱动介绍

Linux驱动属于内核的一部分，学习驱动开发时将驱动设计为内核模块，内核模块是一种可以在系统运行时加载和卸载的机制。

内核编程的注意事项

> 1.不能使用C标准库和C标准头文件
>
> 2.使用GNU C
>
> 3.没有内存保护机制
>
> 4.不能处理浮点运算
>
> 5.注意并发互斥和可移植性问题 

### **1.内核模块**

Linux 驱动有两种运行方式，第一种是将驱动编译进 Linux 内核中，当 Linux 内核启动的时就会自动运行驱动程序。第二种是将驱动编译成模块(Linux 下模块扩展名为.ko)，在Linux 内核启动以后使用insmod或者modprobe命令加载驱动模块。在调试驱动的时候一般都选择将其编译为模块。

模块的加载和卸载注册函数如下

```cpp
#include <linux/init.h>
#include <linux/module.h> //需要包含头文件

module_init(xxx_init); //注册模块加载函数
module_exit(xxx_exit); //注册模块卸载函数
```

**module_init 函数用来向 Linux 内核注册一个模块加载函数，参数 xxx_init 就是需要注册的具体函数，当使用insmod命令加载驱动的时候， xxx_init 这个函数就会被调用。 module_exit()函数用来向 Linux 内核注册一个模块卸载函数，参数 xxx_exit 就是需要注册的具体函数，当使用“rmmod”命令卸载具体驱动的时候 xxx_exit 函数就会被调用**。

**字符设备驱动模块**加载和卸载模板如下所示：

```cpp
#include <linux/init.h>
#include <linux/module.h>
 
/* 驱动入口函数 */
static int __init xxx_init(void)
{
    /* 入口函数具体内容 */
    return 0;
}
 
 
/* 驱动出口函数 */
static void __exit xxx_exit(void)
{
    /* 出口函数具体内容 */
}
 
/* 将上面两个函数指定为驱动的入口和出口函数 */
module_init(xxx_init);
module_exit(xxx_exit)
 
MODULE_LICENSE("GPL");//GPL模块许可证

```

- 注：在编写模块时必须加上模块许可证，防止内核被污染，造成某些功能无法使用。

- 驱动编译完成以后扩展名为.ko，有两种命令可以加载驱动模块： insmod和 modprobe。

- modprobe 命令主要智能在提供了模块的依赖性分析、错误检查、错误报告等功能，推荐使用modprobe 命令来加载驱动。 modprobe 命令默认会去 `/lib/modules/<kernel-version` 目录中查找模块。

- 同时 modprobe 命令也可以卸载掉驱动模块所依赖的其他模块，前提是这些依赖模块已经没有被其他模块所使用，否则就不能使用 modprobe 来卸载驱动模块。所以对于模块的卸载，推荐使用 rmmod 命令。








> 文件所在：drivers/mmc/host/mediatek/mmc-ioctl/mmc-cdev.c

```c++
/**********************************************************************************************************************
*					 Copyright (c) 2023,Shenzhen Shichuangyi Electronics Co., Ltd. <-All rights reserved ->
*
**---------------------------------------------------------------------------------------------------------------------
*	@File   : mmc-cdev.c
*	@Path	: ~/MTK/alps/kernel-4.14/drivers/mmc/host/mediatek/mmc-ioctl
*	@Brief  : .
**---------------------------------------------------------------------------------------------------------------------
*	@Version: V0.1
*	@Author : blake.wu
*	@Date   : 2023.11.16
**---------------------------------------------------------------------------------------------------------------------
*	@History
*		Version		Author			Date			Modification
*		V0.1		blake.wu		2023.11.16		Original Version.
**-------------------------------------------------------------------------------------------------------------------*/
/*--------------------------------------------------- Header File ---------------------------------------------------*/
#define pr_fmt(fmt) KBUILD_MODNAME ": " fmt

#include <linux/types.h>
#include <linux/device.h>
#include <linux/cdev.h>
#include <linux/kref.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include <linux/compat.h>
#include <linux/slab.h>
#include <linux/module.h>
#include <linux/capability.h>
#include "android_ioctl.h"
#include <linux/fs.h>
#include <asm/uaccess.h>

#include "debug.h"
#include "mmc_dev_api.h"
/*------------------------------------------------- Macro Definition ------------------------------------------------*/
static struct mmc_cdev_t mmc_cdev;
static struct class *mmc_cdev_class = NULL;
static struct device *mmc_device = NULL;
static dev_t mmc_dev;
static int open_count = 0;
/*-------------------------------------------------- Local Variable -------------------------------------------------*/
struct mmc_cdev_t
{
    struct cdev cdev;
};
/*----------------------------------------------- Function Declaration ----------------------------------------------*/

/**************************************************************************************
* @description		: 打开设备
* @param - inode 	: 传递给驱动的inode
* @param - filp 	: 设备文件，file结构体有个叫做private_data的成员变量
                            一般在open的时候将private_data指向设备结构体。
* @return 			: 0 成功;其他 失败
*@warning:  #
*@author:   [blake.wu 2023/11/16 16:03]
****************************************************************************************/
int mmc_cdev_open(struct inode *inode, struct file *flip)
{
    pr_notice("mmc_cdev_open start!\n");

#if 1 // 设置 LED
    // 添加 LED2 breath_mode
    /* 设置为 breath_mode */
    mt6360_led_trigger(MT6360_LED2, MT_LED_BREATH_MODE);
    /* 开启 LED2 */
    mt6360_led_control(MT6360_LED2, LED_ON);
#endif

    if(open_count > 0)
    {
        pr_notice("mmc already opened!\n");
        return -EINVAL;
    }
    else
    {
        open_count++;
        flip->private_data = &mmc_cdev;
    }

    // 创建并且打开log 文件
    mmc_logging_ctl(1);
    return 0;
}


/**************************************************************************************
*@brief:  实现ioctl()接口
*@param:  file - 文件指针
*@param:  cmd - 命令
*@param:  arg - 参数
*@return:  long - 返回值
*@warning: 无
*@author:   [blake.wu 2023/11/16 16:08]
****************************************************************************************/
static long mmc_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
    // u8 opcode;
    u8 ret;
    u8 writeBuffer[5];

    switch (cmd)
    {
        case MMC_IOCTL_WRITE_DATA:
            ret = copy_from_user(writeBuffer, (void*)arg, 2);
            if (ret)
            {
                pr_err("Failed to write data: %d\n", ret);
                return -EFAULT;
            }
            switch (writeBuffer[0])
            {
                printk("opcode %x", writeBuffer[0]);
                case MMC_HARD_RESET/* constant-expression */:
                    /* code */
                    mmc_hwreset();
                    break;
                case MMC_VCC_OFF/* constant-expression */:
                    /* code */
                    mmc_power_ctl(0);
                    break;
                case MMC_VCC_ON/* constant-expression */:
                    /* code */
                    mmc_power_ctl(1);
                    break;
                case MMC_SAVE_LOG_NOW /* 0xA0 */:
                    /* code */
                    mmc_log_to_file();  // 刷新log.
                    break;
                default:
                    pr_err("Unknown UFS command: %d\n", writeBuffer[0]);
                    break;
            }
            return 1; /* 返回传输的字节数 */
        case MMC_IOCTL_WRITE_VOL_DATA:   /* 设置VCC VCCQ电压 */
            /* Copy Data */
            // ufs_send_vol_sy81050((void*)arg);
            return mmc_send_vol_sy81050((void*)arg); /* 返回错误码 */
        case MMC_IOCTL_READ_VOL_DATA:    /* 获取VCC VCCQ电压*/
            /* Copy Data */
            // ufs_read_vol_sy81050((void*)arg);
            return mmc_read_vol_sy81050((void*)arg); /* 返回错误码 */
        case MMC_IOCTL_READ_VERSION_DATA:   /* Get Version */
            mmc_get_img_version((void *)arg);
            // IMG VERSION: Commit id
            return 4; // 返回数据 4Byte

        default:
            pr_err("Unknown ioctl command: %d\n", cmd);
            return -EINVAL;
    }
    return 0;
}



/**************************************************************************************
*@brief: release函数，应用程序调用close关闭驱动文件的时候会执行
*@param - inode	: inode节点
*@param - filp    : 要打开的设备文件(文件描述符)
*@return          : 负数表示函数执行失败
*@warning: 无
*@author:   [blake.wu 2023/11/16 16:09]
****************************************************************************************/
int mmc_cdev_release(struct inode *inode, struct file *filp)
{
    pr_notice("mmc_cdev_release start!\n");
    open_count--;
    mmc_logging_ctl(0);

    // 关闭LED 设置
#if 1
    /* 关闭 LED2 */
    mt6360_led_control(MT6360_LED2, LED_OFF);
    /* 把LED2 设置none */
#endif

    return 0;
}


/**************************************************************************************
*@brief:  从设备读取数据
*@param - filp    : 要打开的设备文件(文件描述符)
*@param - buf     : 返回给用户空间的数据缓冲区
*@param - cnt     : 要读取的数据长度
*@param - offt    : 相对于文件首地址的偏移
*@return          : 读取的字节数，如果为负值，表示读取失败
*@warning: 无
*@author:   [blake.wu 2023/11/16 16:10]
****************************************************************************************/
static ssize_t mmc_cdev_read(struct file *file, char __user *buf, size_t size, loff_t *ppos)
{
    // pr_notice("mmc read\n");

	//copy_to_user((void *)query_rsp, (void *)&p_query_rsp, sizeof(struct str_ut_rsp_info));
    mmc_get_data_state(buf);

    *ppos += size;
    // pr_notice("mmc read done\n");
	return size;
}


/**************************************************************************************
*@brief:  ufs_cdev_write函数用于处理写入ufs字符设备的操作
*@param:  file - 文件指针
*@param:  buf - 用户空间数据缓冲区指针
*@param:  size - 写入数据的大小
*@param:  ppos - 文件偏移指针
*@return: ssize_t - 实际写入的字节数
*@warning: 无
*@author:   [blake.wu 2023/11/16 16:11]
****************************************************************************************/
ssize_t mmc_cdev_write(struct file *file, const char __user *buf, size_t size, loff_t *ppos)
{
    // struct et_mmc_request *user_mmc_request = (struct et_mmc_request *)buf;
    struct et_mmc_request mmc_request;
    //pr_notice("mmc write\n");
    memset(&mmc_request, 0, sizeof(struct et_mmc_request));

    if(size < sizeof( struct et_mmc_request))
    {
        pr_notice("mmc xfer size err!\n");
        return 0;
    }

    if (copy_from_user(&mmc_request.cmd.opcode, buf, sizeof( struct et_mmc_request)))
    {
        pr_notice("mmc_cdev_write: copy_from_user fail!!!!!!!!!!!!!!!!\n");
    }

    if(mmc_request.data.blks)
    {
        mmc_request.data.buffer = kmalloc(mmc_request.data.blks * mmc_request.data.blksz, GFP_KERNEL);
        //write
        if (mmc_request.data.dir == (1 << 8))
        {
            //if (copy_from_user(mmc_request.data.buffer, user_mmc_request->data.buffer, sizeof(mmc_request.data.blks * 512)))
            {
                //pr_notice("mmc_cdev_write1: copy_from_user fail!!!!!!!!!!!!!!!!\n");
            }
        }
         //printk("write data %x", mmc_request.data.buffer[0]);
    }

    //pr_notice("opcode code %d\n", mmc_request.cmd.opcode);
    send_mmc_request((void*)buf);
    // pr_notice("mmc write done\n");

    //if (copy_to_user((void *)&user_mmc_request->rsp, (void *)&mmc_request.rsp, sizeof(struct et_mmc_rsp)))
    {
    //    pr_notice("mmc_cdev_write: copy_to_user fail!!!!!!!!!!!!!!!!\n");
    }

    if(mmc_request.data.blks)
    {
        //printk("read data %x %x", user_mmc_request->data.data[0], mmc_request.data.data[0]);
        //read
        if (mmc_request.data.dir == (1 << 9))
        {
            //if (copy_to_user(user_mmc_request->data.buffer, mmc_request.data.buffer, sizeof(mmc_request.data.blks * 512)))
            {
                //pr_notice("mmc_cdev_write1: copy_from_user fail!!!!!!!!!!!!!!!!\n");
            }
        }
        kfree(mmc_request.data.buffer);
    }

    return size;
}


/**************************************************************************************
 * @brief: 刷新设备文件回调函数
 * @param: file - 指向打开的设备文件的指针
 *         id - 文件所有者标识
 * @return:  返回0表示成功，否则表示失败
 * @warning: 无
*@author:   [blake.wu 2023/11/16 16:11]
****************************************************************************************/
int mmc_cdev_flush(struct file *file, fl_owner_t id)
{
    //pr_notice("re-init mmc resp-q");
    //ut_resp_init();
    return 0;
}


/**************************************************************************************
 * @brief:     UFS设备文件操作结构体
 * @param: 无
 * @return: 无
  *@warning: 无
*@author:   [blake.wu 2023/11/16 16:12]
****************************************************************************************/
static const struct file_operations mmc_dev_fops =
{
    .owner          = THIS_MODULE,      // 将 owner 字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
    .open           = mmc_cdev_open,    // 将 open 字段指向 mmc_cdev_open(...)函数
    .read           = mmc_cdev_read,    // 将 read 字段指向 mmc_cdev_read(...)函数
    .write          = mmc_cdev_write,   // 将 write 字段指向 mmc_cdev_write(...)函数
    .flush          = mmc_cdev_flush,
    .unlocked_ioctl = mmc_ioctl,
    .release        = mmc_cdev_release, // 将 release 字段指向 mmc_cdev_release(...)函数
};


/**************************************************************************************
*@brief: 驱动入口函数
*@param: 无
*@return: 无
*@warning: 无
*@author:   [blake.wu 2023/11/16 16:12]
****************************************************************************************/
static int __init mmc_cdev_init(void)
{
    int alloc_ret = -1;
    int cdev_ret = -1;
    int major;

    printk("hello mmc\n");
    // 自动获取设备号，设备名chrdev_name ===> mmc
    alloc_ret = alloc_chrdev_region(&mmc_dev, 0, 1, "mmc");

    if(alloc_ret) {
        pr_notice("alloc_chrdev_region fail!!\n");
        goto error;
    }

    major = MAJOR(mmc_dev);

    memset(&mmc_cdev, 0, sizeof(struct mmc_cdev_t));

    // 使用 cdev_init()函数初始化 mmc_cdev 结构体，并链接到mmc_dev_fops 结构体
    cdev_init(&mmc_cdev.cdev, &mmc_dev_fops);

    // 将 owner 字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
    mmc_cdev.cdev.owner = THIS_MODULE;
    // 使用 cdev_add()函数进行字符设备的添加
    cdev_ret = cdev_add(&mmc_cdev.cdev, MKDEV(major, 0), 1);
    if(cdev_ret) {
        pr_notice("cdev_add fail!!\n");
        goto error;
    }
    // 使用 class_create 进行类的创建，类名称为 mmc_cdev_class
    mmc_cdev_class = class_create(THIS_MODULE, "mmc");
    if(IS_ERR(mmc_cdev_class))
    {
        pr_notice("class_create fail!!\n");
        goto error;
    }

    mmc_device = device_create(mmc_cdev_class, NULL, MKDEV(major, 0), NULL, "mmc");
    if(IS_ERR(mmc_device))
    {
        pr_notice("device_create fail!!\n");
        goto error;
    }

    return 0;

error:
    if(mmc_cdev_class)
    {
        class_destroy(mmc_cdev_class);
        mmc_cdev_class = NULL;
    }

    if(cdev_ret == 0)
    {
        cdev_del(&mmc_cdev.cdev);
    }

    if(alloc_ret == 0)
    {
        unregister_chrdev_region(mmc_dev, 1);
    }

    return -1;
}


/**************************************************************************************
*@brief:  驱动出口函数
*@param:  无
*@return:  无
*@warning:  无
*@author:   [blake.wu 2023/11/16 16:13]
****************************************************************************************/
static void __exit mmc_cdev_exit(void)
{
    device_destroy(mmc_cdev_class, mmc_dev);
    class_destroy(mmc_cdev_class);
    cdev_del(&mmc_cdev.cdev);
    unregister_chrdev_region(mmc_dev, 1);
    pr_notice("close mmc\n");
}



module_init(mmc_cdev_init);      // 模块初始化函数
module_exit(mmc_cdev_exit);      // 模块退出函数

MODULE_AUTHOR("scy-mmc");        // 模块作者
MODULE_LICENSE("GPL");           // 模块使用的许可证
// 模块描述
MODULE_DESCRIPTION("SCY mmc TEST DRIVER");
/*-------------------------------------------------------------------------------------------------------------------*/
```

