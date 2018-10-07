# 使用预置模型实现花卉图像分类应用

本文介绍在华为云ModelArts平台如何使用flowers数据集对预置的ResNet_v1\_50模型进行重训练，快速构建花卉图像分类应用。操作步骤分为4个部分，分别是：

### 1. 准备数据
下载flowers数据集并上传至华为云对象存储服务器（OBS）桶中，操作步骤如下：

**步骤 1** 下载并解压缩数据集压缩包“flower_photos.tgz”，flowers数据集的下载路径为：http://download.tensorflow.org/example_images/flower_photos.tgz 。

**步骤 2** 参考“上传业务数据”章节内容，将数据集上传至华为云OBS桶中（假设OBS桶路径为：“s3://data-tmp/flowers”）。

该路径下包含了用户训练模型需要使用的所有图像文件， 该目录下有5个子目录，代表5种类别，分别为：daisy, dandelion, roses, sunflowers, tulips。每个子目录的文件夹名称即代表该分类的label信息，每个子目录下存放对应该目录的所有图像文件，则目录结构为：

    s3://data-tmp/flowers
        |- daisy
           |- 01.jpg
           |- ...
        |- dandelion
           |- 11.jpg
           |- ...
        |- roses
           |- 21.jpg
           |- ...
        |- sunflowers
           |- 31.jpg
           |- ...
        |- tuplis
           |- 41.jpg
           |- ...

**步骤 3**  参考“访问ModelArts”章节内容，登录“ModelArts”管理控制台，单击左侧导航栏的“开发环境”。

**步骤 4** 在“开发环境”界面，单击“Notebook”，点击左上角的“创建”，在弹出框中，输入开发环境名称、描述、镜像类型、实例规格、代码存储的OBS路径等参数，单击“立即创建”，完成创建操作。

**步骤 5** 在开发环境列表中，单击所创建开发环境右侧的“打开”，进入Jupyter Notebook文件目录界面。

**步骤 6** 单击右上角的“New”，新建一个jupyter代码编辑环境，进入代码开发界面。参见数据格式转换完整代码，在Cell中填写数据代码。


```python
# 配置环境变量使得notebook可以直接访问您OBS上的数据
import moxing.tensorflow as mox
ak= 'xxxx' # 请设置为您OBS账号对应的access_key
sk= 'yyyy' # 请设置为您OBS账号对应的secret_key
server='obs.cn-north-1.myhwclouds.com' # 根据您所在的region区有关，此处以华北区为例
mox.file.set_auth(ak=ak,sk=sk,server=server)
from moxing.tensorflow.datasets.raw.raw_dataset import split_image_classification_dataset
src_dir = 's3://data-tmp/flowers'
dst_dir = 's3://data-tmp/flowers-split'
split_image_classification_dataset(
      split_spec={'train': 0.8, 'eval': 0.2},
      src_dir=src_dir,
      dst_dir=dst_dir,
      overwrite=False)
```


**步骤 7** 单击Cell上方的运行按钮 ，运行代码。将数据集按8：2的比例划分为train和eval两部分，并输出到“s3://data-tmp/flowers-split”，目录结果如下所示：

    s3://data-tmp/flowers-split
        |- train
    	    |- daisy
    	       |- 01.jpg
    	       |- ...
    	    |- dandelion
    	       |- 11.jpg
    	       |- ...
    	    |- roses
    	       |- 21.jpg
    	       |- ...
    	    |- sunflowers
    	       |- 31.jpg
    	       |- ...
    	    |- tuplis
    	       |- 41.jpg
    	       |- ...
        |- eval
    	    |- daisy
    	       |- 02.jpg
    	       |- ...
    	    |- dandelion
    	       |- 12.jpg
    	       |- ...
    	    |- roses
    	       |- 22.jpg
    	       |- ...
    	    |- sunflowers
    	       |- 32.jpg
    	       |- ...
    	    |- tuplis
    	       |- 42.jpg
    	       |- ...

### 2. 训练模型
接下来将使用训练集对预置的ResNet_v1\_50模型进行重训练获取新的模型，操作步骤如下：

**步骤 1** 返回“ModelArts”管理控制台界面。单击左侧导航栏的“训练作业”，进入“训练作业”界面。

**步骤 2** 在“算法/内置算法”列表中找到名称为“ResNet_v1\_50”的模型，其他参数确认无误后，单击“立即创建”完成训练作业创建。

"数据集"请选择训练集和验证集所在的父目录（在本案例中，即 's3://data-tmp/flowers-split'）；

**步骤 3** 在模型训练的过程中或者完成后，通过创建TensorBoard作业查看一些参数的统计信息，如loss， accuracy等。

训练作业完成后，即完成了模型训练过程。如有问题，可点击作业名称，进入作业详情界面查看训练作业日志信息。

**步骤 4** 当训练作业运行成功后，可在“作业输出路径”下查看到模型文件；然后点击“创建模型”，即可在“模型管理”也面中发布一个模型。






