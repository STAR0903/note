因为没有找到合适的快速入门的博客，所以写了这篇，可以帮助我们快速掌握基础和常用内容。

# 规则

```
[target] ... : [prerequisites] ...
<tab>[command]
    ...
    ...
```

target：一个目标代表一条规则，可以是一个或多个文件名。也可以是某个操作的名字（标签），称为**伪目标**

prerequisites：前置条件，这一项是 **可选参数** 。通常是多个文件名、伪目标。它的作用是 target 是否需要重新构建的标准，如果前置条件不存在或有过更新（文件的最后一次修改时间）则认为 target 需要重新构建

command：构建这一个 target 的具体命令集

# 快速入门

### 变量

大多数脚本都支持变量，Makefile也支持变量，当在不同目标中使用相同配置时，使用变量可以让脚本更通用、以维护。定义变量使用等号：`BINARY_NAME=main.exe`, 引用变量使用 `${variable_name}`

```
// main.go

package main

import (
    "fmt"
)

func main() {
    fmt.Println("hello world")
}
```

```
#Makefile

BINARY_NAME=main.exe

build:
    go build -o ${BINARY_NAME} main.go

run:
    ./${BINARY_NAME}
```

```
Windows PowerShell:
PS E:\GOcode\gRPC\makefile> make build
go build -o main.exe main.go
PS E:\GOcode\gRPC\makefile> make run
./main.exe
hello world
```

### @符号

make 默认会打印每条命令，再执行，包括注释。这个行为被定义为 **回声** 。可以在对应命令前加上 @，可指定该命令不被打印到标准输出上。

```
#Makefile

BINARY_NAME=main.exe

build:
    go build -o ${BINARY_NAME} main.go

run:
    ./${BINARY_NAME}

echo:
    echo "test"
    @echo "test2"
```

```
Windows PowerShell:
E:\GOcode\gRPC\makefile
make echo
echo "test"
test
test2
```

### 单独使用make

单独一个make只会列表中的第一个 `target`

```
Windows PowerShell:
E:\GOcode\gRPC\makefile>make
go build -o main.exe main.go
```

### all

`all` 是 Makefile 中常见的默认目标（通常放在文件第一个目标的位置），运行 `make` 或 `make all` 时会自动执行它。

它通过依赖关系（`dependencies`）将所有需要构建的子目标（如编译程序、生成库文件等）组合在一起， **一键完成完整构建** 。

```
#Makefile

BINARY_NAME=main.exe

all: build run

build:
	go build -o ${BINARY_NAME} main.go

run:
	./${BINARY_NAME}

echo:
	echo "test"
	@echo "test2"
```

```
Windows PowerShell:
E:\GOcode\gRPC\makefile
make
go build -o ./main.exe main.go
./main.exe
hello world
```

### .PHONY

 `.PHONY`的作用是声明**伪目标** ，声明为伪目标会怎么样呢？

* 声明为伪目标后：在执行对应的命令时，make 就不会去检查是否存在 build / clean / tool / lint / help 其对应的文件，而是每次都会运行标签对应的命令
* 若不声明：恰好存在对应的文件，则 make 将会认为 xx 文件已存在，没有重新构建的必要了。

如下例：

没有声明时，clean 指令直接不执行。

```
#Makefile

BINARY_NAME=main.exe

all: build run

build:
    go build -o ${BINARY_NAME} main.go

run:
    ./${BINARY_NAME}

echo:
    echo "test"
    @echo "test2"

mk:
	echo "clean" >> clean
	echo "one" >> one.txt

clean:
	rm -f one.txt
```

```
Windows PowerShell:
E:\GOcode\gRPC\makefile
make mk
echo "clean" >> clean
echo "one" >> one.txt
E:\GOcode\gRPC\makefile
make clean
make: `clean' is up to date.
```

声明后执行指令：

```
#Makefile

.PHONY: clean 

BINARY_NAME=main.exe

all: build run
 

build:
	go build -o ${BINARY_NAME} main.go

run:
	./${BINARY_NAME}

echo:
	echo "test"
	@echo "test2"

mk:
	echo "clean" >> clean
	echo "one" >> one.txt

clean:
	rm -f "one.txt"

```

```
Windows PowerShell:
E:\GOcode\gRPC\makefile
make clean
rm -f "one.txt"
```
