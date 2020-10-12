0. 更新软件

`yum -y upgrade`

1. 创建并进入目录

`cd /home/`

`mkdir zhmy`

`mkdir app`

`cd zhmy`

2. 安装git

`yum install -y git`

下载源代码

`git clone https://github.com/pkumza/LiteRadar`

3. 放入数据文件

```shell
IOError: [Errno 2] No such file or directory: '/home/zhmy/LiteRadar/LiteRadar/Data/lite_dataset_10.csv'
```

将csvlite_dataset_10.csv文件放在/home/zhmy/LiteRadar/LiteRadar/Data/目录下

https://github.com/pkumza/Data_for_LibRadar/blob/master/lite_dataset_10.csv

4. 安装python3（centos7自带的python版本太低）

```shell
struct.error: unpack requires a string argument of length 4
```

`yum install -y python3`

5. 将需要解析的app放在/home/zhmy/app/目录下

6. 在/home/zhmy/LiteRadar/LiteRadar/目录下，运行命令解析app

`python3 literadar.py /home/zhmy/app/com.douban.frodo.apk`
