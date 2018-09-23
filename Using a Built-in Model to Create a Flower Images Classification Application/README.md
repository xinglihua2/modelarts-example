# 使用预置模型实现花卉图像分类应用

本文介绍在华为云ModelArts平台如何使用flowers数据集对预置的ResNet_v1\_50模型进行重训练，快速构建花卉图像分类应用。操作步骤分为4部分，分别是：

1.	**准备数据**：下载flowers数据集，并上传至华为云对象存储服务器（OBS）中，并将数据集划分为训练集和验证集。
2.	**训练模型**：使用flowers训练集，对ResNet_v1\_50模型重训练，得到新模型。
3.	**部署模型**：将得到的模型，部署为在线预测服务。
4.	**发起预测请求**：下载客户端工程，发起预测请求获取请求结果。
### 1. 准备数据
下载flowers数据集并上传至华为云对象存储服务器（OBS）桶中，操作步骤如下：

**步骤 1** &#160; &#160; 下载并解压缩数据集压缩包“flower_photos.tgz”，flowers数据集的下载路径为：http://download.tensorflow.org/example_images/flower_photos.tgz 。

**步骤 2**&#160; &#160; 参考“上传业务数据”章节内容，将数据集上传至华为云OBS桶中（假设OBS桶路径为：“s3://automation/data”）。

该路径下包含了用户训练模型需要使用的所有图像文件， 该目录下有5个子目录，代表5种类别，分别为：daisy, dandelion, roses, sunflowers, tulips。每个子目录的文件夹名称即代表该分类的label信息，每个子目录下存放对应该目录的所有图像文件，则目录结构为：

    s3://automation/data/flower_photos
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

**步骤 3**  &#160; &#160; 参考“访问ModelArts”章节内容，登录“ModelArts”管理控制台，单击左侧导航栏的“开发环境”。

**步骤 4**&#160; &#160; 在“开发环境”界面，单击“Notebook”，点击左上角的“创建”，在弹出框中，输入开发环境名称、描述、镜像类型、实例规格、代码存储的OBS路径等参数，单击“立即创建”，完成创建操作。

**步骤 5**&#160; &#160; 在开发环境列表中，单击所创建开发环境右侧的“打开”，进入Jupyter Notebook文件目录界面。

**步骤 6**&#160; &#160; 单击右上角的“New”，选择“Python 2” ，进入代码开发界面。参见数据格式转换完整代码，在Cell中填写数据代码。


    from moxing.tensorflow.datasets.raw.raw_dataset import split_image_classification_dataset

    split_image_classification_dataset(
          split_spec={'train': 0.9, 'eval': 0.1},
          src_dir='s3://automation/data/flower_photos',
          dst_dir='s3://automation/data',
          overwrite=False)


**步骤 7**&#160; &#160; 单击Cell上方的运行按钮 ，运行代码。将数据集按9：1的比例划分为train和eval两部分，并输出到“s3://automation/data”，目录结果如下所示：

    s3://automation/data
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

**步骤 1**&#160; &#160; 返回“ModelArts”管理控制台界面。单击左侧导航栏的“训练作业”，进入“训练作业”界面。

**步骤 2**&#160; &#160; 在“算法/内置算法”列表中找到名称为“ResNet_v1\_50”的模型，其他参数确认无误后，单击“立即创建”完成训练作业创建。

"数据集"请选择训练集和验证集所在的父目录（在本案例中，即s3://automation/data）。

**步骤 3**&#160; &#160; 在模型训练的过程中或者完成后，通过创建TensorBoard作业查看一些参数的统计信息，如loss， accuracy等。

训练作业完成后，即完成了模型训练过程。如有问题，可点击作业名称，进入作业详情界面查看训练作业日志信息。

**步骤 4**&#160; &#160; 当训练作业运行成功后，可在“训练输出”下查看新的模型文件。


### 3. 部署模型

完成模型的训，用户可以使用该模型，通过创建预测作业，部署预测服务，操作步骤如下：

**步骤 1**&#160; &#160; 在“预测作业管理”界面，单击“创建预测作业”，进入“创建预测作业”界面。

**步骤 2**&#160; &#160; 完成参数配置。其中，“使用模型”为模型存放路径，请参考训练作业中运行参数“train_url”的值。

**步骤 3**&#160; &#160; 检查当前配置，确认无误后，单击“提交作业”，完成预测作业的创建。此时，可以在“预测作业管理”界面的作业列表中查看已创建的预测作业。

### 4. 发起预测请求
当预测作业的状态处于“运行中”，表示预测服务已部署。

ModelArts提供一个请求预测服务的客户端工程，用户可以使用该工程向预测服务发起预测请求，获取图片分类的结果，操作步骤如下：

**步骤 1**&#160; &#160; 执行下面命令，下载客户端代码。

    git clone https://github.com/huawei-clouds/dls-tfserving-client.git

**步骤 2**  &#160; &#160; 分别执行如下3条命令，发起预测请求。


    cd dls-tfserving-client/java

    mvn clean install

    java -jar target/predict-1.0.0.jar \
    image_classification \
    --host=10.154.74.249 \
    --port=30600 \
    --dataPath="xx/dls-tfserving-client/data/flowers/flower1.jpg" \
    --labelsFilePath="xx/dls-tfserving-client/data/flowers/labels.txt" \
    --modelName="resnet_v1_50"

**注意：**

**这里3个参数host、port、以及model_name的值即为预测作业的IP地址、端口号和模型名称3个参数值。参数dataPath和labelsFilePath表示待预测图片和类别标签的路径。**

**关于发起预测请求的详细操作指导，请参考：https://github.com/huawei-clouds/dls-tfserving-client 。**
