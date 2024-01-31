### CMake-Makefile Note

> [参考文档-知乎，Makefile入门](https://zhuanlan.zhihu.com/p/618350718)

#### 设计目的：

- 项目通常有多个源文件，如果只修改其中一个，就对所有源文件重新执行编译、链接步骤，就太浪费时间了。因此有必要引入 Makefile 工具：Makefile 工具可以根据文件依赖，自动找出那些需要重新编译和链接的源文件，并对它们执行相应的动作(`$ make`)

- <img title="" src="https://pic3.zhimg.com/v2-30afd31ffc5cad5c7c59b6fcbe8f10d6_r.jpg" alt="" data-align="inline">

### **Makefile三要素**

- 目标（target）、依赖（prerequisite）和执行语句（recipe）

- ![简单Makefile语句](https://pic4.zhimg.com/v2-cc4597a8e6603ffbcc622683db96332f_r.jpg)

### **变量和通配符`wildcard`函数**

> - 观察源文件的命名 `main.c` 、 `bar.c`，我们会发现它们有着共同的模式（或称为规律）：都以 `.c` 结尾，这意味着可以用这种模式匹配所有源文件。在 Makefile 中我们可以使用 wildcard 函数（wildcard function）来达到这一目的.wildcard 函数的使用方法如下：
> 
> ```makefile
> $(wildcard pattern…)
> ```

> - 如果我们想匹配当前目录下的所有源文件，就可以这样写：`$(wildcard *.c)`，其中通配符 `*` 用于匹配任意长度的任何字符，可以是 `main`、`bar`，也可以是其他<u>任何你能想得到的字符组合</u>，后面加上 `.c` 则是要求匹配的字符组合必须以 `.c` 结尾。
> 
> - ```makefile
>   #Makefile
>   main:$(wildcard *.c) 
>       gcc $(wildcard *.c) -o main
>   ```

#### **利用变量**

- 上面的 `Makefile` 还可以再优化一下可读性和效率，我们可以利用变量保存 wildcard 函数展开后的结果。Makefile 中变量定义的形式与C语言类似：`var := value`，调用则和函数调用类似：`$(var)`，所以 `Makefile` 可以进一步修改为

- ```makefile
  #Makefile
  srcs:=$(wildcard *.c)
  main:$(srcs) 
      gcc $(srcs) -o main
  ```

#### **变量赋值与修改**

- 刚才的示例中使用到了赋值符号 `:=` ，该符号与C语言中的赋值符号 `=` 作用效果相同。以下是几个常用符号的简介：
  
  - `=` ：递归赋值[(Recursively Expanded Variable Assignment)](https://link.zhihu.com/?target=https%3A//www.gnu.org/software/make/manual/html_node/Recursive-Assignment.html)，使用变量进行赋值时，会优先展开引用的变量，示例如下：
    
    - ```makefile
      foo = $(bar)
      bar = $(ugh)
      ugh = Huh?
      all:;echo $(foo)
      # 打印的结果为 Huh?，$(foo)展开得到$(bar)，$(bar)展开得到$(ugh)，$(ugh)展开得到Huh?最终$(foo)展开得到Huh?
      ```
  
  - `:=` ：简单赋值[(Simply Expanded Variable Assignment)](https://link.zhihu.com/?target=https%3A//www.gnu.org/software/make/manual/html_node/Simple-Assignment.html)，最常用的赋值符号：
  
  ```makefile
  x := foo
  y := $(x) bar
  x := later
  # 等效于：
  # y := foo bar
  # x := later
  ```
  
  - `+=` ：文本增添[(Appending)](https://link.zhihu.com/?target=https%3A//www.gnu.org/software/make/manual/html_node/Appending.html)，用于向已经定义的变量添加文本：
  
  ```makefile
  objects = main.o foo.o bar.o utils.o
  objects += another.o
  # objects最终为main.o foo.o bar.o utils.o another.o
  ```
  
  - `?=` ：条件赋值[(Conditional Variable Assignment)](https://link.zhihu.com/?target=https%3A//www.gnu.org/software/make/manual/html_node/Conditional-Assignment.html)，仅在变量没有定义时创建变量：
  
  ```makefile
  FOO ?= bar
  # FOO最终为bar
  foo := ugh
  foo ?= Huh?
  # foo最终为ugh
  ```

### 进阶的Makefile

- 实际面对的场景往往要复杂得多：源文件和头文件按照功能或层级区分，散落在一个个子文件夹下，这样做更容易管理工程文件，但也带来了两点小麻烦
  
  (目录结构：/main.c+Makefile+ ./test文件夹/test.h+test.c  )
  
  - 编译 `main.c` 时提示找不到 `test.h` 的头文件，这是编译时没有指定到哪些路径下寻找头文件导致的，解决办法是执行 `gcc` <u>命令时通过 `-I` 选项指定头文件所在路径</u>
    
    - ```makefile
      #Makefile
      incs:=-I./test
      srcs:=$(wildcard *.c)
      main:$(srcs) 
          gcc $(srcs) $(incs) -o main 
      ```
  
  - 但是！！执行 `$ make` 时又发生了错误，提示主函数中调用了未定义的函数 `Print_Progress_Bar`，这个函数定义在 `test.c` 中。仔细观察可以发现 `gcc` 的调用中缺少 `test.c`，这就引发了我们遇到的问题。显然在 `test.c` 装进 `./test` 目录后，`Makefile` 就找不到 `test.c` 文件了，这就是我们在刚才提到的小麻烦。
    
    > - [x] ### 应对复杂的目录结构
    > 
    > - 看一下 `make` 的报错问题如何解决。思路和方法很简单，使用 `wildcard `函数在 `./test` 目录下也匹配一遍源文件，再把这些源文件一同添加到 `SRCS` 变量中就可以了：
    > 
    > - ```makefile
    >   #Makefile
    >   incs:=-I./test
    >   srcs:=$(wildcard *.c)
    >   srcs+=$(wildcard ./test/*.c)
    >   main:$(srcs) 
    >       gcc $(srcs) $(incs) -o main 
    >   ```
    > 
    > - 还有什么办法能将 `Makefile` 写得更清晰一些?(手动将目录添加到 `Makefile` 中去)
    >   
    >   - ```makefile
    >     # Makefile
    >     SUBDIR := .
    >     SUBDIR += ./test
    >     ```
    > 
    > - 这里定义了变量 `SUBDIR`，使用它来指定那些存放着源文件和头文件的目录。接下来请出<mark>另一个功能强大的函数 foreach </mark>来帮助完成一项复杂的功能(接近 Python 的 for 循环。它的功能描述起来就是：从 `list` 中逐个取出元素，赋值给 `var`，然后再展开 `text`)
    >   
    >   - ```makefile
    >     $(foreach var,list,text)
    >     #示例：
    >     SUBDIR := .
    >     SUBDIR += ./func
    >     EXPANDED := $(foreach dir,$(SUBDIR),$(dir)/*.c)
    >     # 等效于EXPANDED := ./*.c ./func/*.c
    >     ```
    >   
    >   - ![](https://pic1.zhimg.com/v2-e717c574ff60b15fabfc906898f89bc0_r.jpg)
    >   
    >   - 有了 foreach 函数，我们就能配合 `wildcard`函数，通过指定路径来获取源文件，并指定头文件所在路径(现在来看，指定路径的做法较之前并没有太大的优势，我们要做的仍是手动指定目录，只是将获取源文件的任务交给了 `foreach` 函数来完成)：
    >     
    >     - ```makefile
    >       #Makefile
    >       
    >       SUBDIR:=.
    >       SUBDIR+=./test
    >       
    >       incs:=$(foreach dir,$(SUBDIR),-I$(dir))
    >       srcs:=$(foreach dir,$(SUBDIR),$(wildcard $(dir)/*.c))
    >       
    >       main:$(srcs)
    >           gcc $(incs) $(srcs) -o main
    >       ```
    > 
    > - 

### 

### 分析编译过程

目前为止，示例程序还保持着较短的编译、链接时间。但当源文件逐渐增多后，只改动其中一个源文件，我们还能在短时间内获得可执行文件吗？为了解答这个问题，我们先来回顾一下编译、链接的过程。

源文件和头文件需要经过四个步骤才能得到可执行文件，分别是预处理、编译、汇编和链接。

- 预处理：预处理器将以字符 `#` 开头的命令展开、插入到原始的C程序中。比如我们在源文件中能经常看到的、用于头文件包含的 `#include` 命令，它的功能就是告诉预编译器，将指定头文件的内容插入的程序文本中。

- 编译阶段：编译器将文本文件 `*.i` 翻译成文本文件 `*.s`，它包含一个汇编语言程序。

- 汇编阶段：汇编器将 `*.s` 翻译成机器语言指令，把这些指令打包成可重定位目标程序（relocatable object program）的格式，并保存在 `*.o` 文件中。

- 链接阶段：在 `test.c` 中我们定义了 `Print_Progress_Bar` 函数，该函数会保存在目标文件 `test.o` 中。直到链接阶段，链接器才以某种方式将 `Print_Progress_Bar` 函数合并到 `main` 函数中去。在链接时如果没有指定 `test.o`，链接器就无法找到 `Print_Progress_Bar` 函数，也就会提示找不到相关函数的定义。

> - <img title="" src="https://pic3.zhimg.com/v2-dce03b81e9ae6005373604876cb34272_r.jpg" alt="" width="584" data-align="center">

### 是否保存 *.o 文件？

从编译过程的分析中，能找到当前 `Makefile` 存在的两点问题：

- - <img src="https://pic1.zhimg.com/v2-207e3ef4ecbbe2e046683c9c44399260_r.jpg" title="" alt="" width="509">
1. 没有保存 `.o` 文件，这导致我们每次文件变动都要重新执行预处理、编译和汇编来得到目标文件，即使新得到的文件与旧文件完全没有差别（即编译用到的源文件没有任何变化，就跟`test.c` 一样）。

2. 有保存 `.o` 文件，则会遇到第二个问题，即依赖中没有指定头文件，这意味着只修改头文件的情况下，源文件不会重新编译得到新的可执行文件！
- > 为了证明以上两个问题，我们对 `Makefile` 做一些改动(`gcc` 命令指定 `-c` 选项后，会只执行编译步骤，而不执行链接步骤，最后得到 `*.o` 文件。这里我们添加新的目标和依赖，目的是编译得到 `main.o bar.o`，最后再手动将它们链接为可执行文件 `main`。值得一提的是 Makefile 文件会自动匹配依赖和目标，如果依赖的依赖有更新，则目标文件也会得到更新。)：

- ```makefile
  #Makefile
  subdir:=.
  subdir+=./test
  
  incs:=$(foreach dir,$(subdir),-I$(dir))
  
  main:./main.o ./test/test.o
      gcc ./main.o ./test/test.o -o main
  
  ./main.o:./main.c
      gcc -c $(incs) ./main.c -o ./main.o
  
  ./test.o:./test/test.c
      gcc -c $(incs) ./test/test.c -o ./test/test.o
  ```

> `make` 执行了我们指定的每一个步骤。现在让我们修改 `main.c`，手动删除 `test.o` 后再执行 `make`。（模拟不保存 `*.o` 文件的情况）
> 
> - ```c
>   // main.c
>   #include <stdio.h>
>   #include "./test/test.h"
>   int main(){
>           printf("Happy Birth Day!\n");
>           Print_Progress_Bar(66.0f/100.0f);
>           return 0;
>   }
>   ```

> 执行`$ make`:不仅重新编译了 `main.o`，还重新编译了 `test.o`，现在再试试保存 `test.o` 的情况下执行 `$ make`(只修改 main.c文件):
> 
> > - 相较于不保存 `test.o` 的情况，我们少执行了 `test.o` 的编译步骤，这对于工程文件编译速度的提升，可能是巨大的！
> 
> ##### 那如果修改的不是源文件而是头文件呢？
> 
> > 不出所料，源文件（test.c）果然没有重新编译。
> > 
> > **结论：所以很多 .o文件都可以保留从而节省编译时间，这也是Makefile的设计目的,但是头文件的依赖关系改变了该如何解决呢？**

### 模式规则和自动变量

- 我们还是先来解决问题，首先是 `*.o` 文件的保存问题，这个问题其实在上面已经解决了——通过手动添加目标和依赖（./main.o:./main.c  gcc -c $(incs) ./main.c -o ./main.o
  
    ./test.o:./test/test.c   gcc -c $(incs) ./test/test.c -o ./test/test.o），我们实现了 `*.o` 文件的保存，同时还确保了源文件在更新后，只会在最小限度内重新编译 `*.o` 文件。

- 现在我们可以利用符号 `%` 和自动变量，来让 `Makefile` 变得更加通用。首先聚焦于编译过程：
  
  - ```makefile
    ./main.o:./main.c
        gcc -c $(incs) ./main.c -o ./main.o
    
    ./test.o:./test/test.c
        gcc -c $(incs) ./test/test.c -o ./test/test.o
    ```

> 上下比较 `./main.o` 和 `./test/test.o` 的目标依赖及执行，可以发现新添加的、用于生成 `*.o` 文件的目标和依赖，有着相同的书写模式，这意味着存在通用的:
> 
> - ```makefile
>   %.o : %.c
>           gcc -c $(INCS) $< -o $@
>   ```
> 
> - ![](https://pic4.zhimg.com/80/v2-dc3eb15d71fc4ac33a5523703eeb86eb_720w.webp)
> 
> - > 这里用上了 `%` ，它的作用有些难以用语言概括，上述例子中， `%.o` 的作用是匹配所有以 `.o` 结尾的目标；而后面的 `%.c` 中 `%` 的作用，则是将 `%.o` 中 `%` 的内容原封不动的挪过来用。
>   > 
>   > 更具体地例子是，`%.o` 可能匹配到目标 `./entry.o` 或 `./func/bar.o`，这样 `%` 的内容就会是 `./entry` 或 `./func/bar`，最后交给 `%.c` 时就变成了 `./entry.c` 或 `./func/bar.c`。
>   > 
>   > 另外我们还使用到了自动变量 `$< $@`，其中 `$<` 指代依赖列表中的第一个依赖；而 `$@` 指代目标。注意自动变量与普通变量不同，它不使用小括号。
>   
>   - 结合起来使用，我们就得到了通用的生成 `*.o` 文件的写法：
>     
>     - ```makefile
>       #Makefile
>       subdir:=.
>       subdir+=./test
>       incs:=$(foreach dir,$(subdir),-I$(dir))
>       %.o:%.c
>           gcc -c $(incs) $< -o $@
>       
>       main:./test/test.o ./main.o
>           gcc ./test/test.o ./main.o -o main
>       ```

> - ### patsubst 函数
>   
>   接下来再让我们关注链接步骤，它需要指定 `*.o` 文件：
>   
>   ```makefile
>   main : ./main.o ./test/test.o
>           gcc ./main.o ./test/test.o-o main
>   ```
>   
>   这看起来十分眼熟，我们最初解决多文件编译问题时也是采用类似的写法，只有文件后缀不一样：
>   
>   ```makefile
>   main : ./main.c ./test/test.c
>           gcc ./main.c ./test/test.c -o main
>   ```

> - 可惜的是，在最开始我们是无法匹配到 `*.o` 文件的，因为起初我们只有 `*.c` 文件， `*.o` 文件是后来生成的。但转换一下思路，我们在获取所有源文件后，直接将 `.c` 后缀替换为 `.o`，不就能得到所有的 `.o` 文件了吗？正巧 patsubst 函数可以用于模式文本替换。
> 
> ```makefile
> $(patsubst pattern,replacement,text)
> ```
> 
> `patsubst` 函数的作用是匹配 `text` 文本中与 `pattern` 模式相同的部分，并将匹配内容替换为 `replacement`。于是链接步骤可以改写为：
> 
> - ```makefile
>   SRCS := $(foreach dir,$(SUBDIR),$(wildcard $(dir)/*.c))
>   OBJS := $(patsubst %.c,%.o,$(SRCS))
>   
>   main : $(OBJS)
>           gcc $(OBJS) -o main
>   ```
> 
> - ![](https://pic4.zhimg.com/80/v2-7fde6378fb2e5f895d08fffcd04c901f_720w.webp)
> 
> - 这里我们先用 wildcard 函数获取所有的 `.c` 文件，并将结果保存在 `SRCS` 中，接着利用 patsubst 函数替换 `SRCS` 的内容，最后将所有的 `.c` 替换为 `.o` 以获得执行编译所得到的目标文件。
> 
> - 于是我们的 `Makefile` 就可以改写为：
>   
>   ```makefile
>   #Makefile
>   subdir:=.
>   subdir+=./test
>   
>   incs:=$(foreach dir,$(subdir),-I$(dir))
>   src:=$(foreach dir,$(subdir),$(wildcard $(dir)/*.c))
>   
>   OBJS:=$(patsubst %.c,%.o,$(src))
>   
>   main:$(OBJS)
>       gcc $(OBJS) -o main
>   
>   %.o:%.c
>       gcc -c $(incs) $< -o $@
>   ```

## 丰富完善Makefile的功能

- 目前为止，我们已经写出足够使用的 Makefile 文件了，接下来我们可以继续完善它的功能。

###### 指定*.o文件的输出路径

- 目前编译得到的 `.o` 文件，都是放在与 `.c` 文件同一级目录下的，从代码编辑习惯考虑，这可能会导致我们无法方便地寻找源文件或头文件。理想的做法是将 `*.o` 文件保存至指定目录，与源文件和头文件区分开

```shell
├── Makefile
├── main.c
├── test
│   ├── test.c
│   └── test.h
├── main
└── output
    ├── main.o
    └── func
        └── main.o
```

单个 `.o` 文件怎么输出到指定文件夹?：

```makefile
./output/func/test.o : ./test/test.c
        mkdir -p ./output/func
        gcc -c $(INCS) ./test/test.c -o ./output/func/test.o
```

解决问题的思路是：把输出目录下的 `.o` 文件作为编译目标，原始目录下的 `.c` 文件作为依赖，来编译得到目标文件。这里我们需要解决两个问题：

1. 如何得到 `./output/func/test.o` 这个路径；

2. 如何保证 `./output/func` 目录存在。

问题1我们可以在执行 patsubst 函数时解决，就像这样：

```makefile
OUTPUT := ./output
SRCS := $(foreach dir,$(SUBDIR),$(wildcard $(dir)/*.c))
OBJS := $(patsubst %.c,$(OUTPUT)/%.o,$(SRCS))
```

接着问题2我们可以使用 `mkdir`命令配合 `dir`函数解决，`dir`函数可以从文本中获得路径，配合`mkdir -p `命令创建目录，可以确保输出路径存在:

- ```makefile
  mkdir -p $(dir ./output/func/test.o)
  #dir 函数会把 ./output/func/test.o 修改成 ./output/func，
  #于是上面的命令就变为了：
  #mkdir -p ./output/func
  ```

> 因此，最终的Makefile 长这样：
> 
> ```makefile
> SUBDIR := ./
> SUBDIR += ./func
> 
> OUTPUT := ./output
> 
> INCS := $(foreach dir,$(SUBDIR),-I$(dir))
> SRCS := $(foreach dir,$(SUBDIR),$(wildcard $(dir)/*.c))
> OBJS := $(patsubst %.c,$(OUTPUT)/%.o,$(SRCS))
> 
> main : $(OBJS)
>         gcc $(OBJS) -o main
> 
> $(OUTPUT)/%.o : %.c
>         mkdir -p $(dir $@)
>         gcc -c $(INCS) $< -o $@
> ```

### 

### 伪目标

> 在 Makefile 中我们可以利用目标执行某些动作。比如定义一个 `clean` 目标，用于清理编译生成的过程文件：

```makefile
OUTPUT := ./output
clean:
        rm -r $(OUTPUT)
```

- 但这样做存在隐患：当前目录下有与目标同名的文件时，在没有依赖的情况下Makefile 会认为目标文件已经是最新的状态了，目标下的语句也就不再执行。为解决这一问题，我们可以将 `clean` 声明为伪目标，表明其并非是文件的命名。向特殊内置目标 `.PHONY` 添加 `clean` 依赖以达成这一目的：

- ```makefile
   .PHONY : clean
  
  OUTPUT := ./output
  clean:
          rm -r $(OUTPUT)
  ```

- 添加上 `.PHONY : clean` 后再执行 `$ make clean`,可以看到 `clean` 目标下的 `rm -r $(OUTPUT)` 得到了执行。



### 简化终端输出

现在我们的 Makefile 所输出的内容会有些杂乱无章，我们很难直观看出哪条命令在编译哪个文件。所以我们常通过 `@` 符号，来禁止 Makefile 将执行的命令输出至终端上：

```text
$(OUTPUT)/%.o : %.c
        mkdir -p $(dir $@)
        @gcc -c $(INCS) $< -o $@
```

> 同时你可能注意到 Makefile 中是可以使用终端命令的，所以我们可以用 echo 命令来拟定自己的输出信息：
> 
> ```makefile
> SUBDIR := ./
> SUBDIR += ./test
> 
> OUTPUT := ./output
> 
> INCS := $(foreach dir,$(SUBDIR),-I$(dir))
> SRCS := $(foreach dir,$(SUBDIR),$(wildcard $(dir)/*.c))
> OBJS := $(patsubst %.c,$(OUTPUT)/%.o,$(SRCS))
> 
> main:$(OBJS)
> 	@echo linking...
> 	@gcc $(OBJS) -o main
> 	@echo Complete!
> 
> $(OUTPUT)/%.o : %.c
> 	@echo compile $<...
> 	@mkdir -p $(dir $@)
> 	@gcc -c $(INCS) $< -o $@
> 
> .PHONY : clean
> clean:
> 	@echo try to clean...
> 	@rm -rf $(OUTPUT)
> 	@echo Complete!
> ```





- [x] ### 自动生成依赖

- 还记得修改头文件后，包含该头文件的源文件不会重新编译的问题吗？现在让我们试试看解决这个问题。

问题的解决思路也很简单，就是将头文件一同加入到 *.o 文件的依赖中：

```makefile
./main.o : ./main.c ./test/test.h
        gcc -c $(INCS) ./main.c -o ./main.o
```

> 但这实现起来并不容易，我们需要在 Makefile 中为每个源文件单独添加头文件依赖，手动维护这些依赖关系会是一件极其痛苦的事情。幸运的是，gcc 提供了强大的自动生成依赖功能，仅需在<mark>编译时指定 `-MMD` 选项</mark>，就能得到记录有依赖关系的 *.d 文件。
> 
> 

> > `-MMD` 选项包含两个动作，一是生成依赖关系，二是保存依赖关系到 *.d 文件。与其类似的选项还有 `-MD`，其作用与 `-MMD` 相同，差别在于 `-MD` 选项会将系统头文件一同添加到依赖关系中。
> > 
> > - <mark>另外我们还可以指定 `-MP` 选项</mark>，这会为每个依赖添加一个没有任何依赖的伪目标。`-MP` 选项生成的伪目标，可以有效避免删除头文件时，Makefile 因找不到目标来更新依赖所报的错误：`` `make: *** No rule to make target 'dummy.h', needed by 'dummy.o'. Stop. ``



**接着我们还需要将 *.d 文件记录的依赖关系，手动包含到 Makefile 中，这样才能使其真正发挥作用。所以 Makefile 又可以修改为：**

> `-MMD` 选项生成的 *.d 文件保存在与* .o 文件相同的路径下。

```makefile
SUBDIR := ./
SUBDIR += ./test

OUTPUT := ./output

INCS := $(foreach dir,$(SUBDIR),-I$(dir))
SRCS := $(foreach dir,$(SUBDIR),$(wildcard $(dir)/*.c))
OBJS := $(patsubst %.c,$(OUTPUT)/%.o,$(SRCS))
DEPS := $(patsubst %.o,%.d,$(OBJS))

main : $(OBJS)
        @echo linking...
        @gcc $(OBJS) -o main
        @echo Complete!

$(OUTPUT)/%.o : %.c
        @echo compile $<...
        @mkdir -p $(dir $@)
        @gcc -MMD -MP -c $(INCS) $< -o $@

.PHONY : clean
clean:
        @echo try to clean...
        @rm -r $(OUTPUT)
        @echo Complete!

-include $(DEPS)
```

> 最后一行的 `include` 用于将指定文件的内容插入到当前文本中。初次编译，或者 make clean 后再次编译时，*.d 文件是不存在的，这通常会导致 include 操作报错。所以我们在 `include` 前加了 `-` 符号，其作用是指示 make 在 include 操作出错时忽略这个错误，不输出任何错误信息并继续执行接下来的操作。

 









- [x] ### 通用模板

文章的末尾，放一个通用的 Makefile 模板吧！

```makefile
ROOT := $(shell pwd)
#获取当前Makefile所在的目录路径，并将其赋值给变量ROOT。这样，你可以在Makefile的其他部分使用$(ROOT)来引用这个路径。
#pwd是Unix/Linux shell中的一个命令，用于打印当前工作目录（Present Working Directory）的路径

SUBDIR := $(ROOT)
SUBDIR += $(ROOT)/func

TARGET := main
OUTPUT := ./output

INCS := $(foreach dir,$(SUBDIR),-I$(dir))
SRCS := $(foreach dir,$(SUBDIR),$(wildcard $(dir)/*.c))
OBJS := $(patsubst $(ROOT)/%.c,$(OUTPUT)/%.o,$(SRCS))
DEPS := $(patsubst %.o,%.d,$(OBJS))

$(TARGET) : $(OBJS)
        @echo linking...
        @gcc $(OBJS) -o $@
        @echo complete!

$(OUTPUT)/%.o : %.c
        @echo compile $<...
        @mkdir -p $(dir $@)
        @gcc -MMD -MP -c $(INCS) $< -o $@

.PHONY : clean

clean:
        @echo try to clean...
        @rm -r $(OUTPUT)
        @echo complete!

-include $(DEPS)
```











---------------------------

##### *补充资料：[Makefile入门(超详细一文读懂)_makefile](http://681314.com/A/0cNlR5d8Xv)*

1. Makefile编译过程：
   
   - makefile文件编写好以后在Linux命令行中执行一条make命令即可自动编译整个工程,落在目标依赖上最广泛使用的是<u>*GNUmake*</u>

2. Makefile语法规则
   
   > ```makefile
   > 目标 ... : 依赖 ...
   >     命令1
   >     命令2
   >     . . .
   > ```

> > Makefile的核心规则类似于一位厨神做菜目标就是做好一道菜那么所谓的依赖就是各种食材各种厨具等等，然后需要厨师好的技术方法类似于**命令**才能作出一道好菜。同时这些依赖也有可能此时并不存在需要现场制作或者是由其他厨师做好那么这个依赖就成为了其他规则的目标，该目标也会有他自己的依赖和命令。这样就形成了一层一层递归依赖组成了Makefile文件。Makefile并不会关心命令是如何执行的仅仅只是会去执行所有定义的命令和我们平时直接输入命令行是一样的效果。
> > 
> > > 1. 目标**即要<u>生成的文件</u>。如果目标文件的更新时间晚于**依赖**文件更新时间则说明依赖文件没有改动目标文件不需要重新编译。否则会进行重新编译并更新目标文件
> > > 
> > > 2. 默认情况下Makefile的第一个目标为终极目标
> > > 
> > > 3. 依赖即目标文件由哪些文件生成。
> > > 
> > > 4. 命令即通过<u>执行命令由依赖文件生成目标文件</u>。注意每条命令之前必须有一个tab保持缩进这是语法要求,会有一些编辑工具默认tab为4个空格会造成Makefile语法错误。
> > > 
> > > 5. all参数,Makefile文件默认只生成第一个目标文件即完成编译但是我们可以通过all 指定所需要生成的目标文件

3. 变量
   
   `$`符号表示取变量的值,当变量名多于一个字符时使用`( )`,举例与其他用法:
   
   ```makefile
   $^ 表示所有的依赖文件
   $@ 表示生成的目标文件
   $< 代表第一个依赖文件
   
   SRC = $(wildcard *.c)
   OBJ = $(patsubst %.c, %.o, $(SRC))
   ALL: hello.out
   hello.out: $(OBJ)
           gcc $< -o $@
   
   $(OBJ): $(SRC)
           gcc -c $< -o $@
   ```

4. 变量赋值
   
   ```makefile
   1、"="是最普通的等号在Makefile中容易搞错赋值等号使用 “=”进行赋值变量的值是整个Makefile中最后被指定的值。
   
   VIR_A = A
   VIR_B = $(VIR_A) B
   VIR_A = AA
   
     经过上面的赋值后最后VIR_B的值是AA B而不是A B在make时会把整个Makefile展开来决定变量的值
     2、":=" 表示直接赋值赋予当前位置的值。
   
   VIR_A := A
   VIR_B := $(VIR_A) B
   VIR_A := AA
   
     最后BIR_B的值是A B即根据当前位置进行赋值。因此相当于“=”“=”才是真正意义上的直接赋值
     3、"?=" 表示如果该变量没有被赋值赋值予等号后面的值。
   
   VIR ?= new_value
   
     如果VIR在之前没有被赋值那么VIR的值就为new_value。
   
   VIR := old_value
   VIR ?= new_value
   
     这种情况下VIR的值就是old_value
     4、"+="和平时写代码的理解是一样的表示将符号后面的值添加到前面的变量上
   ```

5. 预定义变量
   
   ```makefile
   cpp c预编译器的名称默认值为$(CC) -E
   CC = gcc
   回显问题Makefile中的命令都会被打印出来。如果不想打印命令部分 可以使用@去除回显
   @echo "clean done!"
   ```

6. ##### Makefile工作原理
   
   > 目标的生成： a. 检查规则中的依赖文件是否存在； b. 若依赖文件不存在，则寻找是否有规则用来生成该依赖文件。
   > 
   > ![](https://img-blog.csdnimg.cn/004c160f555f4007a0d30791a1b07460.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAQ-WQm-iOq-eskQ==,size_20,color_FFFFFF,t_70,g_se,x_16)



> 目标的更新： a. 检查目标的所有依赖，任何一个依赖有更新时，就重新生成目标； b. 目标文件比依赖文件时间晚，则需要更新。
> 
> > <img src="https://img-blog.csdnimg.cn/40331a847c0240148f4d751c4532bb2d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAQ-WQm-iOq-eskQ==,size_20,color_FFFFFF,t_70,g_se,x_16" title="" alt="" width="467">
