# Raspberry GPIO —— WiringPi  交叉编译


<!-- more -->

## 一、[Wiring Pi](http://wiringpi.com/)

> ***WiringPi*** is a ***PIN*** based GPIO access library written in C for the BCM2835, BCM2836 and BCM2837 SoC devices used in all ***Raspberry Pi***.

## 二、交叉编译

程序在虚拟机编译，在树莓派运行。

### （1）toochain

交叉编译工具链

Download：
```shell
$ git clone https://github.com/raspberrypi/tools.git
```

根据虚拟机配置添加环境变量到 /etc/profile末尾：

32bit：
> export PATH=$PATH:/your path/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin

64bit：
> export PATH=$PATH:/your path/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin

更新环境变量：
```shell
source /etc/profile
```
测试：
```shell
$ arm-linux-gnueabihf-gcc -v
```

### （2）Download and Install on Raspberry

```shell
$ git clone git://git.drogon.net/wiringPi
$ cd wiringPi
$ ./build
$ gpio -v
```

Test:
```shell
$ gpio -v
$ gpio readall
```

### （3）copy Wiring Pi Lib to VM

Wiring Pi在Raspberry上安装后，会在 /usr/local/lib 下生成lib文件 ***libwiringPi.so*** 、***libwiringPiDev.so***; 在 /usr/local/include 下生成 ***wiringPi.h***。将Raspberry中的lib、.h拷贝到VM中。

| 文件名 | Raspberry | VM | 执行操作 |
| --- | --- | ------------------------------------------------------- | --- |
| libwiringPi.so、libwiringPiDev.so | /usr/local/lib | your wiringPiLib path | copy |
| wiringPi.h | /usr/local/include | your wiringPiLib path | copy |

### （4）compile on VM

demo
```c
#include <stdio.h>
#include <wiringPi.h>

#define TEST_GPIO  24

int main(void)
{
    wiringPiSetup();
    pinMode(TEST_GPIO, OUTPUT);

    for(;;)
    {
        digitalWrite(TEST_GPIO, HIGH); delay(500);
        digitalWrite(TEST_GPIO, LOW); delay(500);
    }
    return 0;
}
```

makefile

```makefile
src += ./gpio.c
target = ./gpio

cc  = arm-linux-gcc

obj = $(src:%.c=%.o)

$(target):$(obj)
    $(cc) $^ -o $@ $(cflags) $(ldflags) -I/your wiringPiLib path -L/your wiringPiLib path -lwiringPi -lwiringPiDev -lpthread -lrt -lm -lcrypt
%.o:%.c
    $(cc) -c $< -o $@ $(cflags) $(cppflags) -I/your wiringPiLib path -L/your wiringPiLib path -lwiringPi -lwiringPiDev -lpthread -lrt -lm -lcrypt

.phony:clean
clean:
    rm *.o $(obj) $(target) -fr
```

> -I和-L分别指定了需要include的.h路径和需要链接的lib路径

## 三、Raspberry 40 Pin 引脚对照表（图片摘自树莓派实验室）

![CEMt54.png](https://s1.ax1x.com/2018/04/12/CEMt54.png)

