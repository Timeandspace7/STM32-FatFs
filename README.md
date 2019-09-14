# STM32-FatFs

uVision	: V5.12.0.0
库版本  ：STM32F10x_StdPeriph_Lib_V3.5.0
FATSF   : ff13a  下载地址：http://elm-chan.org/fsw/ff/00index_e.html
作者    ：By:Timeandspace7     At:YSU-B307-2
MCU     ：STM32F103C8T6
描述    ：使用 STM32 控制直流电机转速，并利用霍尔电流传感器采集电机电流，
	  并利用 FafFs 文件系统将其存入 TF 卡中。
硬件    : 
	   电流传感器：https://doc.docsou.com/bae61e1bfdc2f6c9524985cd7.html
           电机驱动  ：https://item.taobao.com/item.htm?spm=a1z09.2.0.0.78a42e8dURtDb8&id=39439405805&_u=e1q8geuh5958
	   直流电机  ：https://item.taobao.com/item.htm?spm=a230r.1.14.20.1538470e0kt8Ry&id=21271920000&ns=1&abbucket=3#detail：

硬件连接：
	  SD-FATFS  :  SPI1_CS->PA4 SPI1_CLK->PA5 SPI1_MISO->PA6 SPI1_MOSI->PA7
	  电流检测  :  PA3->CSM006NPT  VCC: 5V   
	  //温度检测:  PA1->DS18B20    VCC: 5V   	
	  电机的PWM :  PB7->TIM4_CH2   PB8->IN1 PB9->IN2 VCC:3.3V(一定为3.3V)  
	  USART1    :  PA9->TX         PA10->RX
	  LED       :  LED0->PC13   LED1->PA8  
	  启停硬件连接：
	  5V-500R-PB0/PB1
	  启动：PB1 两盏灯同时点亮
	  停止：PB0 两盏灯同时熄灭
	  PB0PB1分别外接500R电阻，接5V
	  PB0/PB1引脚接地，引发EXTI下降沿触发分别执行停止、启动

工程功能：
	  1.数据存储在SD卡中
	  //2.温度检测
	  3.电流检测
	  4.TIM4控制电机
	  5.USART1的通信
	  6.LED可控
	  7.占空比=x/1000*100%,x越大旋转速度越慢，目前是x = 400,一圈约为1.06s
	  8.电机启动，LED0亮；数据开始采集，LED1亮，数据停止采集，LED1灭
	  9.Fatfs 写入 TF 电流数据需要 9ms
	  10.ADC 每采集一次的时间为 t = (239.5+12.5)/12M = 21us
          11.1秒钟写入 TF卡大概为 110 个电流数据
          12.不在进行50个数据平均出来一个数据，即ADC每21us采集一次数据
          //13.温度检测需要6.233ms

总体描述：占空比越高，速度越慢  PWM在40%时候大概1s/r,while死循环，电流数据经过递推平均算法后写入SD卡，从测取到写入一次需要9.04ms.

20180825新增功能：
	在外部中断服务函数中，进行消抖操作，判断PB0/PB1是否为低电平

20180828新改功能：
   代码50行：	f_open( &fil , FileNameTab, FA_CREATE_NEW);
更改为		f_open( &fil , FileNameTab, FA_CREATE_ALWAYS);
	即每次进入主函数，都重新创建文件（删除旧文件，新建新文件）。
   代码89行：	f_open( &fil , FileNameTab, FA_OPEN_EXISTING | FA_WRITE );
更改为		f_open( &fil , FileNameTab, FA_WRITE );
	即每次都只写（在新文件中写入）。
