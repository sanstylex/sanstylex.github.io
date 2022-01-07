# 转置卷积运算（有误）

参考论文 《Up-sampling with Transposed Convolution》

转置卷积(Transposed Convolution)常常在一些文献中也称之为反卷积(Deconvolution)和部分跨越卷积(Fractionally-strided Convolution)，因为称之为反卷积容易让人以为和数字信号处理中反卷积混起来，造成不必要的误解，因此下文都将称为转置卷积，并且建议各位不要采用反卷积这个称呼。

假设有一个特征图（feature map）$X \in \mathbb{R}^{m \times n}$，有一个卷积核 $K \in \mathbb{R}^{k \times k}$，可以把它们的卷积运算结果 $Y = X * K \in \mathbb{R}^{(m-k+1) \times (n-k+1)}$ 转换为矩阵乘法。（我们假定 padding=0，stride=1）

可以将 $X$ 写作分块矩阵：

$$
X = 
\begin{bmatrix}
   X_{00} & X_{01} & \cdots & X_{0,{n-k}} \\
   X_{10} & X_{11} & \cdots & X_{1,{n-k}}\\
   \vdots & \vdots & \ddots & \vdots \\
   X_{{m-k},0} & X_{{m-k},1} & \cdots & X_{{m-k},{n-k}} \\
\end{bmatrix}
$$

这样，有

$$
Y = \begin{bmatrix}
   X_{00} * K & X_{01} * K & \cdots & X_{0,{n-k}} * K \\
   X_{10} * K & X_{11} * K & \cdots & X_{1,{n-k}} * K\\
   \vdots & \vdots & \ddots & \vdots \\
   X_{{m-k},0} * K & X_{{m-k},1} & \cdots & X_{{m-k},{n-k}} * K \\
\end{bmatrix}
$$

可以反过来思考：将 $K$ 补零和 $X$ 一致，定义：

$$
V = \begin{bmatrix}
   V_{00} & V_{01} & \cdots & V_{0,{n-k}} \\
   V_{10} & V_{11} & \cdots & V_{1,{n-k}}\\
   \vdots & \vdots & \ddots & \vdots \\
   V_{{m-k},0} & V_{{m-k},1} & \cdots & V_{{m-k},{n-k}} \\
\end{bmatrix}
$$

其中 $V_{ij}$ 是 $K$ 补零并模拟滑动的分块矩阵。

---------

首先，将卷积核 $K$ 向右向下补零令其尺寸和 $X$ 相同，获得一个矩阵 $K^r \in \mathbb{R}^{m \times n}$：

$$
\begin{bmatrix}
   K_{00} & K_{01} & \cdots & K_{0,{k-1}} & 0  & 0 & \cdots & 0 \\
   K_{10} & K_{11} & \cdots & K_{1,{k-1}} & 0  & 0 & \cdots & 0 \\
   \vdots & \vdots & \ddots & \vdots & \vdots & \vdots & \ddots & \vdots \\
   K_{{k-1},1} & K_{{k-1},2} & \cdots & K_{{k-1},{k-1}} & 0  & 0 & \cdots & 0 \\
   0 & 0 & \cdots & 0 & 0  & 0 & \cdots & 0 \\
   0 & 0 & \cdots & 0 & 0  & 0 & \cdots & 0 \\
   \vdots & \vdots & \ddots & \vdots & \vdots  & \vdots & \ddots & \vdots \\
   0 & 0 & \cdots & 0 & 0  & 0 & \cdots & 0
\end{bmatrix}
$$

接着将 $K^r$ 按行展开为一个向量 $v \in \mathbb{R}^{mn \times 1}$，接着将其写成循环矩阵的形式：

$$
V = 
\begin{bmatrix}
   v_0^T \\
   v_1^T \\
   \vdots \\
   v_{n−k}^T
\end{bmatrix} \in \mathbb{R}^{(n-k+1) \times mn}
$$

其中 $v_0 = v$，其余行便是 $v$ 的循环排列后的向量：

$$
\begin{cases}
   v_0^T &= (p_0, p_1, \cdots, p_{mn-1}) \\
   v_1^T &= (p_1, p_2, \cdots, p_{mn-1}, p_0)\\
   &\vdots \\
   v_{n−k}^T &= (p_{n−k}, p_{n−k-1}, \cdots, p_{0})
\end{cases}
$$

将 $X$ 按行展开为一个列向量 $Z$。最后，计算 $VZ$，并把得到的结果恢复为矩阵，最终的结果等于卷积运算 $X * K$ 的结果。