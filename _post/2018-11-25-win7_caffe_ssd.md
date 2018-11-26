环境

window7 旗舰版

华硕 gtx1070 8g gaming

python 2.7.15(adaconda)

cuda8.0

caffe-ssd-microsoft





下载ssd:

git clone <https://github.com/conner99/caffe.git>

cd caffe

git checkout ssd-microsoft







# win7下 caffe python接口配置 import caffe时报错，ImportError: No module named google.protobuf.internal

win7下，python是安装的Anaconda2，这个工具好，帮你安装了好多能用到的库如numpy,scripy等。

我在配置caffe的python接口时，将编译好的python的caffe文件拷贝到python安装目录C:\ProgramData\Anaconda2\Lib\site-packages下。







在python命令窗口中输入import caffe报错：

ImportError: No module named google.protobuf.internal  

解决方法如下：

0、cmd命令行下输入: pip install protobuf，安装成功后进行后续步骤；

1、下载win64的protobuf 点这里；





2、可对比python安装目录下的Lib\site-packages下的内容，将google文件夹拷贝到



3、同样的方法将Library下的内容拷贝到python对应的目录，再次输入import caffe不报错了。



查看protobuf是否安装成功，可以在命令行输入protoc --version，出现如下说明成功安装。

---------------------




