# Makefile教程

### **1. 简介**

- **Makefile** 用于自动化编译和管理项目，类似Java的`pom.xml`、Node的`package.json`。
- `make`能自动化完成这些工作，是因为项目提供了一个`Makefile`文件，它负责告诉`make`，应该如何编译和链接程序。
- `make` 最初用于C语言，但适用于任何项目。
- 掌握Makefile有助于Linux开发和内核开发。

### **2. 安装make**

- **Windows**：使用WSL或VirtualBox安装Linux（如Ubuntu）。

- **Linux/macOS**：使用包管理器安装：

  ```apl
  sudo apt install build-essential  # Ubuntu
  brew install make gcc            # macOS
  ```

- 安装完成后，可以输入`make -v`验证：

```plain
$ make -v
GNU Make 4.3	...
```

- 输入`gcc --version`验证GCC工具链：

```plain
$ gcc --version
gcc (Ubuntu ...) 11.4.0  ...
```

这样，我们就成功地安装了`make`，以及GCC工具链。

### **3. Makefile基础**

- **规则结构**：

  makefile

  ```
  目标文件: 依赖文件
      [Tab]命令
  ```

- **执行顺序**：`make`默认执行第一条规则。

- **增量编译**：仅更新修改过的文件。

- **伪目标**：如`clean`，不生成文件，仅执行命令：

  makefile

  ```
  .PHONY: clean
  clean:
      rm -f *.o target
  ```

  

- **命令控制**：

  - `@`：不打印命令
  - `-`：忽略错误
  - `;` 或 `&&`：多命令执行

------

### **4. 编译C程序**

- **步骤**：

  1. 编译`.c` → `.o`
  2. 链接`.o` → 可执行文件

- **示例**：

  makefile

  ```
  world.out: hello.o main.o
      cc -o world.out hello.o main.o
  hello.o: hello.c
      cc -c hello.c
  main.o: main.c hello.h
      cc -c main.c
  ```

  

------

### **5. 使用隐式规则**

- `make` 内置规则，如自动将`.c`编译为`.o`。
- **缺点**：不自动追踪头文件依赖。

------

### **6. 使用变量**

- **定义变量**：

  makefile

  ```
  OBJS = hello.o main.o
  TARGET = world.out
  ```

  

- **内置变量**：如`$(CC)`表示C编译器。

- **自动变量**：

  - `$@`：目标文件
  - `$<`：第一个依赖
  - `$^`：所有依赖

------

### **7. 使用模式规则**

- 自定义模式匹配规则：

  makefile

  ```
  %.o: %.c
      cc -c -o $@ $<
  ```

  

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