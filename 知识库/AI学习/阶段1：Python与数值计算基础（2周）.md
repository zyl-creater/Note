# 人工智能学习实施手册
## 阶段1：Python与数值计算基础（2周）
### 📚 学习目标
1. 掌握Python在数值计算场景下的高效写法
2. 构建可复用的数据处理工作流
3. 搭建Linux深度学习开发环境
### 🧠 C++程序员注意要点
- Python与C++联合编程技巧（使用pybind11）
- 理解Python的GIL机制对并行计算的影响
- 掌握numpy内存布局与C/C++数组的互操作
### 📝 每日任务清单
#### 第1-3天：Python核心突击
```
# 重点掌握的Python特性（对比C++）
with open('data.bin', 'rb') as f:
# RAII模式类似C++
data = pickle.load(f) @dataclass 
# 替代C++结构体
class ImageData: pixels: np.ndarray label: int
# 列表推导式（对比C++算法库）
squares = [x**2 for x in range(10) if x%2==0]
```
推荐资源：
- [Python与C++交互实战](https://pybind11.readthedocs.io/)
- 《流畅的Python》第4、5章

#### 第4-7天：Numpy高效计算
```
# 避免循环的矢量化计算
def cosine_similarity(vecs):
    norms = np.linalg.norm(vecs, axis=1) 
    return vecs @ vecs.T / np.outer(norms, norms)

# 内存共享机制
c_array = np.ctypeslib.as_array(
    (ctypes.c_double * 1000000).from_address(ptr)) 
```
实战项目：
- 实现矩阵乘法速度对比（Python循环 vs Numpy）
- 用memory_map处理超大型数据文件

#### 第8-14天：Linux深度学习环境
```
# GPU环境配置脚本示例
#!/bin/bash
conda create -n dl python=3.9 -y
conda activate dl
conda install pytorch torchvision cudatoolkit=11.3 -c pytorch

# 使用tmux进行实验管理
tmux new -s train_session
Ctrl+b d  # 断开会话
```
推荐工具链：
- [CUDA Linux安装指南](https://docs.nvidia.com/cuda/)
- [Linux性能监控手册](https://brendangregg.com/linuxperf.html)
