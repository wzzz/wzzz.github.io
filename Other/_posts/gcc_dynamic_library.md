关于gcc编译链接的动态库目录和运行可执行程序时的动态库目录
gcc编译时链接的动态库目录是由-L和-l指定
运行时的动态库是由LD_LIBRARY_PATH变量指定的目录以及/etc/ld.so.conf中指定的目录指定的
优先从LD_LIBRARY_PATH中查找，再从ld.so.conf中查找

## cmake指定安装目录
cmake -DCMAKE_INSTALL_PREFIX=/usr .

## make指定安装目录
make install DESTDIR=./install
make install PREFIX=/data/zhangcd/redis-3.2.9/


## 设置core

https://blog.csdn.net/u011417820/article/details/71435031

ulimit -c unlimited
sysctl -w kernel.core_uses_pid=1
sysctl -w kernel.core_pattern=/data/corefiles/core-%e-%t-%p
mkdir -p /data/corefiles
sudo chmod 777 /data/corefiles/
