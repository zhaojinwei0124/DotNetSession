## 结构
通过使用CFF Explorer打开一个可执行映像，我们可以简单地看到这个映像的文件内容，而省却了手工分析PE格式的痛苦。

![CFF Explorer](docu\Plane Structure of .Net Image.\CFF Explorer.png)

通过PE文件结构判断一个可执行映像文件是否.Net程序集有多种方法，其中一种较为直接的方式就是查看该映像的可选首部当中的第15个数据目录值是否为0——这个数据目录被用于指定.Net首部的位置。

## 元数据
通过.Net首部中提供的偏移，可以获得元数据的起始位置。

元数据包含五个元数据流：
- **#~/#-** 元数据库。元数据在PE映像中是以一个数据库的形式保存的，这个数据库就是元数据库。针对不同规格的程序，元数据库中包含的元数据表的数量是不同的；在最多的情况下，元数据库中可能包含45个表；
- **#Strings**  元数据表中使用的字符串池。元数据库当中的字符串引用指向该数据流；
- **#US**       用户字符串池。用户在代码中使用的字符串常量指向该数据流；
- **#Blob**     无模式数据。二进制对象集中保存于该数据流；
- **#GUID**     包含元数据使用的全局唯一id信息。

## 元数据表
虽然不太可能通过这一篇文章描述全部元数据表的意义，不过我们可以通过解析下面这些表的关系来一窥元数据表设计上的一些常见做法：
  - TypeDef 保留了开发者定义的类的相关信息；
  - Field   类型字段的相关信息。TypeDef表中的记录通过FieldList字段引用该表中的内容；
  - Method  类型的方法表信息。TypeDef表中记录的MethodList字段指向该表中的一个记录，这个记录是TypeDef记录对应的类型的方法表中第一个方法；
  - Property 定义了类的属性信息；
  - PropertyMap 定义了类和方法之间的映射关系。

将上述的元数据表（以及一些其他的常用元数据表）之间的引用关系画出来的话，大致如下：

![Metadata Reference](docu\Plane Structure of .Net Image.\Metadata Reference.png)

元数据表中存在一些常用的设计特征：
- 不同实体之间常常没有直接的引用连接，而是使用独立的引用表建立连接。这往往是因为相关的特性（比如属性）已通过编译器转换成其他实现方法而不再有意义，留存其元数据只是为了供反射设施使用，因此与其降低常用设施（比如类）的性能，还不如将其独立出来；
- 使用签名标识一个实体的部分特性。签名的意义在下面解释；

## 签名

## IL数据
### IL头