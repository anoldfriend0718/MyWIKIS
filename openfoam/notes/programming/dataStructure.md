# Data Structures

## Primitive Types

解开 OpenFOAM 源代码文件可以在 OpenFOAM 文件夹下看到 primitives 文件夹，里面有一系列的文件夹，这些文件夹里保存的就是所有 OpenFOAM 中实现的基础数据类；

最基本的几个有

- label（其实就是整形，不过可能为 int 也可能为 long），
- bool（自己的逻辑变量类），
- char（自己的 char 类），
- scalar（标量类，single 或 double），
- Vector（矢量类，具有三个分量，秩=1，关于秩的概念可参见软件的 programmerguide：）），
- Tensor（张量类，具有 9 个分量，秩=2），
- complex（复数类），
- word（string 的一个子类，用来保存单词，可以自己判断给出的字符能否在单词中使用！），
- string（std::string 的子类，内嵌了自己的哈希表类，增强了 std::string 的功能），
- fileName（string 的子类，用来表示文件名----具有路径的哦好像），
- Pair（具有 2 个配对元素的 list 类，FixedList 的子类，类似于 stl 中的 pair），
- Hash（哈希函数类，用来进行哈希运算）
