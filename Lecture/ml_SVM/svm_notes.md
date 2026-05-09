- [Understanding the mathematics behind Support Vector Machines](https://shuzhanfan.github.io/2018/05/understanding-mathematics-behind-support-vector-machines/)
  - 介绍了 SVM 的数学原理，包括最大间隔、核函数等概念。


- Anna-Lena Popkes, 2021, Support vector machines, https://alpopkes.com/posts/machine_learning/support_vector_machines/
- Principal component analysis (PCA) https://alpopkes.com/posts/machine_learning/principal_component_analysis/
- Deep (Deep, Deep) Dive into K-Means Clustering, January 27, 2021. https://antoinebrl.github.io/blog/kmeans/

-   Josep Ferrer. 2024. Blog. [Simplifying Support Vector Machines – A Concise Introduction into Binary Classification](https://towardsdatascience.com/support-vector-machines-svm-ml-basics-machine-learning-data-science-getting-started-1683fc99cd45/)

  - 这个是讲的最清楚的

  ![20260503233234](https://fig-lianxh.oss-cn-shenzhen.aliyuncs.com/20260503233234.png){width="90%"}
- [Kernel Functions-Introduction to SVM Kernel &amp; Examples](https://data-flair.training/blogs/svm-kernel-functions/)

### Linear SVMs

![](https://assets.ibm.com/is/image/ibm/3-1_svm_optimal-hyperplane_max-margin_support-vectors-2-1:16x9?fmt=png-alpha&dpr=on%2C1.25&wid=960&hei=540)

Linear SVMs are used with linearly separable data; this means that the data do not need to undergo any transformations to separate the data into different classes. The decision boundary and support vectors form the appearance of a street, and Professor Patrick Winston from MIT uses the analogy of "[fitting the widest possible street](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/resources/mit6_034f10_svm/)" to describe this quadratic optimization problem. Mathematically, this separating hyperplane can be represented as:

$$
w x+b=0
$$

where $w$ is the weight vector, $x$ is the input vector, and $b$ is the bias term.
