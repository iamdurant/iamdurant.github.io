#### 环境
设备：redmi6 android linux-deploy chroot
架构：arm32
无法使用systemctl，无法使用docker部署应用，应用的预编译版本都为64位，无法使用

#### 编译redis
版本：7.0.15
前置依赖：build-essential tcl
准备好源代码

1. make -j6
2. make PREFIX=/tmp/redis install

耗时：10分钟

#### 编译mysql
版本：8.0.38
前置依赖：build-essential cmake libncurses5-dev libssl-dev bison libaio-dev
准备好源代码

1. mkdir build && cd build
2. cmake .. -DDOWNLOAD_BOOST=1 -DWITH_BOOST=boost `此步骤可能提示cmake版本过低，若apt存在最低要求的版本最好，直接安装，否则准备需要cmake源码，编译安装cmake：1. apt remove cmake 2. ./bootstrap 3. make -j6 4. make install 5. echo 'export PATH=/usr/local/bin:$PATH' >> ~/.bashrc 7. source ~/.bashrc`
4. make -j8
5. make install DESTDIR=/tmp/mysql

耗时：将近6小时
与此[博主](https://cloud.tencent.com/developer/article/1799089)的耗时差不多（刚开始我还以为mysql也不至于编译那么久）

**意外情况：编译到73%频繁死机，检查后，内存不足，利用scene4增大了2GB的swap后，得以继续编译**

#### mysql编译完成测试：
1. `groupadd mysql  **not use**`
2. `useradd mysql -r -g mysql -s /bin/false mysql **not use**`
3. `./mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data`
4. `./mysqld_safe --user=root &`