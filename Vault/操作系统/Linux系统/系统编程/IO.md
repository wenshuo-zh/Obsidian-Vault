# 文件描述符fd
0 标准输入（键盘）
1 标准输出（屏幕）
2 标准错误（屏幕）

调用open会在文件描述符表找最小未用过的整数作为新fd返回（3）

# 函数
## open、close、read、write、lseek函数
**open函数：**  
	`int fd = open(文件路径, 访问模式（只读、只写、可读写）, 创建时需加入权限（例如0664）);`
	成功返回最小且没被占用的文件描述符、失败返回-1
	用于将文件描述符与文件绑定，给read，write函数使用
**close函数：** 
	`close(fd);`
	成功返回0，失败返回-1
	关闭并释放文件描述符fd
**read函数：** 
	`ssize_t read_count = read(fd, buf, count);`
		count为期望读到的字节数
	成功返回读到的字节数，失败返回-1
**write函数：** 
	`ssize_t write_count = write(fd, buf, count);`
	成功返回实际写入的字节数，可能小于请求写入的字节数；失败返回-1
**lseek函数：** 修改fd文件偏移量
	`lseek(fd,偏移字节数, 基准位置);`
		SEEK_SET 基准文件开头
		SEEK_CUR 当前指针位置
		SEEK_END 文件末尾
**示例一：** 
```c
int main(int argc, char *argv[]){
	int fd = open("./a.txt", O_RDWR);
	char buf[64];
	
	off_t offset = lseek(fd, 0, SEEK_CUR);
	printf("offset = %ld\n", (long)offset);
	
	read(fd, buf, sizeof(buf));
	
	offset = lseek(fd, 0, SEEK_CUR);
	printf("offset = %ld\n", (long)offset);
	
	lseek(fd, 3, SEEK_SET);
	
	write(fd, "abc", 3);
	off_t file_size = lseek(fd, 0, SEEK_END);
	printf("file_size = %ld\n", (long)file_size);
	return 0;
}
```
![[89d26307e3aaf55b75929ade1399ba1e.png|691]]
**示例二：** 实现自己的cp指令mycp（复制文件）
```c
int main(int argc, char *argv[])
{
	if (argc != 3) {
		printf("argc error!\n");
		return -1;
	}
	int source_fd = open(argv[1], O_RDONLY);	
	int target_fd = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0664);
	char buf[32];
	while (1) {
		ssize_t read_count = read(source_fd, buf, sizeof(buf));
		if (read_count == 0)break;
		write(target_fd, buf, read_count);
	}
	return 0;
}
```
![[ec8f6b750659b055b080aa001f37767d.png|700]]
实现`cp a.txt b.txt` （将a.txt复制到b.txt）
## 阻塞

```c
int main(int argc, char *argv[])
{
    char buf[1024];
    ssize_t read_count = read(0, buf, sizeof(buf));
    printf("read_count = %d\n", read_count);
    write(1, buf, read_count);
    return 0;
}
```
`read`文件描述符0（标准输入）；`write`文件描述符1（标准输出）
默认阻塞
![[b8a8f2ac816dd40d3cb56f771977474e.png|570]]
阻塞，键盘输入1234回车后继续
![[15670a0299519c7b524fe38ed1f809e3.png|606]]
## dup、dup2函数
多个文件描述符打开同一个文件的示例，不能做到复制文件描述符
```c
int main(int argc, char *argv[])
{
    int fd = open("a.txt", O_RDWR);
    int fd2 = open("a.txt", O_RDWR);

    printf("fd = %d, fd2 = %d\n", fd, fd2);
    
    write(fd, "hello\n", 6);
    write(fd2, "world\n", 6);
    return 0;
}
```
![[3de7c8d59dbf02e9e6a832831e10e2a4.png]]
**dup函数：** 复制文件描述符
	`int newfd = dup(fd)`
```c
int main(int argc, char *argv[])
{
    int fd = open("a.txt", O_RDWR);
    printf("fd = %d\n", fd);
    int newfd = dup(fd);
    write(fd, "oldfd\n", 6);
    write(newfd, "newfd\n", 6);
    return 0;
}
```
![[6b37204cf21c1e00eacf7fbf6cf32214.png|536]]
**dup2函数：** dup函数的升级版，可以指定新文件描述符的值；如果`newfd`原来已经打开，`dup2()`会先关闭它，再让`newfd`指向和`oldfd`相同的打开文件描述
	`int newfd = dup2(oldfd, newfd)`
```c
int main(int argc, char *argv[])
{
    int fd = open("a.txt", O_RDWR);
    printf("fd = %d\n", fd);
    int newfd = dup2(fd, 10);
    printf("newfd = %d\n", newfd);
    write(fd, "oldfd\n", 6);
    write(newfd, "newfd\n", 6);
    return 0;
}
```
![[1d8268fc0d1e91085f4218598da16122.png|540]]

# 示例
```c
int main(int argc, char *argv[])
{
// 创建/打开 mc.txt，只写，权限0644
    int fd_mc = open("./mc.txt", O_WRONLY|O_CREAT|O_TRUNC, 0644);
    if(fd_mc == -1){
        perror("error mc.txt");
        return -1;
    }
// 只读打开 /etc/passwd
    int fd_passwd = open("/etc/passwd", O_RDONLY);
    if(fd_passwd == -1){
        perror("error passwd");
        close(fd_mc);
        return -1;
    }
// dup2：把 fd_mc 复制到文件描述符编号7
// dup2：把 fd_passwd 复制到文件描述符编号3
    if(dup2(fd_mc, 7) == -1){
        perror("dup2 fd_mc error");
        return -1;
    }
    if(dup2(fd_passwd, 3) == -1){
        perror("dup2 fd_passwd error");
        return -1;
    }
// fd=3 和 fd_passwd 指向同一个打开文件（/etc/passwd），共享文件偏移

// lseek 在 fd=3（也就是/etc/passwd），从文件开头向后偏移10字节
    char buf[64];
    if(lseek(3, 10, SEEK_SET) == -1){
        perror("lseek error");
        return -1;
    }
// 循环读取 fd=3，写入 fd=7
    while(1){
        ssize_t read_count = read(3, buf, sizeof(buf));
        if(read_count < 0){
            perror("read passwd error");
            return -1;
        }
        if(read_count == 0)break;
        ssize_t write_count = write(7, buf, read_count);
        if(write_count == -1){
            perror("write mc.txt error");
            return -1;
        }
    }

    // struct stat fileStat;
    // if(stat("/etc/passwd", &fileStat) == -1){
    //     perror("stat passwd");
    //     return -1;
    // }
    // if
    return 0;
}
```
![[771e41a517259f6a7fc9b6d5e5058afb.png|672]]实现了：
1.  创建mc.txt文件并打开 /etc/passwd文件
2.  后面的操作使用值为3的文件描述符操作/etc/passwd文件，使用值为7的文件描述符操作mc.txt文件
3.  将 /etc/passwd的文件内容从第10个字节开始全部复制到mc.txt中

后续补充：
4.  判断 /etc/passwd文件其他人是否有该文件的读权限
5.  判断mc.txt文件是否是非阻塞的
6.  所有的系统调用使用后，判断是否报错并打印报错信息
