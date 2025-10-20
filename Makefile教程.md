# Makefile教程

### **1. 简介**

- **Makefile** 用于自动化编译和管理项目，类似Java的`pom.xml`、Node的`package.json`。
- `make`能自动化完成这些工作，是因为项目提供了一个`Makefile`文件，它负责告诉`make`，应该如何编译和链接程序。
- `make` 最初用于C语言，但适用于任何项目。
- 掌握Makefile有助于Linux开发和内核开发。

------

### **2. 安装make**

- **Windows**：使用WSL或VirtualBox安装Linux（如Ubuntu）。

- **Linux/macOS**：使用包管理器安装：

  ```bash
  sudo apt install build-essential  # Ubuntu
  brew install make gcc            # macOS
  ```

- 安装完成后，可以输入`make -v`验证：

```bash
$ make -v
GNU Make 4.3	...
```

- 输入`gcc --version`验证GCC工具链：

```bash
$ gcc --version
gcc (Ubuntu ...) 11.4.0  ...
```

这样，我们就成功地安装了`make`，以及GCC工具链。

------

### **3. Makefile基础**

- **创建Makefile文件：**

  - 没有文件后缀，纯文本文件。一般约定俗成，文件名为：Makefile

    ```bash
    # 使用touch创建空文件
    touch Makefile
    
    # 或者直接用编辑器创建
    vim Makefile
    nano Makefile
    ```

  - 如果非要使用其他文件名，可以用 `-f` 参数：

    ```bash
    # 执行特定的Makefile文件
    make -f makefile.test
    make -f makefile.clean
    make -f Makefile
    ```

- **规则结构**：

  ```
  # 目标文件: 依赖文件1 依赖文件2
      [Tab]命令
  ```

  ```makefile
  #示例：合并a.txt与b.txt，生成中间文件m.txt；
  	 #生成m.txt，依赖a.txt与b.txt
  m.txt: a.txt b.txt
  	cat a.txt b.txt > m.txt
  ```

- **执行顺序**：`make`默认执行第一条规则。（把最终目标列为第一条规则）

- **伪目标**：`clean`规则与我们前面编写的规则有所不同，它没有依赖文件，因此，要执行`clean`，必须用命令`make clean`：

  ```makefile
  .PHONY: clean
  clean:
  	rm -f m.txt
  	rm -f x.txt
  ```

- **执行多条命令**:

  - `make`可以**执行指定规则**

  ```bash
  make cd_ok
  ```

  - `make`针对每条命令，都会创建一个独立的Shell环境，类似`cd ..`这样的命令，并**不会影响当前目录**。

  ```makefile
  cd:
  	pwd   
  	cd ..
  	pwd
  ```

  - 如果要切换目录——解决办法是把**多条命令以`;`分隔**，写到一行：

  ```makefile
  cd_ok:
  	pwd; cd ..; pwd;
  ```

  - 可以使用`\`把**一行语句拆成多行**，便于浏览：

    ```makefile
    cd_ok:
    	pwd; \
    	cd ..; \
    	pwd
    ```

  - 另一种执行多条命令的语法是用`&&`，它的好处是**当某条命令失败时，后续命令不会继续执行**：

    ```makefile
    cd_ok:
    	cd .. && pwd
    ```

- **控制打印：**

  - 默认情况下，`make`会打印出它执行的每一条命令。
  - 如果我们不想打印某一条命令，可以**在命令前加上`@`**，表示不打印命令（但是仍然会执行）

- **控制错误：**

  - `make`在执行命令时，会检查每一条命令的返回值，如果返回错误（非0值），就会中断执行。
  - 想忽略错误，继续执行后续命令，可以在需要忽略错误的命令前加上`-`

- 实际项目结构：

  - 布局：

    ```bash
    project/
    ├── Makefile           # 主入口
    ├── Makefile.build     # 构建规则
    ├── Makefile.test      # 测试规则  
    ├── Makefile.docker    # Docker相关
    └── Makefile.deploy    # 部署规则
    ```

    - **主Makefile内容**

    ```makefile
    # 主 Makefile - 作为入口点
    .PHONY: all build test clean deploy docker
    
    all: build
    
    build:
    	$(MAKE) -f Makefile.build
    
    test:
    	$(MAKE) -f Makefile.test
    
    clean:
    	$(MAKE) -f Makefile.build clean
    
    deploy:
    	$(MAKE) -f Makefile.deploy
    
    docker:
    	$(MAKE) -f Makefile.docker
    ```

    - **专用Makefile示例**

    ```makefile
    # Makefile.test
    .PHONY: test unit integration
    
    test: unit integration
    
    unit:
    	echo "运行单元测试..."
    	./run_unit_tests.sh
    
    integration:
    	echo "运行集成测试..."
    	./run_integration_tests.sh
    ```


------

### **4. 编译C程序**

- C程序的编译**步骤**：

  1. 编译`.c` → `.o`
  2. 链接`.o` → 可执行文件

- **示例**：假设如下的一个C项目，包含`hello.c`、`hello.h`和`main.c`。


```
text
┌─────────┐    ┌─────────┐    ┌─────────┐
│ hello.c │    │ main.c  │    │ hello.h │
└─────────┘    └─────────┘    └─────────┘
      │              │              │
      │              └──────┬───────┘
      │                     │
      ▼                     ▼
┌─────────┐          ┌─────────┐
│ hello.o │          │ main.o  │
└─────────┘          └─────────┘
      │                     │
      └─────────┬───────────┘
                │
                ▼
           ┌───────────┐
           │ world.out │
           └───────────┘
```

```makefile
# 生成可执行文件:
world.out: hello.o main.o
	cc -o world.out hello.o main.o

# 编译 hello.c:
hello.o: hello.c
	cc -c hello.c

# 编译 main.c:
main.o: main.c hello.h
	cc -c main.c

clean:
	rm -f *.o world.out
```

- #### ⚡ **增量编译 vs 全量编译**:

示例：修改hello.h——>

​		**main.c** 需要重新编译，因为函数调用方式可能改变

​		**hello.c** 需要重新编译，因为函数定义需要匹配新的声明

```bash
#全量编译
# 无论什么修改，都重新编译所有文件
$ make clean && make
rm -f *.o world.out    # 第一步：清理
cc -c hello.c          # 第二步：全量重新编译
cc -c main.c           # 第三步：全量重新编译
cc -o world.out hello.o main.o  # 第四步：重新链接
```

```bash
#增量编译
# 只修改 hello.c 时
$ make
cc -c hello.c              # 只重新编译修改的文件
cc -o world.out hello.o main.o
# 只修改 hello.h 时  
$ make
cc -c main.c               # 只重新编译依赖该头文件的文件
cc -o world.out hello.o main.o
```

------

### **5. 使用隐式规则**

- `make` 内置规则，如自动将`.c`编译为`.o`。
- **缺点**：无法跟踪`.h`文件的修改。

------

### **6. 使用变量**

- #### **变量基础用法**

  - ##### 1、**变量定义与引用**：

    - 变量定义用`变量名 = 值`(递归展开)或者`变量名 := 值`(立即展开)，通常变量名全大写。
    - 引用变量用`$(变量名)

    ```makefile
    # 定义变量
    TARGET = world.out
    OBJS = hello.o main.o
    
    # 引用变量
    $(TARGET): $(OBJS)
        cc -o $(TARGET) $(OBJS)
    ```

  - **2、内置变量：**

    - ##### **常用内置变量**

      ```makefile
      $(CC)      # C编译器 (默认: cc)
      $(CXX)     # C++编译器 (默认: g++)
      $(CFLAGS)  # C编译选项
      $(CXXFLAGS) # C++编译选项
      ```

    - 可以用变量`$(CC)`替换命令`cc`

    ```makefile
    $(TARGET): $(OBJS)
    	$(CC) -o $(TARGET) $(OBJS)
    ```

    - 没有定义变量`CC`也可以引用它，因为它是`make`的内置变量（Builtin Variables），表示C编译器的名字，默认值是`cc`，我们也可以修改它，例如使用交叉编译时，指定编译器：

      ```makefile
      CC = riscv64-linux-gnu-gcc
      ...
      ```


  - **3、自动变量**

    - ##### **常用自动变量**：

      ```makefile
      $@    # 当前规则的目标文件
      $<    # 第一个依赖文件
      $^    # 所有依赖文件
      $?    # 比目标更新的所有依赖文件
      ```

    - 示例：

    ```makefile
    # 原始写法
    world.out: hello.o main.o
        cc -o world.out hello.o main.o
    
    # 使用自动变量
    world.out: hello.o main.o
        cc -o $@ $^
    ```

    - 调试输出打印：

      ```makefile
      world.out: hello.o main.o
      	@echo '$$@ = $@' # 变量 $@ 表示target
      	@echo '$$< = $<' # 变量 $< 表示第一个依赖项
      	@echo '$$^ = $^' # 变量 $^ 表示所有依赖项
      	cc -o $@ $^
      输出：
      $@ = world.out
      $< = hello.o
      $^ = hello.o main.o
      cc -o world.out hello.o main.o
      ```

- #### **自动生成文件列表**

  - 每个`.o`文件是由对应的`.c`文件编译产生

    - 让`make`先获取`.c`文件列表
    - 再替换，得到`.o`文件列表

    ```makefile
    # $(wildcard *.c) → 获取所有 .c 文件：hello.c main.c
    # $(patsubst %.c,%.o,$(files)) → 将.c替换为.o：hello.o main.o
    OBJS = $(patsubst %.c,%.o,$(wildcard *.c))
    TARGET = world.out
    
    $(TARGET): $(OBJS)
    	cc -o $(TARGET) $(OBJS)
    
    clean:
    	rm -f *.o $(TARGET)
    ```

    - **为什么不能用 `OBJS = $(wildcard \*.o)`？**
      - `.o` 文件是编译生成的，初始时不存在
      - 使用 `.c` 文件列表可以确保所有源文件都被编译

------

### **7. **自动把`.c`文件编译成`.o`文件——使用模式规则

- **隐式规则：**

  - 隐式规则是make内置的通用构建规则，当没有显式规则匹配时自动使用。

  - **规则组成**：

    ```makefile
    $(CC) $(CFLAGS) -c -o $@ $<
    ```

    **各部分组成：**

    - `$(CC)` - C编译器（默认：`cc`）——可修改
    - `$(CFLAGS)` - 编译选项（默认：空）——可修改
    - `-c` - 编译但不链接
    - `-o $@` - 输出到目标文件
    - `$<` - 第一个依赖文件（.c文件）

- **自定义模式匹配规则：**

  - ##### 🎯 **基本语法**

    ```makefile
    %.目标扩展名: %.依赖扩展名
        命令1
        命令2
        ...
    ```


  - 

- 比隐式规则更灵活。

------

### **8. 自动生成依赖**

- 使用`gcc -MM`生成依赖关系：

  bash

  ```
  gcc -MM main.c  # 输出：main.o: main.c hello.h
  ```

  

- 将依赖写入`.d`文件，并通过`include`引入Makefile。

- 实现头文件修改自动触发重新编译。

------

### **9. 完善Makefile**

- **目录结构**：
  - `src/`：存放源码
  - `build/`：存放编译结果
- **支持功能**：
  - 自动创建目录
  - 清理生成文件
  - 引入依赖文件（`-include`忽略初始错误）