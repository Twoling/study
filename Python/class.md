### 类：
> 在面向对象编程，类（英语：class）是一种面向对象计算机编程语言的构造，是创建对象的蓝图，描述了所创建的对象共同的属性和方法。
> 类的更严格的定义是由某种特定的元数据所组成的内聚的包。它描述了一些对象的行为规则，而这些对象就被称为该类的实例。类有接口和结构。接口描述了如何通过方法与类及其实例互操作，而结构描述了一个实例中数据如何划分为多个属性。类是与某个层的对象的最具体的类型。类还可以有运行时表示形式（元对象），它为操作与类相关的元数据提供了运行时支持。

#### 类的定义：
> 在现实世界中，经常有属于同一个类的对象。例如，某辆自行车只是世界上很多自行车中的一辆。在面向对象软件中，也有很多共享相同特征的不同的对象：矩形、雇用记录、视频剪辑等。可以利用这些对象的相同特征为它们创建一个蓝图。对象的软件蓝图称为类。

> 类是定义同一类所有对象的变量和方法的蓝图或原型。例如，可以创建一个定义包含当前档位等实例变量的自行车类。这个类也定义和提供了实例方法（变档、刹车）的实现。

> 实例变量的值由类的每个实例提供。因此，当创建自行车类以后，必须在使用之前对它进行实例化。当创建类的实例时，就创建了这种类型的一个对象，然后系统为类定义的实例变量分配内存。然后可以调用对象的实例方法实现一些功能。相同类的实例共享相同的实例方法。

> 除了实例变量和方法，类也可以定义类变量和类方法。可以从类的实例中或者直接从类中访问类变量和方法。类方法只能操作类变量 - 不必访问实例变量或实例方法。

> 系统在第一次在程序中遇到一个类时为这个类创建它的所有类变量的拷贝 - 这个类的所有实例共享它的类变量。

> 类（Class）是抽象的模板，实例（instance）是根据类创建出来的一个个具体的"对象"，每个对象都拥有相同的方法，但各自的数据可能不同。

#### 类和对象：
> 对象和类的说明其实很相似。实际上，类和对象之间的差别经常是一些困惑的起源。在现实世界中很明显，类不是他描述的对象-自行车的蓝图不是自行车。但是在软件中就有点难以区分类和对象。分是由于软件对象只是现实世界的电子模型或抽象概念。但是也由于很多人用“对象”指类和它们的实例这两者。

* 一个简单的类：

```
class Dog():
	def __init__(self, name, age):
		self.name = name
		self.age = age

	def sit(self):
		print(self.name.title() + " is now sitting.")

	def roll_over(self):
		print(self.name.title() + " rolled over")
```

* __init__() 函数（方法)：初始化已实例化后的对象。
	* 带两个下划线的函数是声明该属性为私有，不能在类的外部被使用或直接访问

> 类中的每个属性都必须有初始值，哪怕这个值是0或者空字符串。在有些情况下，如设置默认值时，在方法__init__()内指定这种初始值时可行的；如果你对某个属性这么做了，就无需包含为它提供初始值的形参了。

#### 继承
> 编写类时，并非总要从空白开始，如果你要编写的类是另一个现成类的特殊版本，可使用继承。一个类继承另一个类时，它将自动获取另一个类的所有属性和方法；原有类称为父类，而新类称为子类。子类继承其父类的所有属性和方法，同事还可以定义自己的属性和方法。

栗子：
```
# 父类
class Dog():
	def __init__(self, name, age):
		self.name = name
		self.age = age

	def sit(self):
		print(self.name.title() + " is now sitting.")

	def roll_over(self):
		print(self.name.title() + " rolled over")

# 子类
class Cokey(Dog):
	def __init__(self, name, age):
		super().__init__(name, age)

#] my_dog = Cokey('cokey', 10)
#] print(my_dog.sit())
```

* 创建子类时，父类必须包含在当前文件中，且位于子类前面；
* 定义子类时，必须在括号内指定父类的名称；
* `super()`函数帮助Python将父类和子类关联起来；
* 父类也称为超类(superclass)；
* 先继承后构建

