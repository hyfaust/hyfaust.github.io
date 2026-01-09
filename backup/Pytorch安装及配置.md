 #python/pytorch
## CPU版本torch（eg.动手深度学习）
```python
pip install torch==1.12.0  
pip install torchvision==0.13.0
pip install d2l==0.17.6
```

## PyTorch gpu版本安装及Notebook使用虚拟环境的方法
### 安装CUDA并检查
```python
nvidia-smi #查询信息，得知最高CUDA版本（实装12.8）
nvcc -V  #安装完成后检查信息
#可将DNN模块解压至C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.8\bin加速运算
```
### 虚拟环境配置及相关软件安装
```python
conda create --name gputorch python==3.11 -y #创建环境
pip install torch==2.7.1 torchvision torchaudio -f https://mirrors.aliyun.com/pytorch-wheels/cu128/
conda install jupyter notebook #安装notebook或者jupyterlab，conda或pip安装均可
#conda install jupyterlab
#安装完成后进入python检查torch和CUDA
python
import torch
print(torch.__version__)
print(torch.cuda.is_available())
print(torch.backends.cudnn.version())
```
### Base环境的Notebook使用其它虚拟环境的方法
```bash
conda install ipykernel #在对应环境下安装ipykernel,已安装notebook则无需再安装
conda deacitvate #退出至base环境
conda install nb_conda_kernels #内核配置管理
jupyter notebook #kernels很多的时候还是用VScode里的插件吧
```
### conda换源
```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```
## Jupyter notebook 和 lab 快捷方式属性
```powershell
E:\ProgramData\Miniconda3\python.exe E:\ProgramData\Miniconda3\cwp.py  E:\ProgramData\Miniconda3\envs\vml E:\ProgramData\Miniconda3\envs\vml\python.exe E:\ProgramData\Miniconda3\envs\vml\Scripts\jupyter-lab-script.py "F:\Documents\Visualize-ML"
#conda的cwp，环境中的python，jupyterlab的启动py文件，工作目录

E:\ProgramData\Miniconda3\python.exe E:\ProgramData\Miniconda3\cwp.py E:\ProgramData\Miniconda3\envs\d2l E:\ProgramData\Miniconda3\envs\d2l\python.exe E:\ProgramData\Miniconda3\envs\d2l\Scripts\jupyter-notebook-script.py "F:\Documents\d2l-zh-2.0.0"
#同上含义
```