# 使用platformio开发ESP32-C3 SuperMini与双目屏幕


## 开发环境

使用CLion + platform，在windows上开发（使用vscode + platform也可以）。

## 硬件说明

这是一个嵌入式软件项目,需要一块esp32-c3开发板以及GC9D01驱动的屏幕。连接方式如下：

|  屏幕引脚   | ESP32-C3引脚  |
|  ----  | ----  |
| VCC  | 5V |
| GND  | GND |
| DIN  | GPIO4 |
| CLK  | GPIO7 |
| CS1  | GPIO2 |
| CS2  | GPIO6 |
| DC  | GPIO0 |
| RST1  | GPIO8 |
| RST2  | GPIO5 |
| BL1  | GPIO1 |
| BL2  | GPIO3 |

## 踩坑说明

### Windows IDE环境搭建

安装CLion和python3以及MinGW环境，安装platformio插件，该部分可自行检索教程。

### 框架安装
esp32的库版本使用的是2.0.13，使用的是Arduino框架。

### ESP32-C3调试方式
可选择openocd + gdb的调试方式，需要外接调试器，如jlink。因ESP32-C3支持USB CDC类的开发，故而直接使用USB进行输出，对于调试简便而言直接采用USB打印的方式。在
platformio.ini文件中需添加如下配置，开启USB CDC：
```
platform_packages =
    toolchain-riscv32-esp @ 8.4.0+2021r2-patch5
build_flags =
    -D ARDUINO_USB_CDC_ON_BOOT=1
    -D ARDUINO_USB_MODE=1
```
在终端输入如下命令可以打开打印界面：
```
pio device monitor
```
在代码中添加Serial.println()进行打印。
### 分区文件
使用的是huge_app.csv，当代码段和数据段大小超过了分区大小时，可能会编译报错，具体路径在:
```
C:\Users\name\.platformio\packages\framework-arduinoespressif32\tools\partitions
```
**警告：若分区未选择正确，可能会导致固件烧录后esp32持续reboot，无法正常运行**

需要在platformio.ini文件中需添加如下配置，选择分区文件：
```
board_build.partitions = huge_app.csv
```
### TFT_eSPI库
有两种安装方式，一种是直接在platformio的主页->Libraries->Registry选项中搜索TFT_eSPI库进行安装，另外一种是在工程目录中的lib文件夹中添加自己的第三方库。这里我先是安装了官方的TFT_eSPI库，但是其中并没有GC9D01驱动文件，方便起见我直接将厂家提供的包含GC9D01驱动的TFT库覆盖安装完的TFT库。理论上而言只需要修改User_Setup.h、User_Setup_Select.h以及config.h文件即可

### 一些宏定义
厂家提供的例程未使用DMA通道，故而需要注释#define USE_DMA这一行，否则眼睛效果无法实现。

编译时会有warning提示，如果不想看到无关痛痒的warning，可以添加：
```
#define DISABLE_ALL_LIBRARY_WARNINGS
```



## 参考

- [0.71inch DualEye LCD Module](https://www.waveshare.net/wiki/0.71inch_DualEye_LCD_Module)