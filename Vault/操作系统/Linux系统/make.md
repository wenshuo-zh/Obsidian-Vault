make是命令工具，解析Makefile文件中的指令
# Makefile
## 基础规则
### 基本原理
![[1f95d28ac91b84c17a59d3fe1739daef.png|521]]
```makefile
目标: 依赖条件
（一个Tab缩进）命令
```
单个文件：
![[06dfcc7343070172f6c12e4994b7af0a.png|520]]

![[dbc08dbb86e6648e4c3a5491f16c7e4b.png|523]]
多个文件：
![[f8cdd0444022c78254e266a42cc0cc20.png|605]]

![[8e01f12bab17c90a50111d37b689188c.png|599]]
![[Pasted image 20260922135124.png]]

### 通配符：`*`
```makefile
clean:
	rm -f *.o a.out
```
一次性删除所有`.o`文件以及a.out
### 伪目标
如果当前目录存在名为`clean`的文件，`make clean`可能认为目标已经是最新而不执行命令；`.PHONY`用于声明`clean`是伪目标，确保命令始终执行
```makefile
.PHONY:clean
clean:
	rm -f *.o a.out
```
### 变量的定义与使用
```makefile
obj = add.o sub.o main.o
main:$(obj)
	gcc $(obj) -o main
```
![[ce9c62624f48a5cf20f6d43d1dec59a1.png|313]]![[ef1f54785b7bc4fcc35a8478020212a1.png|492]]

## 常用函数、自动变量与模式规则
核心优点：只要用 `wildcard`+`patsubst` + 后缀替换 + `%.o:%.c` 模式规则，==新增`.c`源文件，完全不用手动修改 Makefile==。
### 常用函数
#### wildcard
按给定参数匹配文件名
	在 Makefile 里：直接写`*.c`，放在规则的依赖 / 命令行会自动展开；放在变量赋值里不会自动展开，这时候就要用`$(wildcard *.c)`来手动展开
```makefile
src = *.c             # 错误，src的值就只是字符串 "*.c"，不会去找文件！
src = $(wildcard *.c) # 正确，make主动扫描磁盘，匹配所有.c，得到main.c add.c sub.c
```
#### patsubst
patsubst 参数1 参数2 参数3
	将参数3中，参数1的模式，替换为参数2的模式
```makefile
src = $(wildcard *.c)
obj = $(patsubst %.c,%.o,$(src))    # 最后 obj = main.o add.o sub.o
```
### 自动变量
- $@：当前目标文件
- $<：第一个依赖文件
- $^：所有依赖文件
### 模式规则`%`
一条通用模板，一条规则可以匹配一大批相似的文件，不用给每个文件单独写一条规则。
```makefile
%.o:%.c
	gcc -c $< -o $@
```

### 示例
Makefile
![[d8400c62cfa1a2e62ef2b23c0c98d1ea.png|647]]
shell👇
![[cc3c980d11f4aeb9631c27cd0efe359a.png|655]]
![[ecd31c5b7f3d537c703f0a539f3a85f9.png|658]]
