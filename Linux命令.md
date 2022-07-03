### 软件
```shell
#sudo apt-get install 之后会在路径 /var/cache/apt/archives下有对应的deb包
#知道软件库中有什么版本（或者查命令对应的软件包？）
sudo apt-cache search libjpeg*
#
apt-cache policy libjpeg-dev/*jpeg*
apt-get source 
#查询包的信息
apt-cache show libuuid1
#查看安装的软件(版本)
pkg-config --list-all 
pkg-config --modversion gtk+-3.0
#查看库/头文件信息
pkg-config --libs --cflags libturbojpeg
`pkg-config --libs --cflags gtk+-2.0`
#
dpkg -L libglib2.0-dev #列出与该包先关联的文件 
dpkg -l | grep build-essential #显示包的版本
#查看所在路径
which + 命令  
#检查上一条命令的退出状态，程序正常退出 返回0,错误退出返回非0
echo $? 
#
getconf LONG_BIT

# 查看依赖
ldd + 命令/so文件  

#编译-指定so文件
gcc jpeglib.c -o tn -L/usr/local/lib/ -ljpeg
#添加环境变量
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
source ~/.bashrc #环境变量生效
#将/etc/ld.so.conf列出的路径下的库文件缓存到/etc/ld.so.cache以供使用
ldconfig

#获得程序运行中所做的事情
strace -o main.strace ./main
cat main.strace
```



### 网络

```shell
sudo ./nmon_arm -f -t -s 1 -c 100 -m /home/linaro/
#网络配置
ifconfig eth0 192.168.1.11
ifconfig eth0 0.0.0.0 #清空
route -n
route del default gw 10.20.193.1
route add default gw 192.168.1.100
#当前IP
ifconfig | grep inet | head -1 | awk '{print $2}'
#使用TFTPSvr
tftp -g -r tums.tar.gz 192.168.1.100
 #
ping %s -c 1 -W 1 | grep "Received = 1" | wc -l
 
#nmcli
#查看信息
nmcli connect show "Wired connection 1"

nmcli connect show "Wired connection 1" | grep IP4.ADD awk | '{print $2}' #IP/掩码
nmcli c show "Wired connection 1" | grep IP4.ADD | awk '{print $2}' | cut -d "/" -f2
nmcli connect show "Wired connection 1" | grep IP4.GATEWAY #网关
nmcli connect show "Wired connection 1" | grep IP4.DNS[1]

/etc/NetworkManager/system-connections下有网络配置文件
#删除与新建
nmcli connect delete "Wired connection 1"
nmcli connection add type ethernet con-name "Wired connection 1" ifname eth0
#修改
nmcli con mod "Wired connection 1" ipv4.method auto
nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.1.199/24 ipv4.gateway 192.168.1.123 ipv4.dns "114.114.114.114 8.8.8.8"

nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 192.168.1.211/24 ipv4.gateway 192.168.1.123 ipv4.dns "114.114.114.114 8.8.8.8"
#修改生效：
nmcli connection reload
service network-manager restart

nmcli con down "Wired connection 1"
nmcli con up "Wired connection 1"

 #格式化输出net/netstat
$ cat /proc/13519/net/netstat |  awk '(f==0) {name=$1; i=2; while ( i<=NF) {n[i] = $i; i++ }; f=1; next} (f==1){ i=2; while ( i<=NF){ printf "%s%s = %d\n", name, n[i], $i; i++}; f=0} '

#mac地址
ifconfig eth0 down
ifconfig eth0 hw ether 00:0C:18:EF:FF:ED
ifconfig eth0 up 
```



```shell
#查看默认情况下pkg-config（0.24版或更高版本）在哪里搜索已安装的库; 要修改该路径，可设置PKG_CONFIG_PATH环境变量
pkg-config --variable pc_path pkg-config

```



### 硬盘

```shell
 #UUID
 blkid -s UUID
 ls -l  /dev/disk/by-uuid/
 vol_id /dev/sda1
 #内存状态
 free   
# 首先查看当前系统连接了哪些硬盘以及硬盘分区情况
$ fdisk  -l

#硬盘分区
fdisk /dev/sdc  # 开始分区: 输入m，可以查看有哪些操作;输入p 查看当前硬盘分区;输入n新建一个分区，输入p 建立分区，输入分区编号 1;最后保存分区 输入w

mkpart
parted /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary 0% 100%
parted -s /dev/sdd p

# 格式化
（echo y |）mkfs.ext4 /dev/sda
mfks -t ext3 /dev/sda6

 #硬盘的挂载与卸载
$ mount /dev/sda1 /mnt/media

$ umount /dev/sda1
$ umount /mnt/media
$ umount /dev/sda1 /mnt/media
$ umount  -l  /mnt # 卸载前检查占用该挂载文件的程序并迅速kill掉，以达到快速卸载的目的

#当有别的程序正在访问挂载的文件时，也会提示卸载失败;查看是哪个进程占用了/mnt
$ lsof  /mnt

#远程目录挂载
$ mount -t cifs -o username=root //10.20.193.10/HZRDPublic hzfile/

#查看该硬盘的文件同类型
 file -s /dev/sda1 
 
 
#Linux磁盘分区UUID的获取方法
① $ ls -l /dev/disk/by-uuid/
② 通过blkid命令   $ blkid  /dev/sda1

#查看挂载的情况
$ df -h # 查看所剩空间大小
df -hl # 查看磁盘剩余空间
df -h # 查看每个根路径的分区大小
du -sh [目录名] # 返回该目录的大小
du -sm [文件夹] # 返回该文件夹总M数
du -sm * | sort -n # 统计当前目录下层文件数,并按大小排序
du -sk * | sort -n  # k表示以1024 bytes为单位
du -m | cut -d "/" -f 2 # 看第二个/字符前的文字
## 磁盘空间
du -sh : 查看当前目录总共占的容量。而不单独列出各子项占用的容量 
du -lh --max-depth=1 : 查看当前目录下一级子文件和子目录占用的磁盘容量
```

### 文件

```shell
#NF代表：浏览记录的域的个数；$NF代表：最后一个Field(列)
ifconfig | head -1 | awk '{print $NF}' | sed 's/://g'| cut -b 5-
nmcli c show "Wired connection 1" | grep IP4.ADD | awk '{print $2}' | cut -d "/" -f1
cat /etc/resolv.conf | grep nameserver | awk '{print $NF}' | sed -n '2p'
#
find / -name 
find / -type f -size +5G   # 找到大于5G的文件
find -type f -size +500k -and -size -1000k  # 查找大小为500KB到1000KB之间的文件


# 查找当前目录下，包含“Hello”字符串的所有文件
grep -rn "get_image_compression" ./	# r表示递归，n表示查询结果显示行号

find ./ -name "*.*" | xargs grep "Hello"

find . | xargs grep -ri "Hello"

find . | xargs grep -ri "Hello" -l

#不知道文件所在的大致目录，知道文件的类型，可以在root根目录 / 下根据特定字符串进行查找
find / -type f -name "*.txt" | xargs grep "Hello"

# 将 filename 中所有的 aaa 替换为 bbb
sed -i "s/aaa/bbb/g" filename
sed 's/oldValue/newValue/g'
#s表示替换，\1表示用第一个括号里面的内容替换整个字符串，sed支持*，不支持?、+，不能用\d之类，正则支持有限
echo here365test | sed 's/.*ere\([0-9]*\).*/\1/g'
#
sed -n  '开始行，结束行p'   data.txt #显示文件X行到Y行的内容
tail -n -3 data.txt  #获取文件最后3行数据
tail -n +3 data.txt #获取文件第3行到最后一行数据

#截取字符串
df | grep cql | awk '{print $1}' | cut -d '/' -f 3
#提出数字
echo "2014年7月21日" | tr -cd "[0-9]"

#压缩
tar -czvf test.tar.gz ./test/
tar -cvf test.tar ./test/
#解压
tar -xzvf test.tar.gz
tar -xvf test.tar
tar -Jxf *.tar.xz

# 查看文件被哪个进程占用
## 文件是端口号
netstat -ntlp | grep portNum 
## 普通文件
lsof
fuser
```

### 文本

```shell
# 对每行匹配到的第一个字符串进行替换
sed -i 's/原字符串/新字符串/' ab.txt 
 
# 对全局匹配上的所有字符串进行替换
sed -i 's/原字符串/新字符串/g' ab.txt 
 
# 删除所有匹配到字符串的行
sed -i '/匹配字符串/d'  ab.txt  
 
# 特定字符串的行后插入新行
sed -i '/特定字符串/a 新行字符串' ab.txt 
 
# 特定字符串的行前插入新行
sed -i '/特定字符串/i 新行字符串' ab.txt
 
# 把匹配行中的某个字符串替换为目标字符串
sed -i '/匹配字符串/s/源字符串/目标字符串/g' ab.txt
 
# 在文件ab.txt中的末行之后，添加bye
sed -i '$a bye' ab.txt   
 
# 对于文件第3行，把匹配上的所有字符串进行替换
sed -i '3s/原字符串/新字符串/g' ab.txt 


```





### vim

```shell
#移动光标的方法

ctrl+f: 下翻一屏。
ctrl+b: 上翻一屏。
ctrl+d: 下翻半屏。
ctrl+u: 上翻半屏

HOME/END # 移动光标到行首/行尾
Page Up/Page Down # 上/下翻页
G # 移动到这个档案的最后一行(常用)
nG # n 为数字。移动到这个档案的第 n 行。
gg # 移动到这个档案的第一行，相当于 1G
n<Enter> #	n 为数字。光标向下移动 n 行
n<space> #  n 表示『数字』。按下数字后再按空格键，光标会向右移动这一行的 n 个字符。

#搜索替换
/word # 向光标之下寻找一个名称为 word 的字符串。
?word # 向光标之上寻找一个字符串名称为 word 的字符串
n #这个 n 是英文按键。代表重复前一个搜寻的动作。
N # 这个 N 是英文按键。与 n 刚好相反，为『反向』进行前一个搜寻动作。
:n1,n2s/word1/word2/g # n1 与 n2 为数字。在第 n1 与 n2 行之间寻找 word1 这个字符串，并将该字符串取代为 word2

#删除、复制与贴上
dd #删除游标所在的那一整行
ndd #n 为数字。删除光标所在的向下 n 行，如5dd
d1G #删除光标所在到第一行的所有数据
dG #删除光标所在到最后一行的所有数据
d$ #删除游标所在处，到该行的最后一个字符
d0	#那个是数字的 0 ，删除游标所在处，到该行的最前面一个字符
yy	#复制游标所在的那一行
nyy #n 为数字。复制光标所在的向下 n 行，例如 20yy 则是复制 20 行
y1G	#复制游标所在行到第一行的所有数据
yG	#复制游标所在行到最后一行的所有数据
y0	#复制光标所在的那个字符到该行行首的所有数据
y$	#复制光标所在的那个字符到该行行尾的所有数据
p, P #p 为将已复制的数据在光标下一行贴上，P 则为贴在游标上一行！
J #将光标所在行与下一行的数据结合成同一行

u #复原（撤销）前一个动作。
[Ctrl]+r	#重做上一个动作
. #小数点,意思是重复前一个动作

#批量添加注释
方法一 ：块选择模式
批量注释：
Ctrl + v 进入块选择模式，然后移动光标选中你要注释的行，再按大写的 I 进入行首插入模式输入注释符号如 // ，输入完毕之后，按两下 ESC，Vim 会自动将你选中的所有行首都加上注释，保存退出完成注释。
取消注释：
Ctrl + v 进入块选择模式，选中你要删除的行首的注释符号，注意 // 要选中两个，选好之后按 d 即可删除注释，ESC 保存退出。

方法二: 替换命令
批量注释。
使用下面命令在指定的行首添加注释。
使用名命令格式： `:起始行号,结束行号s/^/注释符/g`（注意冒号）。

取消注释：
使用名命令格式： `:起始行号,结束行号s/^注释符//g`（注意冒号）。


#windows下，每一行的结尾是\n\r，而在linux下文件的结尾是\n，那么你在windows下编辑过的文件在linux下打开看的时候每一行的结尾就会多出来一个字符\r,用cat -A urfile时你可以看到这个\r字符被显示为^M，这时候只需要删除这个字符就可以了。可以使用命令
sed -i 's/\r$//' urfile
```



### 进程

```shell
ps aux | grep ssh
pidof MediaServer
pgrep /usr/bin/dsd
pgrep -f
#指定进程名的所有进程号
pgrep -f ${progressName}
#查看某个进程打开的fd数量；pid需要替换成进程的真实pid
ls -la /proc/${pid}/fd | wc -l

#查看端口占用
lsof -i:端口号

#批量kill进程
ps -ef | grep gst | grep -v grep | awk '{print $2}' | xargs kill
#
ps -auxww
# 查看进程状态信息
cat /proc/{pid}/status
//
systemctl status NetworkManager
```

### 系统

```shell
#
cat /proc/version
file /bin/bash
uname -r
```



## Shell脚本





```shell
#判断文件、目录是否存在

-e filename 如果 filename存在，则为真
-d filename 如果 filename为目录，则为真
-f filename 如果 filename为常规文件，则为真
-L filename 如果 filename为符号链接，则为真
-r filename 如果 filename可读，则为真
-w filename 如果 filename可写，则为真
-x filename 如果 filename可执行，则为真
-s filename 如果文件长度不为0，则为真
-h filename 如果文件是软链接，则为真

#判断字符串

test –n 字符串             字符串的长度非零
test –z 字符串             字符串的长度为零
test 字符串1＝字符串2        字符串相等
test 字符串1！＝字符串2       字符串不等
```

2

```

```