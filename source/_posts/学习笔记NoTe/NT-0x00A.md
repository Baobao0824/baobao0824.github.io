---
title: PyTorch for Deep Learning & Machine Learning – Full Course(5) - NoTe0x00A
tags:
  - 代码
  - 学习笔记NoTe
  - 人工智能
categories:
  - 学习笔记NoTe
date: 2026-06-24 11:11:23
---



> 原链接：
> https://www.youtube.com/watch?v=V_xro1bcAuA

## 写在前面

这一系列是我在学习他人的优质课程（博客文章、视频课程等）时所做的学习笔记，根据我自身的水平来进行学习，同时会进行思维的发散，补充原文中没有提到或者有错误的地方。同时也会进行经常性的更新和整理，让它和我的当下的状态更加契合。

接下来是视频教程的最后一个部分，即Cutsom Datasets，这个结束了之后我感觉我就对深度学习能有不错的基础了（但愿）。

## custom dataset of pizza, steak and sushi images

这一章的主要内容是将自己的自定义数据集导入pytorch并使用。

```py
import torch 
from torch import nn
torch.__version__
# '2.10.0'
device = "cuda" if torch.cuda.is_available() else "mps" if torch.mps.is_available() else "cpu"
device
# 'mps'
```

我们选择的数据集是Food101数据集的一个子集。原始Food101包括了101种不同的食物，每个类别有1000张图（750训练250测试），而我们在这里只选取其中3个食物类别，使用10%的图片，即75张训练，25张测试。

为什么要这样做呢？因为在开展机器学习项目时，最好先在小规模数据集上进行尝试，这样就可以加速实验迭代过程。

我们选择披萨、牛排和寿司，开始下载数据集，作者已经给了我们示例的链接了。

```py
import requests
import zipfile
from pathlib import Path
# 将路径设置到data文件夹
data_path = Path("data")
image_path = data_path/ "pizza_steak_sushi"
# 如果这个文件夹不存在，那么就创建文件夹
if image_path.is_dir():
    print("路径已存在，跳过下载")
else:
    print("图片路径不存在，正在创建")
    image_path.mkdir(parents=True, exist_ok=True)

with open(data_path / "pizza_steak_sushi.zip", "wb") as f:
    request = requests.get("https://github.com/mrdbourke/pytorch-deep-learning/raw/main/data/pizza_steak_sushi.zip")
    print("正在下载🍕🥩🍣")
    f.write(request.content)

# 解压文件
with zipfile.ZipFile(data_path / "pizza_steak_sushi.zip", "r") as zip_ref:
    zip_ref.extractall(image_path)
    print("解压文件完成")
# 图片路径不存在，正在创建
# 正在下载🍕🥩🍣
# 解压文件完成
```

解压之后的文件目录如下所示：

```text
pizza_steak_sushi
├── test
│   ├── pizza
│   │   ├── 1152100.jpg
│   │   ├── 1503858.jpg
│   │   └── ...
│   ├── steak
│   │   ├── 100274.jpg
│   │   ├── 1016217.jpg
│   │   └── ...
│   └── sushi
│       ├── 1172255.jpg
│       ├── 1230335.jpg
│       └── ...
└── train
    ├── pizza
    │   ├── 1008844.jpg
    │   ├── 1033251.jpg
    │   └── ...
    ├── steak
    │   ├── 100135.jpg
    │   ├── 1225762.jpg
    │   └── ...
    └── sushi
        ├── 1070104.jpg
        ├── 1129338.jpg
        └── ...
```

除了使用`tree`命令，我们也可以直接写一个函数来看一下这个文件夹的详细信息。这个视频的作者就是很细和用不同的方式来展示这些内容。

```py
import os
def walk_through_dir(dir_path):
    """遍历dir_path返回内容

    Args:
        dir_path (stirng): 对应的目录
    """
    for sub_dir_path, dir_list, file_list in os.walk(dir_path):
        print(f"目录{sub_dir_path}中有{len(dir_list)}个文件夹和{len(file_list)}个文件")
        pass
    pass

walk_through_dir(image_path)
# 目录data/pizza_steak_sushi中有2个文件夹和0个文件
# 目录data/pizza_steak_sushi/test中有3个文件夹和0个文件
# 目录data/pizza_steak_sushi/test/steak中有0个文件夹和19个文件
# 目录data/pizza_steak_sushi/test/sushi中有0个文件夹和31个文件
# 目录data/pizza_steak_sushi/test/pizza中有0个文件夹和25个文件
# 目录data/pizza_steak_sushi/train中有3个文件夹和0个文件
# 目录data/pizza_steak_sushi/train/steak中有0个文件夹和75个文件
# 目录data/pizza_steak_sushi/train/sushi中有0个文件夹和72个文件
# 目录data/pizza_steak_sushi/train/pizza中有0个文件夹和78个文件
```

接下来我们要获取图片的具体信息（元数据），步骤如下：

1. 获取所有图像的路径。
2. 使用`random.choice()`选取一个图像路径。
3. 通过`pathlib.Path.parent.stem`获取图像对应的类别名称（其实就是对应母文件夹的名称）。
4. 用PIL库打开图像，显示并打印元数据。

```py
import random
from PIL import Image
# 获取所有图片的路径
train_dir = image_path / "train"
test_dir = image_path / "test"
train_dir, test_dir
# (PosixPath('data/pizza_steak_sushi/train'),
#  PosixPath('data/pizza_steak_sushi/test'))
image_path_list = list(image_path.glob("*/*/*.jpg"))
# 随机选一张图片路径
random_image_path = random.choice(image_path_list)
# 获取图片对应的类别（父文件夹名称）
image_class = random_image_path.parent.stem
# 加载图片
random_image = Image.open(random_image_path)
random_image_path
# PosixPath('data/pizza_steak_sushi/train/steak/2880035.jpg')
image_class
# 'steak'
random_image.height, random_image.width
# (512, 512)
random_image
```

![这个绿不拉几的皮是什么玩意，怎么还剩了半瓶百事可乐](1.png)

或者选择把所有元数据弄成一张图片输出。这个array的三个信息分别是长宽和channel数，每个像素点有三个数RGB来表示。然后看一下这个array到底长什么样。

```py
import numpy as np
import matplotlib.pyplot as plt

# 把图片变成np.array
image_as_array = np.asarray(random_image)
# 用matplotlab展示
plt.figure(figsize=(10,7))
plt.imshow(image_as_array)
plt.title(f"type: {image_class}, array shape: {image_as_array.shape}")
plt.axis(False)
```

每一行都代表一个像素，分别是RGB的值。

```text
array([[[140, 155, 126],
        [141, 156, 127],
        [142, 156, 130],
        ...,
        [ 70,  62,  60],
        [ 65,  57,  55],
        [ 59,  51,  48]],

       [[133, 148, 117],
        [137, 152, 121],
        [143, 158, 129],
        ...,
        [ 65,  59,  59],
        [ 63,  54,  55],
        [ 59,  50,  51]],

       [[131, 146, 113],
        [127, 142, 109],
        [128, 143, 112],
        ...,
        [ 74,  68,  72],
        [ 73,  64,  69],
        [ 69,  60,  63]],

       ...,
...
        [191, 186, 190],
        ...,
        [  7,   6,  12],
        [  7,   6,  12],
        [  6,   5,  11]]], shape=(512, 512, 3), dtype=uint8)
```

![懒得换图片了还用这张](2.png)

## Turning images into tensors

在进行训练之前，我们要将图片数据转成tensor，然后封装为`torch.utils.data.Dataset`，进一步构建为`torch.utils.data.DataLoader`。

```py
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

# 给图片写对应的transform函数，大小改为64*64，同时随机水平翻转图像
data_transform = transforms.Compose([
    transforms.Resize(size=(64,64)),
    transforms.RandomHorizontalFlip(p=0.5),
    transforms.ToTensor()
])
data_transform(random_image).shape
# torch.Size([3, 64, 64])
```

写一个函数，直观展示对比一下图片处理前后到底是什么变化。

```py
def plot_transformed_images(image_paths: list[Path], transform:transforms.Compose, n=3, seed=None):
    """在图片列表中获取随机图片，转换之后给出对比图片

    Args:
        image_paths (list): 图片列表
        transform (transforms.Compose): 转换函数
        n (int, optional): 选取图片个数. Defaults to 3.
        seed (int | float, optional): 图片种子 Defaults to None.
    """
    if seed:
        random.seed(seed)
    random_image_paths = random.sample(image_paths, k=n)
    for path in random_image_paths:
        with Image.open(path) as f:
            fig, ax  = plt.subplots(nrows=1,ncols=2)
            ax[0].imshow(f)
            ax[0].set_title(f"Original \n size: {f.size}")
            ax[0].axis(False)
            # 变换之后再进行展示，注意我们会把原来在第0维的通道放在第2维
            transformed_image = transform(f).permute(1,2,0)
            ax[1].imshow(transformed_image)
            ax[1].set_title(f"Transformed \n shape: {transformed_image.shape}")
            ax[1].axis("off")
            fig.suptitle(f"class:{path.parent.stem}",fontsize=16)
            pass
        pass
    pass
pass

plot_transformed_images(
    image_paths=image_path_list,
    transform=data_transform,)
```

![好不容易找到一张三种食物各一个的](3.png)

接下来，我们有两种选择，要么是用`ImageFolder`，要么是用自定义`Dataset`。

### ImageFolder

首先我们来用`ImageFolder`，因为我们图片路径的组织方式符合`ImageFolder`要求。这个类在`torchvision.datasets`中，构造函数的参数有以下这些：

+ `root`，接收str或者Path，数据集根目录。
+ `transform`，对图像做的变换链，用来跑模型的话基本都要配一下。
+ `target_transform`，对标签（即整数index）做额外变换，一般不用。
+ `loader`，自定义图片加载函数，默认用`PIL.Image.open`。
+ `is_valid_file`，过滤函数，用来排除损坏的文件或者非图片文件。
+ `allow_empty`，是否允许某个类别文件夹为空，默认为False，此时空文件夹直接报错。

```py
from torchvision.datasets import ImageFolder
train_data_image_folder = ImageFolder(
    root=train_dir,
    transform=data_transform,
)
test_data_image_folder = ImageFolder(
    root=test_dir,
    transform=data_transform
)
train_data_image_folder, test_data_image_folder
# (Dataset ImageFolder
#      Number of datapoints: 225
#      Root location: data/pizza_steak_sushi/train
#      StandardTransform
#  Transform: Compose(
#                 Resize(size=(64, 64), interpolation=bilinear, max_size=None, antialias=True)
#                 RandomHorizontalFlip(p=0.5)
#                 ToTensor()
#             ),
#  Dataset ImageFolder
#      Number of datapoints: 75
#      Root location: data/pizza_steak_sushi/test
#      StandardTransform
#  Transform: Compose(
#                 Resize(size=(64, 64), interpolation=bilinear, max_size=None, antialias=True)
#                 RandomHorizontalFlip(p=0.5)
#                 ToTensor()
#             ))
```

接下来我们看一下这个数据集在这里是如何组织的。，我们可以很轻易的获取类别名列表和字典，以及通过下标获取图片的Tensor。

```py
# 类别名列表
class_names = train_data_image_folder.classes
class_names
# ['pizza', 'steak', 'sushi']
class_dict = train_data_image_folder.class_to_idx
class_dict
# {'pizza': 0, 'steak': 1, 'sushi': 2}
len(train_data_image_folder),len(test_data_image_folder)
# (225, 75)
train_data_image_folder.samples[0]
# ('data/pizza_steak_sushi/train/pizza/1008844.jpg', 0)
img, label = train_data_image_folder[0][0],train_data_image_folder[0][1]
img
# tensor([[[0.1137, 0.1020, 0.0980,  ..., 0.1255, 0.1216, 0.1176],
#          [0.1059, 0.0980, 0.0980,  ..., 0.1294, 0.1294, 0.1294],
#          [0.1020, 0.0980, 0.0941,  ..., 0.1333, 0.1333, 0.1333],
#          ...,
#          [0.1098, 0.1098, 0.1255,  ..., 0.1686, 0.1647, 0.1686],
#          [0.0863, 0.0941, 0.1098,  ..., 0.1686, 0.1647, 0.1686],
#          [0.0863, 0.0863, 0.0980,  ..., 0.1686, 0.1647, 0.1647]],

#         [[0.0745, 0.0706, 0.0745,  ..., 0.0588, 0.0588, 0.0588],
#          [0.0706, 0.0706, 0.0745,  ..., 0.0627, 0.0627, 0.0627],
#          [0.0706, 0.0745, 0.0745,  ..., 0.0706, 0.0706, 0.0706],
#          ...,
#          [0.1255, 0.1333, 0.1373,  ..., 0.2510, 0.2392, 0.2392],
#          [0.1098, 0.1176, 0.1255,  ..., 0.2510, 0.2392, 0.2314],
#          [0.1020, 0.1059, 0.1137,  ..., 0.2431, 0.2353, 0.2275]],

#         [[0.0941, 0.0902, 0.0902,  ..., 0.0196, 0.0196, 0.0196],
#          [0.0902, 0.0863, 0.0902,  ..., 0.0196, 0.0157, 0.0196],
#          [0.0902, 0.0902, 0.0902,  ..., 0.0157, 0.0157, 0.0196],
#          ...,
#          [0.1294, 0.1333, 0.1490,  ..., 0.1961, 0.1882, 0.1804],
#          [0.1098, 0.1137, 0.1255,  ..., 0.1922, 0.1843, 0.1804],
#          [0.1059, 0.1020, 0.1059,  ..., 0.1843, 0.1804, 0.1765]]])
img.shape
# torch.Size([3, 64, 64])
img.dtype
# torch.float32
label
# 0
type(label)
# int
# 如果要将图片展示的话，就需要把维度换一下。
img_permute = img.permute(1,2,0)
img_permute.shape
# torch.Size([64, 64, 3])
```

在这之后，我们将整理好的`ImageFolder`转换为`DataLoader`，让其成为可迭代对象，并且可以自定义批次大小。

```py
from torch.utils.data import DataLoader
BATCH_SIZE =1
train_dataloader_image_folder = DataLoader(
    dataset=train_data_image_folder,
    batch_size=BATCH_SIZE,
    num_workers=1,
    shuffle=True
)
test_dataloader_image_folder = DataLoader(
    dataset=test_data_image_folder,
    num_workers=1,
    batch_size=BATCH_SIZE,
)
train_dataloader_image_folder, test_dataloader_image_folder
# (<torch.utils.data.dataloader.DataLoader at 0x3055c7b90>,
#  <torch.utils.data.dataloader.DataLoader at 0x3056757c0>)
len(train_dataloader_image_folder), len(test_dataloader_image_folder)
# (225, 75)
img, label = next(iter(train_dataloader_image_folder))
img.shape, label.shape
# (torch.Size([1, 3, 64, 64]), torch.Size([1]))
```

### 自定义Dataset

如果我们使用自定义dataset，也能实现：从文件中直接加载图像、从数据集中获取类别名称，以字典形式从数据集中获取所有类别。使用它之后，几乎可以将任何数据源封装为dataset，不受pytorch预置dataset功能的限制。不过问题就在于，即使你能转成dataset，但是不代表他能够正常工作，同时也需要编写更多代码（因为要集成`Dataset`类），可能会出现错误。

首先，我们需要一个辅助函数，用来获取类别名称。使用`os.scandir`遍历目标目录以获取类别名称，如果未找到类别名称就抛出异常（一般是意味着目录结构存在问题），然后将类别名称分别转换为字典和列表格式并返回。

```py
from typing import Tuple, Dict, List

def find_classes(direcotry:str) -> Tuple[List[str], Dict[str, int]]:
    classes = sorted(entry.name for entry in os.scandir(direcotry) if entry.is_dir())
    # 当出现问题的时候抛出异常
    if not classes:
        raise FileNotFoundError(f"在{direcotry}中找不到任何类，请检查文件结构")
    # 创建一个label的字典
    class_to_idx = {
        class_name: i for i, class_name in enumerate(classes)
    }
    return classes,class_to_idx

find_classes(train_dir)
# (['pizza', 'steak', 'sushi'], {'pizza': 0, 'steak': 1, 'sushi': 2})
```

为了复现和`ImageFolder`相同的功能，我们需要定义下面的属性和方法。

+ `paths`，存储所有图像的路径。
+ `transform`，要应用的数据变换操作。
+ `classes`，目标类别的名称列表。
+ `class_to_idx`，将目标类别映射到整数标签的字典。
+ `__init__`，设置数据源目录，传入`transform`作为可选项。
+ `load_images()`方法，用于加载并打开图像。
+ `__len__()`，重写该方法，让其返回数据集的总样本数量。
+ `__getitem__()`，重写该方法，让其能够根据传入的索引返回一个对应的样本，即图像和标签。

```py
from torch.utils.data import Dataset
import pathlib

class ImageDataset(Dataset):
    def __init__(self, target_dir: str | Path, transform = None):
        self.paths = list(pathlib.Path(target_dir).glob("*/*.jpg"))
        self.transform = transform
        self.classes, self.class_to_idx = find_classes(str(target_dir))
    
    def load_image(self, index:int) -> Image.Image:
        image_path = self.paths[index]
        return Image.open(image_path)
    
    def __len__(self) -> int:
        return len(self.paths)
    
    def __getitem__(self, index:int) -> Tuple[torch.Tensor, int]:
        img = self.load_image(index)
        class_name = self.paths[index].parent.name
        class_idx = self.class_to_idx[class_name]
        if self.transform:
            return self.transform(img), class_idx
        else:
            return img, class_idx
```

```py
train_data_dataset = ImageDataset(
    target_dir=train_dir,
    transform=data_transform
)
test_data_dataset = ImageDataset(
    target_dir= test_dir,
    transform= data_transform
)
train_data_dataset,test_data_dataset
# (<__main__.ImageDataset at 0x305b8bf20>,
#  <__main__.ImageDataset at 0x305b885f0>)
len(train_data_dataset),len(test_data_dataset)
# (225, 75)
train_data_dataset.classes
# ['pizza', 'steak', 'sushi']
train_data_dataset.class_to_idx
# {'pizza': 0, 'steak': 1, 'sushi': 2}
```

接下来，我们需要为这个`dataset`编写展示随机图像的函数，输入一个dataset，输出随机的若干张图片（最大10张）。

```py
def display_random_images_dataset(
        dataset: torch.utils.data.Dataset,
        classes: List[str] = None,
        n: int =10,
        display_shape: bool = True,
        seed: int = None
):
    if n>10:
        n=10
        display_shape=False
        print("为了便于展示，n不应该大于10，因此此时移除形状特征显示")
    if seed:
        random.seed(seed)
    random_samples_idx = random.sample(range(len(dataset)),k=n)
    plt.figure(figsize=(16,8))
    for i, target_sample in enumerate(random_samples_idx):
        target_image, target_label = dataset[target_sample][0], dataset[target_sample][1]
        target_image_adjust = target_image.permute(1,2,0)
        plt.subplot(1,n,i+1)
        plt.imshow(target_image_adjust)
        plt.axis("off")
        if classes:
            title = f"class name: {classes[target_label]}"
            if display_shape:
                title += f"\n shape: {target_image_adjust.shape}"
        plt.title(title)
```

然后我们直接放三张举例，看看图片长什么样。

```py
display_random_images_dataset(
    train_data_dataset,
    classes=class_names,
    n=3
)
```

![我们又再次实现了ImageFolder的功能](4.png)

接下来，我们为自定义`dataset`创造对应的`dataloader`，定义已经在上面弄完了，直接声明变量就可以了。

```py
train_dataloader_dataset = DataLoader(
    dataset= train_data_dataset,
    batch_size= BATCH_SIZE,
    shuffle=True
)
test_dataloader_dataset = DataLoader(
    dataset= test_data_dataset,
    batch_size=BATCH_SIZE,
    shuffle=True
)
train_dataloader_dataset, test_dataloader_dataset
# (<torch.utils.data.dataloader.DataLoader at 0x3055708f0>,
#  <torch.utils.data.dataloader.DataLoader at 0x305d308f0>)
```

## Creating a custom dataset class (overview)

接下来我们进行数据增强。数据增强时值通过人工手段为训练数据引入多样性的过程，对于图像数据来说，这通常意味着对训练图像施加各种图像变换操作。刚才我们那个给图片弄成64乘64并随机翻转，就是一种数据增强。接下来我们尝试另一种数据增强，也就是`transform.TrivialAugmentWide`，他是官方自带的随机增强，会在14变换中随机选择一种，通过`num_magnitude`参数来控制强弱。 

```py
train_transform = transforms.Compose([
    transforms.Resize(size=(224,224)),
    transforms.TrivialAugmentWide(num_magnitude_bins=31),
    transforms.ToTensor()
])
test_transform = transforms.Compose([
    transforms.Resize(size=(224,224)),
    transforms.ToTensor()
])
```

我们用这种新的图像增强来图示进行对比，看看torch是如何进行随机变换的。

```py
plot_transformed_images(
    image_paths=image_path_list,
    transform=train_transform,
    n=3,
    seed=None
)
```

![这张图片可以明显的看出系统对图像做了变换](5.png)

## Model 0: TinyVGG without data augmentation

我们首先复现CNN Explainer网站中的TinyVGG架构。它只有2-3个卷积块，通道和参数都很小。

```py
# 定义model_0所需要的数据
model_0_transform = transforms.Compose([
    transforms.Resize(size=(64,64)),
    transforms.ToTensor()
])
model_0_train_data = datasets.ImageFolder(
    root=train_dir,
    transform=model_0_transform
)
model_0_test_data = datasets.ImageFolder(
    root = test_dir,
    transform= model_0_transform
)
BATCH_SIZE = 32
model_0_train_dataloader = DataLoader(
    dataset=model_0_train_data,
    batch_size=BATCH_SIZE,
    shuffle=True
)
model_0_test_dataloader = DataLoader(
    dataset=model_0_test_data,
    batch_size= BATCH_SIZE,
    shuffle= False
)
```

接下来是模型定义，一共有两个conv block。

```py
class Model0(nn.Module):
    def __init__(
            self,
            input_shape:int,
            hidden_units:int,
            output_shape:int
    )->None:
        super().__init__()
        self.conv_block_1 = nn.Sequential(
            nn.Conv2d(
                in_channels=input_shape,
                out_channels=hidden_units,
                kernel_size=3,
                stride=1,
                padding=0
            ),
            nn.ReLU(),
            nn.Conv2d(
                in_channels=hidden_units,
                out_channels=hidden_units,
                kernel_size=3,
                stride=1,
                padding=0
            ),
            nn.ReLU(),
            nn.MaxPool2d(kernel_size=2,stride=2)
        )
        self.conv_block_2 = nn.Sequential(
            nn.Conv2d(
                in_channels=hidden_units,
                out_channels=hidden_units,
                kernel_size=3,
                stride=1,
                padding=0
            ),
            nn.ReLU(),
            nn.Conv2d(
                in_channels=hidden_units,
                out_channels=hidden_units,
                kernel_size=3,
                stride=1,
                padding=0
            ),
            nn.ReLU(),
            nn.MaxPool2d(kernel_size=2, stride=2)
        )
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(in_features=hidden_units*13*13,out_features=output_shape)
        )
    
    def forward(self, x):
        x = self.conv_block_1(x)
        x = self.conv_block_2(x)
        x = self.classifier(x)
        return x
```

```py
# 这里的input_shape其实是图片的通道数量3
torch.manual_seed(42)
model_0 = Model0(
    input_shape=3,
    hidden_units=10,
    output_shape=len(class_names)
).to(device)
model_0
# Model0(
#   (conv_block_1): Sequential(
#     (0): Conv2d(3, 10, kernel_size=(3, 3), stride=(1, 1))
#     (1): ReLU()
#     (2): Conv2d(10, 10, kernel_size=(3, 3), stride=(1, 1))
#     (3): ReLU()
#     (4): MaxPool2d(kernel_size=2, stride=2, padding=0, dilation=1, ceil_mode=False)
#   )
#   (conv_block_2): Sequential(
#     (0): Conv2d(10, 10, kernel_size=(3, 3), stride=(1, 1))
#     (1): ReLU()
#     (2): Conv2d(10, 10, kernel_size=(3, 3), stride=(1, 1))
#     (3): ReLU()
#     (4): MaxPool2d(kernel_size=2, stride=2, padding=0, dilation=1, ceil_mode=False)
#   )
#   (classifier): Sequential(
#     (0): Flatten(start_dim=1, end_dim=-1)
#     (1): Linear(in_features=1690, out_features=3, bias=True)
#   )
# )
```

接下来我们来测试模型是否正常，在单个图片上模拟前向传播。

```py
image_batch, label_batch = next(iter(model_0_train_dataloader))
image_batch.shape, label_batch.shape
# (torch.Size([32, 3, 64, 64]), torch.Size([32]))
model_0(image_batch.to(device))
# tensor([[ 2.0789e-02, -1.9351e-03,  9.5317e-03],
#         [ 1.8427e-02,  2.4670e-03,  6.6757e-03],
#         [ 1.7699e-02,  1.0262e-03,  9.4657e-03],
#         [ 2.4441e-02, -3.3526e-03,  9.6011e-03],
#         [ 1.9930e-02,  6.6316e-04,  1.0779e-02],
#         [ 2.1281e-02,  2.0434e-03,  5.0047e-03],
#         [ 2.0999e-02,  1.2870e-04,  1.2473e-02],
#         [ 2.1577e-02, -1.9507e-03,  9.6941e-03],
#         [ 2.4504e-02, -4.7745e-03,  8.5280e-03],
#         [ 2.0252e-02, -4.7293e-04,  1.0908e-02],
#         [ 2.2215e-02, -4.1837e-04,  9.8123e-03],
#         [ 2.2313e-02, -2.1622e-03,  9.4455e-03],
#         [ 2.1841e-02, -3.7132e-03,  8.3783e-03],
#         [ 2.2863e-02, -1.7723e-03,  1.0287e-02],
#         [ 2.1647e-02, -4.4139e-03,  9.5022e-03],
#         [ 2.2096e-02, -4.1426e-03,  9.3853e-03],
#         [ 2.1209e-02, -4.4219e-03,  1.1475e-02],
#         [ 2.1711e-02, -2.7656e-03,  8.5006e-03],
#         [ 1.9951e-02,  2.8270e-05,  8.4380e-03],
#         [ 1.8298e-02,  1.6306e-03,  8.5499e-03],
#         [ 2.0768e-02,  1.7942e-03,  7.9412e-03],
#         [ 1.9834e-02, -3.9071e-03,  9.8740e-03],
#         [ 2.0893e-02,  1.3043e-04,  8.4190e-03],
#         [ 2.3202e-02, -3.4328e-03,  9.4937e-03],
#         [ 2.0501e-02, -2.5545e-03,  8.4874e-03],
# ...
#         [ 1.8347e-02, -1.5215e-03,  9.4640e-03],
#         [ 1.9868e-02, -2.3248e-03,  9.0062e-03],
#         [ 2.2065e-02, -4.6434e-03,  1.2666e-02],
#         [ 1.8059e-02,  2.6858e-03,  5.7898e-03]], device='mps:0',
#        grad_fn=<LinearBackward0>)
```

这个东西看起来实在是太费劲了。有没有什么其他的可视化方法呢？这里介绍一下`torchinfo`，他能够直观地向你展示模型每一层输出的形状是什么样的、有多少参数等，这样你就不用直接盯着print出来的信息了。

```py
from torchinfo import summary
summary(model_0, input_size=[1,3,64,64])
```

整个模型输出的结果如下所示。

```text
==========================================================================================
Layer (type:depth-idx)                   Output Shape              Param #
==========================================================================================
Model0                                   [1, 3]                    --
├─Sequential: 1-1                        [1, 10, 30, 30]           --
│    └─Conv2d: 2-1                       [1, 10, 62, 62]           280
│    └─ReLU: 2-2                         [1, 10, 62, 62]           --
│    └─Conv2d: 2-3                       [1, 10, 60, 60]           910
│    └─ReLU: 2-4                         [1, 10, 60, 60]           --
│    └─MaxPool2d: 2-5                    [1, 10, 30, 30]           --
├─Sequential: 1-2                        [1, 10, 13, 13]           --
│    └─Conv2d: 2-6                       [1, 10, 28, 28]           910
│    └─ReLU: 2-7                         [1, 10, 28, 28]           --
│    └─Conv2d: 2-8                       [1, 10, 26, 26]           910
│    └─ReLU: 2-9                         [1, 10, 26, 26]           --
│    └─MaxPool2d: 2-10                   [1, 10, 13, 13]           --
├─Sequential: 1-3                        [1, 3]                    --
│    └─Flatten: 2-11                     [1, 1690]                 --
│    └─Linear: 2-12                      [1, 3]                    5,073
==========================================================================================
Total params: 8,083
Trainable params: 8,083
Non-trainable params: 0
Total mult-adds (Units.MEGABYTES): 5.69
==========================================================================================
Input size (MB): 0.05
Forward/backward pass size (MB): 0.71
Params size (MB): 0.03
Estimated Total Size (MB): 0.79
==========================================================================================
```

我们可以清楚的看到模型的每一层到底都做了什么，同时能看到最终输出的结果。接下来，我们为`model_0`准备训练和测试用的函数，分别命名为`train_step`和`test_step`。

```py
def train_step(
        model: nn.Module,
        dataloader: torch.utils.data.DataLoader,
        loss_fn: nn.Module,
        optimizer: torch.optim.Optimizer,
        device = device
):
    model.train()
    train_loss, train_acc =0,0
    for batch, (X,y) in enumerate(dataloader):
        # 先把数据发送到对应的设备中
        X,y = X.to(device), y.to(device)
        y_pred = model(X)
        loss = loss_fn(y_pred,y)
        train_loss += loss.item()
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        y_pred_class = torch.argmax(torch.softmax(y_pred, dim=1), dim=1)
        train_acc +=(y_pred_class==y).sum().item()/len(y_pred)
    train_loss /= len(dataloader)
    train_acc /= len(dataloader)
    return train_loss, train_acc
```

```py
def test_step(
        model: nn.Module,
        dataloader: torch.utils.data.DataLoader,
        loss_fn: nn.Module,
        device=device
):
    model.eval()
    test_loss, test_acc =0,0
    with torch.inference_mode():
        for batch, (X,y) in enumerate(dataloader):
            X,y = X.to(device), y.to(device)
            test_pred_logits = model(X)
            loss = loss_fn(test_pred_logits,y)
            test_loss += loss.item()
            test_pred_labels = test_pred_logits.argmax(dim=1)
            # 这个代码写的也太烂了我说实话
            test_acc += ((test_pred_labels==y).sum().item()/len(test_pred_labels))
    test_loss /= len(dataloader)
    test_acc /=len(dataloader)
    return test_loss, test_acc
```

每一个`train_step`和`test_step`函数都是一个epoch的流程，然后我们再利用它来实现完整的`train`函数。

```py
from tqdm import tqdm

def train(
        model:nn.Module,
        train_dataloader,
        test_dataloader,
        optimizer,
        loss_fn: nn.Module = nn.CrossEntropyLoss(),
        epochs:int =5,
        device = device
):
    results={
        "train_loss": [],
        "train_acc": [],
        "test_loss": [],
        "test_acc": []
    }
    # 根据epoch数量来进行每一个步骤
    for epoch in tqdm(range(epochs)):
        train_loss, train_acc = train_step(
            model= model,
            dataloader=train_dataloader,
            loss_fn=loss_fn,
            optimizer=optimizer,
            device=device
        )
        test_loss, test_acc = test_step(
            model=model,
            dataloader=test_dataloader,
            loss_fn=loss_fn,
            device=device
        )
        # 看一下这个epoch发生了什么事情
        print(f"Epoch{epoch}, train loss: {train_loss:.2f}, train acc: {train_acc:.2f}, test loss: {test_loss:.2f}, test acc: {test_acc:.2f}")
        results["train_loss"].append(train_loss)
        results["train_acc"].append(train_acc)
        results["test_loss"].append(test_loss)
        results["test_acc"].append(test_acc)
    return results
```

接下来我们正式的对`model_0`开始训练和评估。

```py
torch.manual_seed(42)
torch.mps.manual_seed(42)
EPOCH_NUM = 20
model_0 =Model0(
    input_shape=3,
    hidden_units=10,
    output_shape=len(model_0_train_data.classes)
).to(device)
loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(
    params=model_0.parameters(),
    lr=0.001)
from timeit import default_timer
start_time = default_timer()
# 开始训练model_0
model_0_results = train(
    model=model_0,
    train_dataloader=model_0_train_dataloader,
    test_dataloader=model_0_test_dataloader,
    optimizer=optimizer,
    loss_fn=loss_fn,
    epochs= EPOCH_NUM
)
end_time = default_timer()
print(f"training time: {end_time-start_time:.0f} seconds")
# ...
# Epoch19, train loss: 0.78, train acc: 0.65, test loss: 1.03, test acc: 0.44
# training time: 14 seconds
```

最后的准确率也只有44%左右，还是差得很远。我们尝试画出`model_0`的损失率变化图表。

```py
def plot_loss_curves(results: Dict[str, List[float]]):
    loss = results["train_loss"]
    test_loss = results["test_loss"]
    accuracy = results["train_acc"]
    test_accurancy = results["test_acc"]
    epochs = range(len(results["train_loss"]))

    plt.figure(figsize=(15,7))

    plt.subplot(1,2,1)
    plt.plot(epochs,loss,label="train_loss")
    plt.plot(epochs, test_loss, label="test_loss")
    plt.title("Loss")
    plt.xlabel("Epochs")
    plt.legend()

    plt.subplot(1,2,2)
    plt.plot(epochs, accuracy, label= "train_accurancy")
    plt.plot(epochs, test_accurancy, label= "test_accurancy")
    plt.title("Accurancy")
    plt.xlabel("Epochs")
    plt.legend()

plot_loss_curves(model_0_results)
```

![左侧是损失率，右侧是准确率](6.png)

那么问题来了，什么样的损失曲线才是理想的呢？[谷歌开发者的这篇文章](https://developers.google.com/machine-learning/testing-debugging/metrics/interpretic)给我们列举了一些不同情况损失曲线下的调整策略。

+ 震荡的损失曲线，此时可以移除不良示例、缩减训练集为少量可信示例（排查到底是哪些不良示例导致曲线波动）、以及降低学习率。
+ 损失曲线急剧增加，一般有以下两种原因，首先是输入数据包括NaN，其次可能是输入数据包含大量离群值（可能是批处理的随机排序不当等）。
+ 测试损失与训练损失不一致，通常是测试损失大幅增加，可能是模型对训练集过拟合，需要减少特征数量、提高正则化率等。
+ 损失曲线在特定步数后出现混乱，那么可能是因为训练集没有进行充分洗牌。

观察曲线，我们会发现，我们的损失曲线既震荡，又出现了测试比训练高的情况，那么就说明模型对训练集过拟合、学习率太高。

## Model 1: TinyVGG with Data Augmentation

接下来我们来为模型添加数据增强，将其命名为`model_1`。首先添加对应的`dataLoader`等必要变量。

```py
model_1_transform_train = transforms.Compose([
    transforms.Resize(size=(64,64)),
    transforms.TrivialAugmentWide(num_magnitude_bins=31),
    transforms.ToTensor()
])
model_1_transform_test = transforms.Compose([
    transforms.Resize(size=(64,64)),
    transforms.ToTensor()
])
model_1_train_data = datasets.ImageFolder(
    root=train_dir,
    transform=model_1_transform_train
)
model_1_test_data = datasets.ImageFolder(
    root=test_dir,
    transform= model_1_transform_test
)
torch.manual_seed(42)
model_1_dataloader_train = DataLoader(
    dataset=model_1_train_data,
    batch_size=BATCH_SIZE,
    shuffle=True,
)
model_1_dataloader_test = DataLoader(
    dataset=model_1_test_data,
    batch_size=BATCH_SIZE
)
```

实际上，`model_1`和`model_0`的模型结构是一模一样的，只是对应的数据不同。

```py
model_1_transform_train = transforms.Compose([
    transforms.Resize(size=(64,64)),
    transforms.TrivialAugmentWide(num_magnitude_bins=31),
    transforms.ToTensor()
])
model_1_transform_test = transforms.Compose([
    transforms.Resize(size=(64,64)),
    transforms.ToTensor()
])
model_1_train_data = datasets.ImageFolder(
    root=train_dir,
    transform=model_1_transform_train
)
model_1_test_data = datasets.ImageFolder(
    root=test_dir,
    transform= model_1_transform_test
)
torch.manual_seed(42)
model_1_dataloader_train = DataLoader(
    dataset=model_1_train_data,
    batch_size=BATCH_SIZE,
    shuffle=True,
)
model_1_dataloader_test = DataLoader(
    dataset=model_1_test_data,
    batch_size=BATCH_SIZE
)
model_1 = Model0(
    input_shape=3,
    hidden_units=10,
    output_shape=len(model_1_train_data.classes)
).to(device)
```

我们继续给`model_1`同样的20个epoch，看一下结果是什么样的。
```py
torch.manual_seed(42)
torch.mps.manual_seed(42)
loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(params=model_1.parameters())
start_time = default_timer()
model_1_results = train(
    model=model_1,
    train_dataloader= model_1_dataloader_train,
    test_dataloader=model_1_dataloader_test,
    optimizer=optimizer,
    epochs=EPOCH_NUM,
    device = device

)
end_time = default_timer()
print(f"training time: {end_time-start_time:.0f} seconds")
# Epoch19, train loss: 0.81, train acc: 0.58, test loss: 1.06, test acc: 0.50
# training time: 13 seconds
plot_loss_curves(model_1_results)
```

![其实这个问题我感觉还是没有解决，原文训练的epoch数量实在是太少了](7.png)

## Compare model results

接下来，我们详细对比一下两个模型的结果。我们之前的结果对比都是通过硬编码。除此之外还有其他几种方法，我们接下来分别进行介绍。

我们首先使用TensorBoard，它本质上是一个读日志的看板。当我们用`pip`安装好`tensorboard`之后，只需要运行命令`tsneorboard --logdir=runs`来启动，然后点开链接`localhost:6006`即可。同时在代码中，我们也需要让torch生成对应的日志，也就是`torch.utils.tensorboard.SummaryWriter`。我们创建一个`train_with_writer`的函数。

```py
from torch.utils.tensorboard import SummaryWriter

writer = SummaryWriter(log_dir="runs/model_1")
def train_with_writer(
        model: nn.Module,
        train_dataloader,
        test_dataloader,
        optimizer,
        loss_fn: nn.Module = nn.CrossEntropyLoss(),
        epochs: int =5,
        device = device
):
    for epoch in tqdm(range(epochs)):
        train_loss, train_acc = train_step(
            model=model,
            dataloader=train_dataloader,
            loss_fn=loss_fn,
            optimizer= optimizer,
            device=device
        )
        writer.add_scalar(
            "Train/Loss",train_loss,epoch
        )
        writer.add_scalar(
            "Train/Acc", train_acc, epoch
        )
        test_loss, test_acc = test_step(
            model=model,
            dataloader=test_dataloader,
            loss_fn=loss_fn,
            device=device
        )
        writer.add_scalar(
            "Test/Loss",test_loss,epoch
        )
        writer.add_scalar(
            "Test/Acc",test_acc,epoch
        )
        pass

torch.manual_seed(42)
torch.mps.manual_seed(42)
loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(params=model_1.parameters())
model_1 = Model0(
    input_shape=3,
    hidden_units=10,
    output_shape=len(model_1_train_data.classes)
).to(device)
train_with_writer(
    model=model_1,
    train_dataloader=model_1_dataloader_train,
    test_dataloader=model_1_dataloader_test,
    loss_fn=loss_fn,
    optimizer=optimizer
    )
writer.close()
```

然后点开链接，就能看到可视化图表了。

![说实话我也没看懂这个图表](8.png)

## 总结

在这一篇教程中，我们体验了自己下载和简单整理数据集，使用真正的数据集来进行模型的训练。在这之后就可以进一步的使用transformer之类的其他模型了，对深度学习的主要流程有了更加深刻的理解。