# Lab Xv6 and Unix utils
***
Instruction: [Lab: Xv6 and Unix utilities (mit.edu)](https://pdos.csail.mit.edu/6.S081/2020/labs/util.html)
## Boot Xv6
Information: [6.S081 / Fall 2020 (mit.edu)](https://pdos.csail.mit.edu/6.S081/2020/tools.html)
I boot Xv6 on my arch linux 
```shell
sudo pacman -S riscv64-linux-gnu-binutils riscv64-linux-gnu-gcc riscv64-linux-gnu-gdb qemu-arch-extra
```

I make qemu meet some error in rep 2020, so I change to the 2022 reo.
Clone the Xv6 2022 repo and check out the util branch.
```shell
git clone git://g.csail.mit.edu/xv6-labs-2020
cd xv6-labs-2022
git status
On branch util
Your branch is up to date with 'origin/util'.

nothing to commit, working tree clean
```
Make qemu and test
```shell
make qemu
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2227
xargstest.sh   2 3 93
cat            2 4 32824
echo           2 5 31704
forktest       2 6 15840
grep           2 7 36264
init           2 8 32168
kill           2 9 31616
ln             2 10 31440
ls             2 11 34768
mkdir          2 12 31680
rm             2 13 31672
sh             2 14 54248
stressfs       2 15 32552
usertests      2 16 180312
grind          2 17 47432
wc             2 18 33752
zombie         2 19 31040
console        3 20 0
```

Press `ctrl+p` to check the process running
```shell
$
1 sleep  init
2 sleep  sh
```
After I press `ctrl+p` I found I cannot exit the status which I can't run any cmd, so I use `ctrl+a x` to teminated the qemu :(

sleep.c
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(2, "Usage: sleep sometimes ..\n");
        exit(1);
    }
    sleep(atoi(argv[1]));
    exit(0);
}
```

pingpong.c
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
int main(int argc, char *argv[]) {
    int p1[2],p2[2];
    pipe(p1);
    pipe(p2);
    char buf[1];
    if (fork()==0)
    {
        close(p1[0]);
        close(p2[1]);
        read(p2[0],buf,1);
        fprintf(1,"%d: received ping\n",getpid());
        write(p1[1],buf,1);
        exit(0);
    }else{
        close(p2[0]);
        close(p1[1]);
        write(p2[1],"a",1);
        read(p1[0],buf,1);
        fprintf(1,"%d: received pong\n",getpid());
    }
    exit(0);
}
```

primes.c
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
int is_prime(int num) {
    int flag = num;
    for (int i = 2; i <= num / 2; i++) {
        if (num % i == 0) {
            flag = 0;
            break;
        }
    }
    return flag;
}
int main(int argc, char *argv[]) {
    int p1[2], p2[2];
    int num[1];
    int i, j;
    for (i = 2; i < 36; i++) {
        pipe(p1);
        pipe(p2);
        if (fork() == 0) {
            close(p1[0]);
            close(p2[1]);
            j = read(p2[0], num, 1);
            if (j < 1) {
                fprintf(2, "Receive error!\n");
                exit(1);
            }
            num[0] = is_prime(num[0]);
            write(p1[1], num, 1);
            exit(0);
        } else {
            close(p2[0]);
            close(p1[1]);
            write(p2[1], &i, 1);
            j = read(p1[0], num, 1);
            if (j < 1) {
                fprintf(2, "Receive error!\n");
                exit(1);
            }
            if (num[0] != 0) {
                fprintf(1, "prime %d\n", num[0]);
            }
            close(p1[0]);
            close(p2[1]);
        }
    }
    exit(0);
}
```

find.c
find.c 实现基本参照ls.c 但是需要注意的是ls.c中并没有对"."和".."目录做处理，因此在对目录操作时，注意要检查路径是不是"."和".."，防止无限递归，其次是ls.c文件中的buf为static，如果在一个语句中调用多次fmtname，返回的结果是一样的，如`strcmp(ffmtname(buf1),fmtname(buf2))` , 不太懂为什么。
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

char *fmtname(char *path) {
    static char buf[DIRSIZ + 1];
    char *p;

    // Find first character after last slash.
    for (p = path + strlen(path); p >= path && *p != '/'; p--)
        ;
    p++;
  
    // Return blank-padded name.
    if (strlen(p) >= DIRSIZ)
        return p;
    memmove(buf, p, strlen(p));
    memset(buf + strlen(p), ' ', DIRSIZ - strlen(p));
    return buf;
}

void find(char *path, char *fname) {
    char buf[512], *p,buf2[DIRSIZ];
    int fd;
    struct dirent de;
    struct stat st;
  
    //验证输入的path是否打开
    if ((fd = open(path, 0)) < 0) {
        fprintf(2, "find: cannot open %s\n", path);
        return;
    }

  

    // 检查是否能查看文件状态
    if (fstat(fd, &st) < 0) {
        fprintf(2, "find: cannot stat %s\n", path);
        close(fd);
        return;
    }
    // 根据文件类型执行对应操作
    switch (st.type) {
    case T_DEVICE:
    case T_FILE:
        strcpy(buf, path);
        strcpy(buf2, fname);
        strcpy(buf,fmtname(buf));
        strcpy(buf,fmtname(buf));
        strcpy(buf2,fmtname(buf2));
        if (strcmp(buf,buf2)==0)
        {
            printf("%s\n",path);
        }
        break;
  
    case T_DIR:
        if (strlen(path) + 1 + DIRSIZ + 1 > sizeof buf) {
            printf("find: path too long\n");
            break;
        }
        strcpy(buf, path);
        p = buf + strlen(buf);
        *p++ = '/';
        while (read(fd, &de, sizeof(de)) == sizeof(de)) {
            if (de.inum <2)
                continue;
            memmove(p, de.name, DIRSIZ);
            p[DIRSIZ] = 0;
            if (stat(buf, &st) < 0) {
                printf("find: cannot stat %s\n", buf);
                continue;
            }
            if (strcmp(p,".")!=0 && strcmp(p,"..")!=0)
            {
                find(buf,fname);
            }
        }
        break;
    }
    close(fd);
}

int main(int argc, char *argv[]) {
    // int i;
    if (argc < 2) {
        fprintf(2,"Usage: find files\n");
        exit(0);
    }else if (argc == 2)
    {
        find(".",argv[1]);
    }else{
        find(argv[1],argv[2]);
    }
    exit(0);
}
```

xargs.c
写此程序的时候踩了点坑，主要是`exec` 的调用不熟悉，导致编写错误，exec fail 而且没有在子进程让执行失败后输出日志，卡了好久。
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/param.h"

int main(int argc, char *argv[]) {
  
  char buf[1];     // 缓存
  char *cmd[5]; // 存储cmds
  int i;
  for (int i = 0; i<4; i++) {
    cmd[i] = malloc(sizeof(char)*MAXARG);
  }
  // 将命令行参数存入cmds中
  for (i = 1; i < argc; i++) {
    strcpy(cmd[i - 1], argv[i]);
  }
  // 读输入
  int j = 0;
  while (read(0, buf, sizeof(buf))) {
    if (buf[0] != '\n') {
      cmd[i-1][j++] = buf[0];
    } else {
      cmd[i-1][j] = 0;
      cmd[i][0] = 0;
      if (fork() == 0) {
        exec(argv[1], (char **)cmd);
        printf("exec fail\n");
        exit(0);
      }
      wait(0);
      j = 0;
    }
  }
  wait(0);
  exit(0);
}
```
