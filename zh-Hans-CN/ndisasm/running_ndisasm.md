## A.3 运行 NDISASM

要反汇编一个文件，你可以象下面这样使用命令:

```shell
$ ndisasm [-b16 | -b32] filename
```

 NDISASM 可以很容易地反汇编 16 位或 32 位代码，当然，前提是你必须记得给它指定是哪种方式.如果`-b`开关没有，NDISASM 缺省工作在 16 位模式下.`-u`开关也包含 32 位模式.还有两个命令行选项，`-r`打印你正运行的 NDISASM 的版本号，`-h`给你一个有关命令行选项的简短介绍.

## A.3.1 COM 文件: 指定起点地址

 要正确反汇编一个`DOS.COM`文件，反汇编器必须知道文件中的第一条指令是被装载到地址`0x100`处的，而不是 0，NDISASM 缺省地认为你给它的每一个文件都是装载到 0处的，所以你必须告诉它这一点。

`-o`选项允许你为你正反汇编的声明一个不同的起始地址。

它的参数可以是任何NASM 数值格式:缺省是十进制，如果它以``$``或``0x``开头，或以``H`结尾，它是十六进制的，如果以``Q``结尾，它是 8 进制的，如果是``B``结尾，它是二进制的。

所以，反汇编一个`.COM`文件:

```shell
$ ndisasm -o100h filename.com
```

能够正确反汇编。

## A.3.2 代码前有数据: 同步。

假设你正反汇编一个含有一些不是机器码的数据的文件，这个文件当然也含有一些机器码。

NDISASM 会很诚实地去研究数据段，尽它的能力去产生机器指令(尽管它们中的大多数看上去很奇怪，而且有些还含有不常见的前缀，比如:`FS OR AX， 0x240A``)然后，它会到达代码段处。

假设 NDISASM 刚刚完成从一个数据段中产生一堆奇怪的机器指令，而它现在的位置正处于代码段的前面一个字节处。

它完全有可能以数据段的最后一个字节为开始产生另一个假指令，然后，代码段中的第一条正确的指令就看不到了，因为起点已经跳过这条指令，这确实不是很理想。

为了避免这一点，你可以指定一个`同步`点，或者可以指定你需要的同步点的数目(但NDISASM 在它的内部只能处理 8192 个同步点)。

同步点的定义如下:NDISASM 保证会到达这个同步点。

如果它认为某条指令会跳过一个同步点，它会忽略这条指令，代之以一个`DB`。

所以它会从同步点处开始反汇编，所以你可以看到你的代码段中的所有指令。

同步点是用`-s`选项来指定的:它们以从程序开始处的距离来衡量，而不是文件位置。

所以如果你要从 32bytes 后开始同步一个`.COM`文件，你必须这样做:

```shell
$ ndisasm -o100h -s120h file.com
```

而不是:

```shell
$ ndisasm -o100h -s20h file.com
```

就象上面所描述的，如果你需要，你可以指定多个同步记号，只要重复`-s`选项即可。

## A.3.3 代码和数据混合: 自动(智能)同步。

假设你正在反汇编一个`DOS`软盘引导扇区(可能它含有病毒，而你需要理解病毒，这样你就可以知道它可能会对你的系统造成什么样的损害)。

一般的，里面会含有`JMP`指令，然后是数据，然后接下来才是代码，所以，这很可能会让 NDISASM 不能在数据与代码交接处找不到正确的点，所以同步点是必须的。

另一方面，你为什么要手工指定同步点呢?

你要找出来的同步点的地址，当然是可以从`JMP`中读取，然后可以用它的目标地址作为一个同步点，而 NDISADM 是否可以为你做到这一点?

答案当然是可以:使用同步开关`-a`(自动同步)或`-i`(智能同步)会启用`自动同步"模式。

自动同步模式为 PC 相关的前向引用或调用指令自动产生同步点。

(因为 NDISASM是一遍的，如果它遇上一个目标地址已经被处理过的 PC 相关的跳转，它不能做什么。)

只有 PC 相关的 jump 才会被处理，因为一个绝对跳转可能通过一个寄存器(在这种情况下，NDISASM 不知道这个寄存器中含有什么)或含有一个段地址(在这种情况下，目标代码不在 NDISASM 工作的当前段中，所以同步点不能被正确的设置)对于一些类型的文件，这种机制会自动把同步点放到所有正确的位置，可以让你不必手工放置同步点。

但是，需要强调的是自动模式并不能保证找出所有的同步点，你可能还是需要手工放置同步点。

自动同步模式不会禁止你手工声明同步点:它仅仅只是把自动产生的同步点加上。

同时指定`-i`和`-s`选项是完全可行的。

关于自动同步模式，另一个需要提醒的是，如果因为一些讨厌的意外，你的数据段中的一些数据被反汇编成了 PC 相关的调用或跳转指令，NDISASM 可能会很诚实地把同步点放到所有的这些位置，比如，在你的代码段中的某条指令的中间位置。

同样，我们不能为此做什么，如果你有问题，你还是必须使用手工同步点，或使用`-k`选项(下面介绍)来禁止数据域的反汇编。



## A.3.4 其他选项。

`-e`选项通过忽略一个文件开头的 N 个 bytes 来跳过一个文件的文件头。

这表示在反汇编器中，文件头不被计偏移域中:如果你给出`-e10 -o10`，反汇编器会从文件开始的10byte 处开始，而这会对偏称域给出 10，而不是 20。

`-k`选项带有两个逗号--分隔数值参数，第一个是汇编移量，第二个是跳过的 byte 数。

这是从汇编偏移量处开始计算的跳过字节数:它的用途是禁止你需要的数据段被反汇编。