对象结构

Java对象包含了三个部分:对象头，实例数据，填充字节。

对象头存放了对象本身的运行时信息。

实例数据就是初始化对象时候设置的属性和方法等内容。

填充字节是为了让对象依然是8bit的倍数。



对象头占了32bit。存放了对象本身的运行时信息。包含Mark Word和Class Point。前者存储了当前对象运行时状态有关的信息。后者为一个指针，指向了当前对象类型所在方法区中的类型数据。

Mark Word的最后两位是锁标志位。

