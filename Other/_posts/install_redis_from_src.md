在使用redis源码安装redis时，遇到一些问题，这里记录一下。

使用源码版本：
	redis-4.0.14.tar.gz

解压后到目录下执行
make

发生如下错误：
```
../deps/jemalloc/lib/libjemalloc.a(nstime.o): In function `nstime_get':
/root/redis-4.0.14/deps/jemalloc/src/nstime.c:120: undefined reference to `clock_gettime'
collect2: ld returned 1 exit status
make[1]: *** [redis-server] Error 1
make[1]: Leaving directory `/root/redis-4.0.14/src'
make: *** [all] Error 2
```

在网上搜索后，是因为clock_gettime这个函数会用到librt这个库，但是在gcc链接的参数中没有指定-lrt参数

解决方案：
修改src/Makefile，在FINAL_LIBS变量中加入-lrt
```
else
        # All the other OSes (notably Linux)
        FINAL_LDFLAGS+= -rdynamic
        FINAL_LIBS+=-ldl -pthread -lrt #此处加上-lrt参数
endif
```
