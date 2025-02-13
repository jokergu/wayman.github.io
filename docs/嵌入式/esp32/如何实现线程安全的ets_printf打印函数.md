## 背景
如果在线程A中随机打印`ets_printf("name=Jay");`，在另一个线程B中随机打印`ets_printf("sex=male")`，则有可能出现打印乱码的情况，如下所示：
```
name=sex=maJayle
```

## 为什么ets_printf在不同线程中打印信息可能乱码
因为ets_printf不是线程安全的。

## 如何实现线程安全的ets_printf函数
线程安全的est_printf实现如下所示：
```c
void thread_safe_ets_printf(const char *format, ...) {
    SemaphoreHandle_t mutex = NULL;
    if(mutex == NULL)
    {
        mutex = xSemaphoreCreateMutex();
    } 
    xSemaphoreTake(mutex, portMAX_DELAY);
    // portDISABLE_INTERRUPTS();
    va_list args;
    va_start(args, format);
    static char output[512] = {0};
    vsnprintf(output, sizeof(output), format, args);
    ets_printf(output);
    // printf(output);
    va_end(args);
    // portENABLE_INTERRUPTS();
    xSemaphoreGive(mutex);
}
```

注意：不要使用`portDISABLE_INTERRUPTS()`和`portENABLE_INTERRUPTS()`开关中断，这并不能保证其线程安全，因为有些esp32芯片是多核的，`portDISABLE_INTERRUPTS()`和`portENABLE_INTERRUPTS()`仅能关闭单核的中断(官方人员透露)。

