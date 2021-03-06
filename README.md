## 概述
根据RBG大神的faster-rcnn代码调整，主要是去除了标注越界错误，numpy更新错误以及cudnn7的支持。建议使用官方原生的python去编译caffe。注意Makefile.config中默认OpenCV2，如是OpenCV3请取消该文件中的注释。
具体命令顺序如下
```bash
pip install cython  
pip install easydict  
sudo apt-get install python-opencv
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler
git clone https://github.com/xs020105/faster-rcnn.git
cd py-faster-rcnn/lib
make
cd ../caffe-fast-rcnn
make -j8 && make pycaffe
```
随后将PYTHONPATH添加到环境变量中：
```bash
export PYTHONPATH=~/py-faster-rcnn/caffe-fast-rcnn/python:$PYTHONPATH 
```
配置完成后本人对的bashrc如下：
```bash
export CUDA_HOME=/usr/local/cuda
export PATH=$PATH:$CUDA_HOME/bin
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
# export PYTHONPATH=/home/nanfeicaffe/py-faster-rcnn/caffe-fast-rcnn/python/caffe:$PYTHONPATH 
export PYTHONPATH=/home/nanfeicaffe/py-faster-rcnn/caffe-fast-rcnn/python:$PYTHONPATH 
```
## 数据准备
## 预训练模型
下载ImageNet数据集下预训练得到的模型参数（用来初始化）
提供一个百度云地址：http://pan.baidu.com/s/1hsxx8OW
解压，然后将该文件放在py-faster-rcnn\data下
### 数据集放置
### VOC2007数据集
提供一个百度云地址：http://pan.baidu.com/s/1mhMKKw4
解压，然后，将该数据集放在py-faster-rcnn/data下
#### 自己的数据集
自己的数据集可以放在py-faster-rcnn/data/VOCdevkit2007/VOC2007中（替换Annotations，ImageSets和JPEGImages）
py-faster-rcnn/data/VOCdevkit2007/VOC2007/ImageSets/Main/trainval.txt存放训练的序号。
py-faster-rcnn/data/VOCdevkit2007/VOC2007/ImageSets/Main/test.txt存放测试的序号。
### 修改类别参数
在py-faster-rcnn/models/pascal_voc/ZF/faster_rcnn_alt_opt/stage1_fast_rcnn_train.pt和stage2_fast_rcnn_train.pt中修改所有num_class值为自己的类别数量加一。
bbox_pred层的num_output改为类别数加一再乘以4。
py-faster-rcnn/models/pascal_voc/ZF/faster_rcnn_alt_opt/stage1_rpn_train.pt和stage2_rpn_train.pt修改input-data层的num_class。
py-faster-rcnn/models/pascal_voc/ZF/faster_rcnn_alt_opt/faster_rcnn_test.pt修改cls_score层的num_output为类别数量加一。
bbox_pred的num_output为类别数加一以后再乘4。
### 修改标签参数
py-faster-rcnn/lib/datasets/pascal_voc.py中需要修改自己的标签，标签请用小写字母。
### 修改其他参数
学习率等之类的设置，可在py-faster-rcnn/models/pascal_voc/ZF/faster_rcnn_alt_opt中的solve文件设置，迭代次数可在py-faster-rcnn/tools的train_faster_rcnn_alt_opt.py中修改。
## 开始训练
```bash
./experiments/scripts/faster_rcnn_alt_opt.sh 0 ZF pascal_voc
```
## 测试
### 放置模型
将训练得到的py-faster-rcnn/output/faster_rcnn_alt_opt/***_trainval中ZF的caffemodel拷贝至py-faster-rcnn/data/faster_rcnn_models（如果没有这个文件夹，就新建一个）
### 放置测试图片
im_names改为自己要测试的文件名
测试图片放在py-faster-rcnn/data/demo中
### 修改标签
修改：py-faster-rcnn/tools/demo.py
其中CLASSES改为自己的标签。
随后执行
```bash
./tools/demo.py --net zf  
```
可以看到各个类的检测结果被保存在第一级目录下 