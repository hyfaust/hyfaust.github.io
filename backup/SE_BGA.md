#SE
## BGA-1
采用六边形棱柱作初始几何构型，固定约束上下两面及边线，恒定体积设置为圆柱的1.3倍
## BGA-2
为上下两个固定面添加no_refine属性以加快计算
仅考虑表面张力的情况下，恒定体积的最终形状与张力的数值无关
## BGA-3
同时考虑表面张力和重力作用，故上下面应设置tension 0
> [! Uncertain]
> 仅考虑表面张力时，顶面和底面可以不设置；当同时考虑重力时，必须将其设置为0
> 严格意义上应仅在壁面，气液相界面处接触设置非零的表面张力

 采用cgs单位系统
## **BGA-4** 
base3 在3的基础上计算焊盘上的垂直力
一共有7种方法，推荐linear move方法
```c
zforce: 24.194068467172272  (single difference, top pad only)
zforce: 24.197468840488526  (central difference, top pad only)
zforce: 24.583191529832973  (single difference, linear move)
zforce: 24.583596719788638  (central difference, linear move)
  1. area: 0.154763478044165 energy:  47.0582870000139
  2. area: 0.154763478041577 energy:  47.0582870000139
  3. area: 0.154763708748211 energy:  47.0583460006838
  4. area: 0.154763708753397 energy:  47.0583460006838
zforce: 24.583612461270828  (central difference, linear move, reconverge)
  5. area: 0.154763593385103 energy:  47.0583164931673
  6. area: 0.154763593382516 energy:  47.0583164931673
zforce: 17.323186353177952  (geometric)
zforce: 24.583607830407118  (variational integrands)
```
chartdata可输出不同高度和体积约束下的结果表，[数据分析](https://www.qianwen.com/share?shareId=909f4285-55f3-4bc1-ae58-8f4bf272df1e)
在 `facet_general_integral` 的上下文中，Evolver 提供以下内置变量：

| 变量            | 含义                          |
| ------------- | --------------------------- |
| `x1, x2, x3`  | 面上某点的坐标 (x, y, z)           |
| `x4, x5, x6`  | **面法向向量分量**，即 n=∂u∂r​×∂v∂r​ |
| `x7, x8, x9`  | 第一偏导 ∂r/∂u                  |
| `x10,x11,x12` | 第二偏导 ∂r/∂v                  |
```bash
method_instance area_change method facet_general_integral 
scalar_integrand: ...
```
定义了一个名为area_change的标量积分方法，被积函数由scalar_integrand给出，[被积函数的推导](https://www.qianwen.com/share?shareId=b45175c2-3bfa-4d12-9ce3-38e71cbb17cd)
[三部分的推导](https://www.qianwen.com/share?shareId=3f8684c5-a4df-4ead-9fc8-7b1327d51b50)

## BGA-5
base3  with vertical load and optimizing pad height上部焊盘可移动，有一外加载荷pad_weight。将高度设置为了优化参量optimizing_parameter
设置了一虚拟点idx=13用于加载载荷
```
13 0 0 height bare
```
## BGA-6
base5 with upper and lower barriers 在3的基础上设定了两个barries，即将钎料限制在中间
~~运行时存在一定问题。重力设置为200可行，两百以上存在收敛性问题，原因未知，250虽然也存在收敛问题，但尚可求出合理解。~~
```shell
constraint_tolerance 1e-5
scale_limit 0.1/S_TENSION
```
设置最大步长，放宽收敛容差，最终可以很好求解
## BGA-7
base3  with lateral movement of upper pad. 上侧焊盘还有横向位移，大小由offset指定
但不能保证网格点变化均匀，通过约束实现横向位移
```bash
constraint 4
formula: (x-offset)^2 + y^2 = radius^2
```
## BGA-8
base7 converted to boundary on top in anticipation of optimizing lateral position and tilting.
通过边界实现横向位移，使网格点间距均匀
```shell
// upper pad rim boundary 1 parameters 1 
x1: offset + radius*cos(p1) 
x2: radius*sin(p1) 
x3: height 

// upper pad center point boundary 2 parameter 1 
x1: offset 
x2: 0 
x3: height
```
## BGA-9
base8 with z and offset as optimizing parameters. And vertical and horizontal loads.
类似bga-5，将横向位移当作优化参数，由横向载荷控制。无侧向载荷时，会优化回0.
## BGA-10
base9 with 3D motion and tilting one way. With forces and torque.
引入上焊盘的倾斜角度，有x,y两方向的位移。可计算力和扭矩
## BGA-11
base10 with optimizing parameters.
将倾斜角度、xy方向位移均作为优化变量
```shell
// loads on upper pad, dynes
parameter pad_weight = 200 // 垂直向下力（dyne） 
parameter x_load = 0.0 // x 方向剪切力 
parameter y_load = 0.0 // y 方向剪切力 
parameter cg_z = .02 // 重心高出 pad 的距离（cm）

// upper pad energy
quantity pad_energy energy method vertex_scalar_integral
scalar_integrand: pad_weight*z - x_load*x - y_load*y
```
## BGA-12
 base11 with boundary integrals to replace top and bottom
 替代上下焊盘，通过边界积分实现体积贡献，[具体推导](https://www.qianwen.com/share?shareId=ae660579-59b2-42f4-b8b8-ffcf4c623145)
~~运行时存在问题~~，xz应为正号，修改后正确。
## BGA-30
String model equivalent of bga-1.fe, taking advantage of circular symmetry.
根据径向对称性简化问题，仅考虑边界线框
## BGA-31
base30 with specified contact angle on pads.
在30的基础上考虑接触角
~~运行时存在问题，计算不收敛~~，原输入文件中步长设置过大，将scale_limit改为0.1/S_TENSION时结果正常收敛
参数：表面张力、密度
高度、半径+接触角

> [Ball Grid Array Example for the Surface Evolver](https://kenbrakke.com/evolver/evolver.htm)