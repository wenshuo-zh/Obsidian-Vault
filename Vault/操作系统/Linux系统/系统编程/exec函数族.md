不创建新的进程，而是用新程序替换当前进程的代码段、数据段和堆栈，进程的 `pid` 保持不变。
`exec` 成功后不会返回，因此后面的代码不会执行；失败时返回 `-1`，并设置 `errno`。
`exec` 一般和 `fork()` 一起使用，由子进程调用 `exec` 执行新程序。
`l`：参数以可变参数形式逐个传入，最后一个参数必须是 `NULL`。
`p`：只需提供程序名，系统会按照 `PATH` 环境变量搜索程序。
`v`：参数通过 `argv` 数组传入（不是 C++ 的 `vector`）。
`e`：调用者可以通过 `envp` 数组自定义新程序的环境变量。
```c
int execl(const char *pathname, const char *arg, ...);

int execlp(const char *file, const char *arg, ...);

int execle(const char *pathname, const char *arg, ..., char * const envp[]);

int execv(const char *pathname, char *const argv[]);

int execvp(const char *file, char *const argv[]);

int execvpe(const char *file, char *const argv[],
            char *const envp[]);
```



**execl**
`int execl(const char *path, const char *arg, ...);`
	path：可执行文件的路径
	arg：可执行程序的参数
**execlp**
借助环境变量

例子：获取指定目录下所有文件的个数
```c
int main(int argc, char *argv[])
{
    pid_t pid = fork();
    if(pid == 0){
        int fd = open("a.txt", O_RDWR | O_CREAT | O_TRUNC, 0664);
        dup2(fd, 1);//把fd重定向到1，向标准输出1输出的都会写入到"a.txt"里面
        execlp("ls", "ls", "-a", argv[1], NULL);
    }else{
        wait(NULL);
        execlp("wc", "wc", "-l", "a.txt", NULL);
    }
    return 0;
}
```
`./main`不加参数，默认当前目录
![[Pasted image 20260930132934.png]]
`./main ~/os`加参数，看os目录下文件个数
`execlp("ls", "ls", "-a", argv[1], NULL);`
代码中**arg\[1]** 就是用来传参的，在终端输入路径
![[Pasted image 20260930133248.png]]

