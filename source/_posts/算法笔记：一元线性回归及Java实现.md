---
title: 算法笔记：一元线性回归及Java实现
date: 2017-07-23 12:30:50
tags: ['算法','机器学习','Java']
mathjax: true
---
> 这是voidAlex原创的第四篇博文。
> 源码在[我的GitHub](https://github.com/voidAlex/Linear_Regression)
<!-- more -->

## 回归问题
回归问题是研究自变量和因变量之间关系的一种预测模型技术。例如我们可以通过回归模型去预测房价与房子面积之间的关系，一个人每周花在微信上的时间和他微信好友数量之间的关系等。

回归模型定义了输入和输出的关系。输入为现有信息，输出为预测。

一个预测问题在回归模型下的解决步骤为：

1. 构造训练集；
2. 学习，得到输入输出间的关系；
3. 预测，通过学习得到的关系预测输出。

## 线性回归
假设要使用回归模型预测一个人每周花在微信上的时间和微信好友数量之间的关系，可以用如下的表达式表示：

$$ y=ax+b+e $$

其中，*y*是你每周需要花费在微信上的时间，*x*是你的微信好友数量，*e*是误差。对于误差*e*，它不是一个定值，有一对*y*和*x*，就有一个*e*，*e*的值满足正态分布。

假设有数据集：

|好友数量|	花费的时间|
|---|---|
|50	|55|
|53	|56|
|80	|79|
|90	|88|
|63	|58|
|89	|93|
|120 |90|
|155 |120|

将数据集用散点图表示：

![](http://osuro1ft2.bkt.clouddn.com//17-7-14/7010575.jpg)

我们假定*x*和*y*之间的关系确实是线性的，那么可以尝试在散点图上画一条直线：

![](http://osuro1ft2.bkt.clouddn.com//17-7-14/92659690.jpg)

可以看出，我们能画出很多条直线。接下来就是要从存在的直线中确定一条最佳的直线来拟合数据。

如果存在一条最佳拟合的直线，那么所有的样本数据到这条直线的距离应该是最小的。对于线性回归来说，“最佳”指的就是距离最小化。因此，将参数求解问题转换为求最小误差问题。常见的获得最佳拟合线的方法有最小二乘法、梯度下降算法等。

## 使用最小二乘法拟合

对于上面的样本集，我们尝试用$ y=ax+b+e $去进行拟合，那么可以得到：

$$ \mid e \mid = \mid ax + b - y \mid $$

误差大小其实就是猜想的$ax + b$的值和观测到的*y*值之间的差值。把所有的$\mid e \mid$都求和，构造一个函数：

$$Q = \sum_{i=1}^n (ax_i + b - y_i)^2$$

当$Q$最小的时候，即可得到最佳拟合的直线。

可以求导得到$a$和$b$，也可以直接用下面的公式求得：（省略数学推导过程）

$$
a=\frac
{
    \frac
    {\sum\_{i = 1}^n y\_i  \sum\_{i = 1}^n x\_i}
    {n} - 
    \sum\_{i = 1}^n x\_i y\_i
    }
{
    \frac
    {\sum\_{i = 1}^n x\_i * \sum\_{i = 1}^n x\_i}
    {n} - 
    \sum\_{i = 1}^n x\_i^2
    }
$$

$$
b=\frac {\sum\_{i = 1}^n y\_i - a  \sum\_{i = 1}^n x\_i}{n}
$$

得到$a$和$b$之后，可以用R平方来评估拟合程度：

$$
R^2=1-\frac
{\sum\_{i=1}^n (y\_i - \hat{y\_i})^2}
{\sum\_{i=1}^n (y\_i - \bar{y})^2}
$$

R平方可以解释为数据集中能够被模型所解释的方差占数据总方差的比重，R平方值越大，说明模型对数据的拟合程度越高。

## 编码实现

首先构造一个类来存放记录：

```Java
public class DataNode {
    private double x;
    private double y;

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }

    public double getXY(){
        return this.x * this.y;
    }
}
```

然后构造`MyLinearRegression`类：

```Java
public class MyLinearRegression {
    private List<DataNode> list;
    private double alpha;
    private double beta;
    private double r;

    public MyLinearRegression(String path) throws IOException{
        this.list = new ArrayList<DataNode>();

        init(path);
    }

    public double getAlpha() {
        return alpha;
    }

    public double getBeta() {
        return beta;
    }

    public double getR(){
        return this.r;
    }

    private void init(String path) throws IOException{
        BufferedReader reader = new BufferedReader(new FileReader(new File(path)));
        String line = "";

        while ((line = reader.readLine()) != null){
            String str[] = line.split(",");
            DataNode dataNode = new DataNode();
            dataNode.setX(Double.parseDouble(str[0]));
            dataNode.setY(Double.parseDouble(str[1]));
            this.list.add(dataNode);
        }

        reader.close();
    }
}
```

最后要根据读取到的数据去求`alpha`，`beta`和`r`的值。可以直接利用上面的公式：

```Java
public void getAB(){
    int n = list.size();
    double sumX = 0;
    double sumY = 0;
    double sumXY = 0;
    double sumX2 = 0;

    for (DataNode dataNode : list){
        sumX += dataNode.getX();
        sumY += dataNode.getY();
        sumXY += dataNode.getXY();
        sumX2 += Math.pow(dataNode.getX(), 2);
    }
    this.alpha = (((sumY * sumX) / n) - sumXY) / (((sumX * sumX) / n) - sumX2);
    this.beta = (sumY - this.alpha * sumX) / n;
}

public void getR2(){
    double num = 0;
    double den = 0;
    double sumY = 0;

    for (DataNode dataNode : list){
        sumY += dataNode.getY();
    }

    double avgY = sumY / list.size();

    for (DataNode dataNode : list){
        num += Math.pow((dataNode.getY() - (dataNode.getX() * this.alpha + beta)), 2);
        den += Math.pow((dataNode.getY() - avgY), 2);
    }
    this.r = 1 - (num / den);
}
```

使用上述数据集拟合：

```Java
public class Main {
    public static void main(String args[]) throws IOException{
        String path = "test.txt";
        MyLinearRegression linearRegression = new MyLinearRegression(path);
        linearRegression.getAB();
        linearRegression.getR2();
        System.out.println("alpha = " + linearRegression.getAlpha());
        System.out.println("beta = " + linearRegression.getBeta());
        System.out.println("R2 = " + linearRegression.getR());
    }
}
```

得到结果：

```
alpha = 0.6058710840658103
beta = 26.861280144241604
R2 = 0.9001321912140731
```

所以，拟合该数据集的直线为$y=0.61x+26.86$。

## 过拟合和欠拟合

### 过拟合

过拟合，即在拟合过程中“做过头”。在拟合过程中，可能为了迎合所有的样本甚至是噪声点，使得模型的描述过于复杂，或者失去泛化能力。造成过拟合的原因可能有：

1. 训练样本过少；
2. 迎合了所有的样本甚至是噪声点。

### 欠拟合

欠拟合与过拟合相反，由于操作不当导致模型产生的误差$e$分布过于分散或者太大。欠拟合会因为误差太大导致模型没有泛化能力而失去指导意义。造成欠拟合的方法可能有：

1. 参数过少；
2. 拟合方法不当。

## 参考资料

[白话大数据与机器学习](https://item.jd.com/11932929.html)

[数据科学实战](https://item.jd.com/11617070.html?dist=jd)
