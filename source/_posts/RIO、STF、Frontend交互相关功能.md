---
title: RIO、STF、Frontend交互相关功能
date: 2017-04-21 12:45:40
categories: Testing
tags: 
  - Testbird
  - 项目关系
---
# RIO部分
## 一、获取设备列表
### 1、从cache中取缓存设备信息
1.1 取到设备信息，调用“api/rio/alive/”接口，获取在线设备列表  
<!-- more -->
* om会返回当前所有有心跳的设备，及其对应的locker_key,  
* 如果在线设备列表中，对应locker_key不为空，则将对应的设备status设置为Locked，否则置为Free
* 如果在线设备列表中，有新增设备，则调用“api/rio/”获取全部设备信息列 

1.2 无法取到设备信息，调用"api/rio/"接口，获取全部设备信息
* OM会返回当前所有的设备的全量信息，包括status和locker_key  
* 获取到当前设备列表后，结合本地租用记录判断，如果有对应的租用记录，则将该设备status设置成Locked，并设置locker_key

## 二、 Rent
1、先判断余额，余额不足，返回失败(如果是续租(duration=0)则不判断余额)，返回：余额不足:当前可用时长=x小时，租用时长=y小时

2、判断要租用的设备key是否在设备列表中，不在则返回：无法识别的设备key  
3、已获取待租用设备，调用'api/rio/alive/'接口，判断设备是否在线，如不在线，返回：该设备无法租用。注：这里没判断是否续租，即如果是续租，也会提示：该设备无法租用  
4、判断设备是否被其他用户租用，如已被租用，提示：该设备已被其他用户租用！  
5、扣费，扣费成功，通知OM lock device，并生成租用记录，并返回成功租用的设备信息，包括：  
```
data = {  
    "code": 0,  
    "public_ip": device["public_ip"],  
    "massage": u'',  
    "serial_no": device["serial_no"],  
    "public_port": device["public_port"],  
    "device_key": device["key"],  
    "device_name": device["key"],  
    "time_left": time_left,  
    "is_first_rent": is_first_rent,  
    "rent_key": rent_record.key,  
}
```
扣费失败：返回扣费失败信息  
6、成功租用，创建并返回租用记录  
[image](http://confluence.testbird.com/download/attachments/6362722/image2016-3-7%2021%3A20%3A50.png?version=1&modificationDate=1457356851000&api=v2)

## 三、Unlock（停止租用接口）
1、expend接口中，如果发现租用记录中当秋安剩余可用时间不足，则提示余额不足并调用unlock退出租用  
2、前端主动调用unclock，停止租用
