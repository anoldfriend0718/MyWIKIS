## Run Time Selection 机制总结

1. 基类里定义一个 hashTable，其 key 为类的 typeName （static 类型，在类内调用宏 TypeName("typeName")申明，在类外调用宏 defineTypeNameAndDebug 定义） ，value 为一个函数指针，这个函数指针指向的函数的返回值是基类类型的 autoPtr ，并且这个 autoPtr 指向类的一个临时对象（用 C++ 的 new 关键字创建 ）。这些在宏函数 declareRunTimeSelectionTable 中完成。

2. 每创建一个派生类，都会调用一次 addToRunTimeSelectionTable 宏函数。这个宏函数会触发一次 hashTable 的更新操作。具体地说，宏函数的调用，会往基类里定义的 hashTable 插入一组值，这组值的 key 是该派生类的 typeName ，value 是一个函数，该函数返回的是指向派生类临时对象的指针。

3. 类及其派生类编译成库，在编译过程中(通过定义全局变量 add##argNames##ConstructorToTable )，会逐步往 hashTable 增加新元素，直到可选的模型全部添加到其中。

4. 在需要调用这些类的地方，只需要定义基类的 autoPtr，并用基类中定义的 New 函数来初始化，即 autoPtr<AlgorithmBase> algorithmPtr = AlgorithmBase::New(algorithmName);。这样， New 函数就能根据调用的时候所提供的参数（即 hashTable 的 key），来从 hashTable 中选择对应的派生类（即 hashTable 的 value）。

实现 RTS，在基类和派生类需要的：
基类类体里调用 TypeName 和 declareRunTimeSelectionTable 两个函数，类体外面调用 defineTypeNameAndDebug 和 defineRunTimeSelectionTable 两个函数。
基类中需要一个静态 New 函数作为 selector。
派生类类体中需要调用 TypeName 函数，类体外调用 defineTypeNameAndDebug （定义 hashTable 的 key）和 addToRunTimeSelectionTable（定义全局变量 add##argNames##ConstructorToTable ，实现 add typename：new 进入 hashTable） 两个宏函数。

## Dimensions

![](img/programming%20guides_2020-11-05-16-31-44.png)
![](img/programming%20guides_2020-11-05-16-32-39.png)

## Fields

![](img/programming%20guides_2020-11-05-16-39-46.png)

## Mesh

![](img/programming%20guides_2020-11-05-16-33-36.png)
![](img/programming%20guides_2020-11-05-16-33-59.png)

![](img/programming%20guides_2020-11-05-16-35-58.png)

## Geometric Fields

![](img/programming%20guides_2020-11-05-16-41-29.png)
![](img/programming%20guides_2020-11-05-16-42-31.png)
![](img/programming%20guides_2020-11-05-16-42-49.png)
![](img/programming%20guides_2020-11-05-16-43-02.png)
![](img/programming%20guides_2020-11-05-16-47-46.png)

## Discretization

![](img/programming%20guides_2020-11-05-16-44-37.png)
![](img/programming%20guides_2020-11-05-16-45-21.png)
![](img/programming%20guides_2020-11-05-16-45-47.png)

## customize a solver

- develop the new equation solver and the workflow
- modify the compilation instructions by adding the new file, Include or libraries
- create a test case
  - create the 0/fields
  - create the necessary input dictionaries in the constant folder
  - update the files in system/fvSchemes and system/fvSolution to take into account the new equation
  - update the controlDict with probing, or sampling, or any other postprocessings

## openFOAM 的基础数据结构汇总

openFOAM 将数组链表等数据结构也进行了封装，这里进行一个汇总。可能陆续也会更新

### 标签 label

其实就是指 i,j,k 这类浮标使用的类型。我们通常就使用 int 就可以，但是这里也进行了封装，具体见如下链接：
https://blog.csdn.net/qq_40583925/article/details/107812187

### 标量 scalar

其实就是浮点数，不过浮点数有多重类型精度，比如 float double longdouble，这里将类型统一为 scalar 这个类型使用，具体见如下链接：
https://blog.csdn.net/qq_40583925/article/details/107735351

### 向量 vector

并不是 C++中常用的容器 vector，而是指(x,y,z)这样的长度为 3 的行向量，用来表示速度坐标等。这是因为 openFOAM 从 90 年代开始迭代遗留下来的习惯，具体见如下链接：
https://blog.csdn.net/qq_40583925/article/details/107735914
不过它还有对应的拓展，比如二维的 vector2D。以及列向量 RowVector。

### 张量 tensor

是指 3\*3 的张量，具体见如下链接：
https://blog.csdn.net/qq_40583925/article/details/107737705
它也有对应的拓展，如’Tensor2D SymmTensor SymmTensor2D DiagTensor’

### 数组 UList 和 List

是可以指定类型的一维数组，具有和 vector 数组类似的功能，具体见如下链接：
https://blog.csdn.net/qq_40583925/article/details/106963933

### 场 Field

用来存储我们平时说到的速度域，压力域等。当然这需要网格相关的信息，会非常的复杂，具体见如下链接：
https://blog.csdn.net/qq_40583925/article/details/107800987
