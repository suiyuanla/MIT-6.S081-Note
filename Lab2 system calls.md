# 系统调用

## 一、准备

```shell
git fetch
git checkout syscall
make clean
```

## 二、GDB调试

1. 在一个终端窗口运行`make qemu-gdb`
2. 在第2个终端窗口运行`riscv64-linux-gnu-gdb `，打开gdb
    ```
    (gdb) target remote localhost:26000
    (gdb) file kernal/kernal
    (gdb) b 79          # 在80行打断点
    (gdb) b syscall     # 在syscall函数处打断点
    (gdb) layout src    # 显示源码窗口
    (gdb) layout asm    # 显示汇编窗口
    (gdb) s             # step
    (gdb) n             # next
    ```