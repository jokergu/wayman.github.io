> 以分析ESP32-P4为例。

假设程序中有一处除零错误：
```c
static void _div_by_zero(void)
{
    int a = 0;
    int temp = 100 / a;
    printf("temp=%d\n", temp);
}

extern "C" void app_main(void)
{
    _div_by_zero();
}
```

运行后输出如下异常信息：
```log
Guru Meditation Error: Core  0 panic'ed (Breakpoint). Exception was unhandled.

Core  0 register dump:
MEPC    : 0x48016588  RA      : 0x48121362  SP      : 0x4ff30a20  GP      : 0x4ff15c80  
TP      : 0x4ff30a70  T0      : 0x4fc0a9f8  T1      : 0x4800a1a0  T2      : 0xffffffff  
S0/FP   : 0x48133000  S1      : 0x00000003  A0      : 0x00000027  A1      : 0x00000000  
A2      : 0x00000000  A3      : 0x00000000  A4      : 0x20800008  A5      : 0x00000000  
A6      : 0xa0000000  A7      : 0x0000000a  S2      : 0x00000000  S3      : 0x00000000  
S4      : 0x00000000  S5      : 0x00000000  S6      : 0x00000000  S7      : 0x00000000  
S8      : 0x00000000  S9      : 0x00000000  S10     : 0x00000000  S11     : 0x00000000  
T3      : 0x00000000  T4      : 0x00000000  T5      : 0x00000000  T6      : 0x00000000  
MSTATUS : 0x00011880  MTVEC   : 0x4ff00003  MCAUSE  : 0x00000003  MTVAL   : 0x00000000  
MHARTID : 0x00000000  

Stack memory:
4ff30a20: 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00001388 0x00000003 0x00000001
...
4ff30e00: 0xa5a5a5a5 0xa5a5a5a5 0xa5a5a5a5 0xa5a5a5a5 0xa5a5a5a5 0xa5a5a5a5 0xa5a5a5a5 0xa5a5a5a5



ELF file SHA256: 6613aca3f
```

在项目中新建一个`dump.log`文件，并将如上的异常信息拷贝到`dump.log`文件中。

使用命令`riscv32-esp-elf-gdb --batch -n ./build/myproject.elf -ex 'target remote | python $IDF_PATH/tools/gdb_panic_server.py --target esp32p4 ./dump.log' -ex bt`打印异常发生时的函数调用栈：
```
app_main () at /home/xxx/govee/myproject/main/govee_app_main.cpp:33
33          _div_by_zero();
#0  app_main () at /home/xxx/govee/myproject/main/govee_app_main.cpp:33
#1  0x48121362 in main_task (args=<optimized out>) at /home/xxx/devenv/esp-idf/components/freertos/app_startup.c:208
#2  0x00000000 in ?? ()
Backtrace stopped: frame did not save the PC
```

若内核是`xtensa`的芯片，如`esp32-s3`，则使用`xtensa-esp32s3-elf-gdb --batch -n ./build/myproject.elf -ex 'target remote | python $IDF_PATH/tools/gdb_panic_server.py --target esp32s3 ./dump.log' -ex bt`打印异常发生时的函数调用栈。
