---
 title: Markdown数学公式
date: 2021.12.5
author: Aik
img: /source/images/xxx.jpg
top: false
hide: false
cover: false
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: 记录markdown常用的一些数学公式
categories: Markdown
tags:
  - Markdown
---

**行内公式尽量也使用$$,不要用单个$**



**符号**

|         符号          |                  markdown                  |
| :-------------------: | :----------------------------------------: |
|       $\times$        |                 ``\times``                 |
|       $$\cdot$$       |                 ``\cdot``                  |
|      $$\bullet$$      |                ``\bullet``                 |
|     $$\partial$$      |                ``\partial``                |
|         空格          |              ``\hspace{1em}``              |
|    $$\sqrt[3]{x}$$    |              ``\sqrt[3]{x}``               |
|    $$\frac{N}{2}$$    |              ``\frac{N}{2}``               |
| $$\lfloor x \rfloor$$ |           ``\lfloor x \rfloor``            |
|  $$\lceil x \rceil$$  |           ``$$\lceil x \rceil``            |
|  $$\left\{\right\}$$  | ``$$\left\{\right\}$$``  or  ``$$\{ \}$$`` |

**矩阵**

|                        算式                         |                      markdown                       |
| :-------------------------------------------------: | :-------------------------------------------------: |
|       $$\begin{matrix}0&2\\0&0\end{matrix}$$        |       ``\begin{matrix}0&2\\0&0\end{matrix}``        |
| $$\left[\begin{matrix}0&2\\0&0\end{matrix}\right]$$ | ``\left[\begin{matrix}0&2\\0&0\end{matrix}\right]`` |

**上标下标**

|   算式    |  markdown   |
| :-------: | :---------: |
|   $x^2$   |   ``x^2``   |
|   $x_1$   |   ``x_1``   |
|  $x^4_2$  |  ``x^4_2``  |
| $x^{y_2}$ | ``x^{y_2}`` |

**对齐方式**

|       左对齐       |                           markdown                           |
| :----------------: | :----------------------------------------------------------: |
|     第一种方式     | ``\begin{array}{l}中间内容\end{array} 中间的l,r,c表示左右中`` |
| 第二种方式（推荐） | ``\begin{align} \end{align}中间正常使用\\进行换行，用&确定对齐的位置`` |

第一种效果：
$$\begin{array}{l}&cdot o_{2}=o_{1} \cdot\left(o_{1}-1\right) \\
&\frac{\partial y_{1}}{\partial w_{11}^{(2)}}=a_{1}\end{array}$$

```
$$\begin{array}{l}&cdot o_{2}=o_{1} \cdot\left(o_{1}-1\right) \\
&\frac{\partial y_{1}}{\partial w_{11}^{(2)}}=a_{1}\end{array}$$
```



第二种效果：

$$\begin{align}&cdot o_{2}=o_{1} \cdot\left(o_{1}-1\right) \\
&\frac{\partial y_{1}}{\partial w_{11}^{(2)}}=a_{1}\end{align}$$

```
$$\begin{align}&cdot o_{2}=o_{1} \cdot\left(o_{1}-1\right) \\
&\frac{\partial y_{1}}{\partial w_{11}^{(2)}}=a_{1}\end{align}$$
```

还是更推荐用第二种

**编号**

|                      公式                       |                      markdown                       |
| :---------------------------------------------: | :-------------------------------------------------: |
| $$\begin{equation}1+2=3 \tag{1}\end{equation}$$ | ``$$\begin{equation}1+2=3 \tag{1}\end{equation}$$`` |

