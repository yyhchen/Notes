# 深度学习中，图片表示为模型输入的方法

---

在深度学习中，表示图像数据通常涉及几个关键步骤，包括读取图像、调整大小、归一化、转换为张量等。以下是一些常用的方法：
1. **读取图像**：
   - 使用OpenCV：`cv2.imread('image_path.jpg')`，这会读取图像并返回一个NumPy数组。
   - 使用PIL（Python Imaging Library）：`Image.open('image_path.jpg')`，这会返回一个PIL图像对象。
2. **调整图像大小**：
   - 使用OpenCV：`cv2.resize(image, (new_width, new_height))`，这会调整图像的大小，同时保持宽高比。
   - 使用PIL：`image.resize((new_width, new_height))`，这也会调整图像大小。
3. **转换数据类型**：
   - 将图像数据转换为`float32`，因为大多数深度学习模型使用32位浮点数来表示像素值。
4. **归一化**：
   - 将图像像素值归一化到[0, 1]范围。例如，`image = image / 255.0`。
5. **调整维度顺序**：
   - 将图像数组的维度顺序从`(height, width, channels)`（HWC）转换为`(channels, height, width)`（CHW），这是大多数深度学习框架（如PyTorch、TensorFlow）中图像张量的标准顺序。
6. **转换为张量**：
   - 将NumPy数组或PIL图像转换为张量。在PyTorch中，可以使用`torch.from_numpy(numpy_array)`或`image.to(torch.float32)`。
7. **数据增强**（可选）：
   - 在训练之前，可以对图像进行旋转、裁剪、颜色调整等操作，以增加模型的泛化能力。可以使用`torchvision.transforms`中的函数来实现。
8. **批处理**（可选）：
   - 将多个图像组成一个批次，这通常在训练时进行。可以使用`torch.utils.data.DataLoader`来加载和批处理数据。

<br>

以下是一个简单的示例，展示了如何使用PyTorch和OpenCV来表示图像数据：
```python
import cv2
import numpy as np
import torch
# 使用OpenCV读取图像 uint8
image = cv2.imread('image_path.jpg')
# 转换数据类型为float32
image = image.astype(np.float32)
# 归一化像素值到[0, 1]范围
image /= 255.0
# 调整维度顺序从HWC到CHW (256, 256, 3)--> (3, 256, 256)
image = np.transpose(image, (2, 0, 1))
# 将NumPy数组转换为PyTorch张量  torch.Size([3, 256, 256])
tensor = torch.from_numpy(image)
# 如果需要，可以将张量添加到批次中  torch.Size([1, 3, 256, 256])
batch_tensor = torch.unsqueeze(tensor, 0)  # 添加批次维度
```
这个示例展示了如何将一个图像读取为NumPy数组，进行归一化，调整维度顺序，然后转换为PyTorch张量。后面可以根据需要添加数据增强和其他预处理步骤。


<br>
<br>
<br>


## 在pytorch中的torchvision库中还提供了 transforms 库实现 image2tensor的方法

```python
from torchvision import datasets, transforms

transform = transforms.Compose([
    transforms.ToTensor(),      # 将PIL对象（即图片中的每一个像素）转换成 Tensor
    transforms.Normalize((0.1307, ), (0.3081, ))    # 均值 和 方差 (可以有多个, 此处是固定值，数据集已经给好了的)
])


# 训练集（这是torchvision 内置数据，可以直接调用，而不需要建立 Dataset 类）
train_datasets = datasets.MNIST(
    root='../dataset/mnist/',
    train=True,
    download=True,
    transform=transform     # 拿到数据就进行Tensor转换 和 标准化数据
)
train_loader = DataLoader(
    dataset=train_datasets,
    batch_size=batch_size,
    shuffle=True
)
```
