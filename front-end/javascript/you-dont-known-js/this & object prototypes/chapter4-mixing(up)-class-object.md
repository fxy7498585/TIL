# You Don't Know JS: *this* & Object Prototypes

# Chapter 4: Mixing (Up) "Class" Objects

在我们探索前一章中的对象之后，我们现在将注意力转向“面向对象（OO）编程”，“类”。我们将首先将“面向类”视为一种设计模式，然后研究“类”的机制:“实例化”、“继承”和“(相对)多态性”。

我们将看到，这些概念并不能很自然地映射到JS中的对象机制，而且许多JavaScript开发人员需要克服这些挑战。

**注意：** 这一章花了相当多的时间(前半部分!)在着重解释“面向对象编程”理论上。当我们在下半部分讨论“mixin”时，我们最终将这些想法与实际的JavaScript代码联系起来。但是有很多概念和伪代码需要首先仔细阅读，所以不要迷失方向——只要坚持下去!

## Class Theory

“类/继承”描述了一种特定形式的代码组织和体系结构——一种在我们的软件中建模的方法。

面向对象(OO)或面向类编程强调数据本质上具有对其进行操作的相关行为(当然，这取决于数据的类型和性质!)，因此正确的设计是将数据和行为打包(也就是封装)在一起。这有时被称为正式计算机科学中的“数据结构”。

例如，表示单词或短语的一系列字符通常称为“字符串”。字符是数据。但是你几乎从不只关心数据，你通常希望对数据进行处理，因此可以应用于该数据的行为(计算其长度、追加数据、搜索等)都被设计为`String`类的方法。

任何给定的字符串只是该类的一个实例，这意味着它是一个整齐地收集的字符数据和我们可以在其上执行的功能的包装。

类也意味着对某个数据结构进行 *分类* 的方法。我们这样做的方法是将任何给定的结构看作是更一般的基本定义的一个特定变体。

让我们通过查看一个常用的例子来探索这个分类过程。*汽车* 可以被描述为一种更一般的“类”事物的具体实现，称为*车辆* 。

我们通过定义`Vehicle`类和`Car`类在软件中使用类来建模这种关系。

`Vehicle`的定义可能包括推进（发动机等）、载人能力等，这些都是行为。我们在`Vehicle`中定义的是所有（或大多数）不同类型的车辆（“飞机，火车和汽车”）共有东西。

在我们的软件中，为每种不同类型的车辆重新定义“载人能力”的基本本质可能没有意义。相反，我们在`Vehicle`中定义一次该功能，然后在我们定义`Car`时，我们只是表明它“继承”（或“扩展”）来自`Vehicle`的基本定义。`Car`的定义被称为专用于一般的`Vehicle`定义。

虽然`Vehicle`和`Car`通过方法共同定义行为，但是实例中的数据可能是特定车辆的唯一VIN等。

**因此，类、继承和实例化就出现了。**

类的另一个关键概念是“多态性”，它描述了一个概念，即父类的一般行为可以在子类中被重写，以提供更详细的信息。事实上，相对多态性允许我们从被覆盖的行为中引用基本行为。

类理论强烈建议父类和子类为某个行为共享相同的方法名，以便子类覆盖父类（差异）。正如我们稍后将看到的，在JavaScript代码中这样做会导致挫败感和代码脆弱性。

### "Class" Design Pattern

你可能从来没有将类看作是“设计模式”，因为最常见的情况是讨论流行的“OO设计模式”，如“迭代器”、“观察者”、“工厂”、“单例”等。以这种方式呈现，这几乎是假设OO类是实现所有(更高级别)设计模式的底层机制，就好像OO是 *所有* (适当的)代码的给定基础。

根据你在编程方面的正规教育程度，你可能听说过“过程编程”，它是一种描述代码的方法，代码只包含调用其他函数的过程(即函数)，没有任何更高的抽象。你可能已经学过，类是将过程式风格程序的“意大利面条式代码”转换为格式良好、组织良好的代码的正确方法。

当然，如果你有“函数式编程”(Monads，等等)的经验，你就会非常清楚类只是几种常见设计模式之一。但是对于其他人来说，这可能是你第一次问自己类是否真的是代码的基础，或者它们是否是代码之上的可选抽象。

有些语言（比如Java）没有给你选择，所以它根本不是 *可选* 的 - 一切都是类。其他语言，如C/ c++或PHP，既提供过程语法，也提供面向类的语法，这更多地取决于开发人员选择哪种风格或哪种风格的混合是合适的。

### JavaScript "Classes"

JavaScript在这方面处于什么位置? JS已经有 *一些* 类似于类的语法元素（比如`new`和`instanceof`）已经有一段时间了，最近在ES6中有一些补充，比如`class`关键字（见附录A）。

但这是否意味着JavaScript真的 *有* 类呢?简单明了: **没有**。

由于类是一种设计模式，你 *可以* 通过相当多的工作(我们将在本章的其余部分看到)来实现许多近似经典类的功能。JS试图通过提供类似类的语法来满足用类进行设计的普遍需求。

虽然我们可能有一个看起来像类的语法，但JavaScript机制似乎在使用 *类设计模式* 与你作对，因为在幕后，你所构建的机制的操作方式是完全不同的。语法糖和(非常广泛使用的)JS“类”库在很大程度上向你隐瞒了这一事实，但你迟早将面对这样一个事实，即你在其他语言中拥有的类与你在JS中伪装的“类”不同。

这可以归结为，类在软件设计中是可选的模式，你可以选择是否在JavaScript中使用它们。由于许多开发人员对面向类的软件设计有很强的亲和力，我们将在本章的其余部分探索如何使用JS提供的东西来维护类的假象，以及我们所经历的痛点。

## Class Mechanics

在许多面向类的语言中，“标准库”作为`Stack`类提供“堆栈”数据结构(push、pop等)。这个类有一组存储数据的内部变量，还有一组由类提供的可公开访问的行为(“方法”)，这使代码能够与(隐藏的)数据交互(添加和删除数据，等等)。

但是在这样的语言中，你实际上并不直接对`Stack`进行操作(除非创建 **静态** 类成员引用，这超出了我们的讨论范围)。`Stack`类仅仅是对 *任何* “栈”应该做什么的一个抽象解释，但它本身并不是一个“栈”。在对具体的数据结构进行操作之前，必须 **实例化** `Stack`类。

### Building

“类”和“实例”思维的传统隐喻来自于建筑构造。

建筑师设计出一座建筑物的所有特征：有多宽，有多高，有多少窗户，在什么位置，甚至墙壁和屋顶使用什么类型的材料。在这一点上，她并不需要关心，将在 *哪里* 建造这个建筑，也不需要关心将会建造 *多少* 个副本。

她也不太关心建筑的内容——家具、墙纸、吊扇等——只关心它们将被包含在什么类型的结构中。

她设计的建筑蓝图只是一个建筑 *规划* 。它们实际上并不构成我们可以走进坐下来的建筑。我们需要一个建设者来完成这项任务。建筑商将采取这些计划, 并在建造大楼时准确地遵循这些计划。从非常真实的意义上说, 他正在将预期的特征从计划 *拷贝* 到物理建筑。

一旦完成，这座建筑就是蓝图计划的物理实例，希望是一个基本上完美的 *拷贝*。然后建设者可以移动到隔壁的空地上，重新做一遍，再创建一个 *拷贝*。

建筑和蓝图之间的关系是间接的。你可以检查蓝图，以了解建筑物的结构，但对于直接考察建筑物的每一部分，仅有蓝图是不够的。但如果你想打开一扇门，你必须去大楼本身——蓝图只是在一张纸上画了几条线，*表示* 门应该在哪里。

类就是一个蓝图。为了实际获得一个我们可以交互的对象，我们必须从类中构建（又称“实例化”）一些东西。这种“构造”的最终结果是一个对象，通常称为“实例”，我们可以根据需要直接调用方法并从中访问任何公共数据属性。

对于类所描述的所有特征来说，**这个对象是一个副本** 。

你可能不会想到，当你走进一栋大楼，发现里面有一幅蓝图，被裱起来挂在墙上，这幅蓝图是用来规划这栋大楼的，不过这份蓝图很可能已经在公共档案办公室(public records office)存档。类似地，通常不使用对象实例直接访问和操作其类，但通常至少可以确定对象实例来自 *哪个类* 。

考虑类与对象实例的直接关系比考虑对象实例与其所属类之间的任何间接关系更有用。**类通过复制操作实例化为对象形式。**

![](https://raw.githubusercontent.com/getify/You-Dont-Know-JS/master/this%20%26%20object%20prototypes/fig1.png)

如你所见，箭头从左向右移动，从上向下移动，这表明在概念上和物理上都发生了复制操作。

### Constructor

类的实例由类的特殊方法构造，通常与类同名，称为 *构造函数* 。此方法的显式工作是初始化实例所需的任何信息(状态)。

例如，考虑这个松散的伪代码（发明语法）用于类：

```js
class CoolGuy {
	specialTrick = nothing

	CoolGuy( trick ) {
		specialTrick = trick
	}

	showOff() {
		output( "Here's my trick: ", specialTrick )
	}
}
```

要 *创建* 一个`CoolGuy`实例，我们将调用类构造函数：

```js
Joe = new CoolGuy( "jumping rope" )

Joe.showOff() // Here's my trick: jumping rope
```

注意，`CoolGuy`类有一个构造函数`CoolGuy()`，这实际上是我们在说`new CoolGuy(..)`时所调用的。我们从构造器中得到一个对象（类的一个实例），然后我们可以调用方法`showoff()`，它打印出特定的`CoolGuys`特殊技巧。

*显然，跳绳让Joe变得很酷。*

类的构造函数属于该类，几乎普遍与类具有相同的名称。此外，构造函数几乎总是需要使用`new`来调用，以使语言引擎知道你要构造 *新的* 类实例。

## Class Inheritance

在面向类的语言中，不仅可以定义可以实例化自身的类，还可以定义从第一个类 **继承** 的另一个类。

第二个类通常被称为“子类”，而第一个类是“父类”。这些术语显然来自父母和孩子的隐喻，尽管这里的隐喻有点牵强，稍后你将看到。

当父母有亲生子女时，父母的遗传特征会被复制到子女身上。显然，在大多数生物生殖系统中，双亲共同为这一组合贡献基因。但是为了隐喻的目的，我们假设只有一个。

一旦孩子存在，他或她就与亲人分开。这孩子受到父母遗传的影响很大，但很独特。如果孩子的头发是红色的，这并不意味着父母的头发是红的或自动变红了。

以类似的方式，一旦定义了子类，它就与父类分开并且不同。子类包含来自父级的行为的初始副本，但可以覆盖任何继承的行为，甚至可以定义新行为。

重要的是要记住，我们正在讨论父子 **类** ，这些不是物理的东西。这就是父类和子类的比喻变得有点混乱的地方，因为我们实际上应该说父类就像父类的DNA而子类就像子类的DNA。我们必须从每一组DNA中造出一个人(也就是“实例化”)，以实际上有一个与之交谈的自然人。

让我们把亲生父母和孩子放在一边，通过一个稍微不同的视角来看待遗传：不同类型的车辆。这是理解继承的最典型（而且常常值得争议）隐喻之一。

让我们重新回顾一下本章前面的`Vehicle`和`Car`讨论。考虑一下继承类的松散伪代码（发明语法）：

```js
class Vehicle {
	engines = 1

	ignition() {
		output( "Turning on my engine." )
	}

	drive() {
		ignition()
		output( "Steering and moving forward!" )
	}
}

class Car inherits Vehicle {
	wheels = 4

	drive() {
		inherited:drive()
		output( "Rolling on all ", wheels, " wheels!" )
	}
}

class SpeedBoat inherits Vehicle {
	engines = 2

	ignition() {
		output( "Turning on my ", engines, " engines." )
	}

	pilot() {
		inherited:drive()
		output( "Speeding through the water with ease!" )
	}
}
```

**注意：** 为了清楚和简洁，忽略了这些类的构造函数。

我们定义 `Vehicle` 类，假定它有一个引擎，有一个打开点火装置的方法，和行驶方式。但是你不会只制造一个通用的“车辆”，所以它在这一点上真的只是一个抽象的概念。

那么我们定义两种特定类型的车辆：`Car`和`SpeedBoat`。它们各自继承了`Vehicle`的一般特征，但随后它们专门针对每种特性进行了特殊处理。一辆汽车需要4个轮子，一艘快艇需要2个引擎，这意味着它需要额外的注意来打开两个引擎的点火。

### Polymorphism

`Car`定义了自己的`drive()`方法，该方法覆盖了从`Vehicle`继承的同名方法。但随后，`Car`的`drive()`方法调用`inherited:drive()`，这表明`Car`可以引用它继承的，覆盖之前的原版 `drive()`。`SpeedBoat` 的 `pilot()` 方法也引用了它继承的 `drive()` 拷贝。

该技术称为“多态”或“虚拟多态”。更具体地说，我们现在称之为“相对多态”。

多态性是一个比我们将在这里讨论的更广泛的主题，但是我们当前的“相对”语义指的是一个特定的方面:任何方法都可以在继承层次结构的更高级别上引用另一个方法(名称相同或不同)。我们之所以说“相对”，是因为我们不确定要访问哪个继承级别（也就是class），而是相对地通过说“向上看一个级别”来引用它。

在许多语言中，使用关键字`super`代替本例的`inherited:`，这是基于“超类”是当前类的父类/祖先类的思想。

多态性的另一个方面是，一个方法名可以在继承链的不同级别上有多个定义，在解析要调用哪些方法时，会根据需要自动选择这些定义。

我们在上面的示例中看到了两次出现的行为：在`Vehicle`和`Car`中定义了`drive()`，在`Vehicle`和`SpeedBoat`中定义了`ignition()`。

**注意：** 传统的面向类语言通过`super`提供的另一件事是，子类的构造函数直接引用其父类的构造函数。这很大程度上是正确的，因为对于真正的类，构造函数属于该类。然而，在JS中，情况正好相反——实际上更适合考虑属于构造函数的“类”（`Foo.prototype…`类型引用）。由于在JS中，子对象和父对象之间的关系只存在于两个`.prototype`对象之间，因此，各个构造函数本身并不直接相关，因此没有简单的方法可以相对地引用其中一个对象（ES6的`class`用`super`“解决”这个问题，请参见附录A）。

在`ignition()`中可以看到多态性的一个有趣的含义。在`pilot()`中，对（继承的）`Vehicles`版本的`drive()`进行了相对多态引用。但是那个`drive()`只是按名称引用了`ignition()`方法（没有相对引用）。

语言引擎使用哪种版本的`ignition()`，来自`Vehicle`或`SpeedBoat`的版本？**它使用`SpeedBoat`版本的`ignition()`。** 如果你 *要* 实例化`Vehicle`类本身，然后调用其`drive()`，语言引擎将只使用`Vehicle`的 `ignition()`方法定义。

换句话说，方法`ignition()` *多态* 的定义将根据你引用的实例所属的类（继承级别）进行更改。

这可能看起来像是过于深刻的学术细节。但是理解这些细节对于正确对比JavaScript的`[[Prototype]]`机制中类似(但不同)的行为是必要的。

在继承类时，**类本身** (而不是从中创建的对象实例!)有一种方法可以相对引用继承的类，这种相对引用通常称为`super`。

记住前面的那幅图：

![](https://raw.githubusercontent.com/getify/You-Dont-Know-JS/master/this%20%26%20object%20prototypes/fig1.png)

注意如何实例化（`a1`，`a2`，`b1`和`b2`）和继承（`Bar`），箭头表示拷贝操作。

从概念上讲，似乎子类`Bar`可以使用相对多态引用（也就是`super`）访问其父类`foo`中的行为。但是，实际上，子类只是从其父类中获得了继承行为的拷贝。如果子类“重写”它继承的方法，则实际上会维护方法的原始版本和被重写的版本，以便它们都是可访问的。

不要让多态性让你误以为将子类链接到其父类。相反，子类从父类中获取所需内容的副本。**类继承意味着拷贝。** 

### Multiple Inheritance

回想一下我们之前对父母和孩子以及DNA的讨论？我们说这个比喻有点奇怪，因为生物学上大多数后代来自两个双亲。如果一个类可以从其他两个类继承，那么它将更接近于父/子隐喻。

一些面向类的语言允许你指定多个“父”类来“继承”。多重继承意味着将每个父类定义拷贝到子类中。

从表面上看，这似乎是对类导向的强大补充，使我们能够将更多功能组合在一起。但是，肯定会出现一些复杂的问题。如果两个父类都提供了一个名为`drive()`的方法，那么该子类中的`drive()`引用会解析为哪个版本？你是否总是需要手动指定你所指的父级的`drive()`，从而失去多态继承的优雅？

还有另一种变体，即所谓的“菱形问题”，它指的是子类“D”继承自两个父类(“B”和“C”)，而这两个类又分别继承自公共的“A”父类。如果“A”提供了一个方法`drive()`，而“B”和“C”都覆盖(多态)该方法，当`D`引用`drive()`时，它应该使用哪个版本(`B:drive()`或`C:drive()`)?

![](https://raw.githubusercontent.com/getify/You-Dont-Know-JS/master/this%20%26%20object%20prototypes/fig2.png)

这些复杂的情况甚至比这个快速的一瞥更加深刻。我们在这里讨论它们只是为了对比JavaScript的机制是如何工作的。

JavaScript更简单：它不提供“多重继承”的原生机制。许多人认为这是一件好事，因为复杂性的节省超过了“减少”功能。但这并不能阻止开发人员以各种方式伪造它，正如我们接下来会看到的那样。

## Mixins

当你“继承”或“实例化”时，JavaScript的对象机制不会自动执行拷贝行为。显然，JavaScript中没有“类”来实例化，只有对象。并且对象不会被拷贝到其他对象，它们会被 *链接* 在一起（更多内容将在第5章中介绍）。

由于在其他语言中观察到的类行为意味着拷贝，让我们检查JS开发人员如何在JavaScript中 **伪造** 类 *缺失* 的拷贝行为：mixins(混合)。我们将看到两种类型的“混合”: **显式** 和 **隐式** 。

### Explicit Mixins

让我们再次重温之前的`Vehicle`和`Car`示例。由于JavaScript不会自动将行为从`Vehicle`拷贝到`Car`，因此我们可以创建一个手动拷贝的实用程序。这样的实用程序通常被许多库/框架称为`extend(..)`，但为了说明的目的，我们将其称为`mixin(..)`。

```js
// vastly simplified `mixin(..)` example:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// only copy if not already present
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}

var Vehicle = {
	engines: 1,

	ignition: function() {
		console.log( "Turning on my engine." );
	},

	drive: function() {
		this.ignition();
		console.log( "Steering and moving forward!" );
	}
};

var Car = mixin( Vehicle, {
	wheels: 4,

	drive: function() {
		Vehicle.drive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	}
} );
```

**注意<font size=3>⚠️</font>：** 微妙但重要的是，我们不再处理类，因为JavaScript中没有类。`Vehicle`和`Car`只是我们分别复制的对象。

`Car`现在拥有`Vehicle`的属性和功能的拷贝。从技术上讲，函数实际上并不重复，而是拷贝对函数的引用。因此，`Car`现在有一个名为`ignition`的属性，它是对`ignition()`函数的拷贝引用，以及一个名为`engine`的属性，其复制值为`1`。

`Car` *已经* 有了一个`drive`属性（函数），因此没有覆盖属性引用（参见上面`mixin(..)`中的`if`语句）。

#### "Polymorphism" Revisited

我们来看看这个语句：`Vehicle.drive.call( this )`。这就是我所说的“显式伪多态”。回想一下我们之前的伪代码，这一行是`inherited:drive()`，我们称之为“相对多态”。

JavaScript没有（在ES6之前;参见附录A）相对多态性的工具。因此，**因为`Car`和`Vehicle`都具有相同名称的函数：`drive()`** ，以区分对一个或另一个的调用，我们必须进行绝对（非相对）引用。我们按名称显式指定`Vehicle`对象，并在其上调用`drive()`函数。

但是如果我们说`Vehicle.drive()`，那个函数调用的`this`绑定将是`Vehicle`对象而不是`Car`对象（参见第2章），这不是我们想要的。因此，我们使用`.call(this)`（第2章）来确保在`Car`对象的上下文中执行`drive()`。

**注意：** 如果`Car.drive()`的函数名称标识符没有重叠(也就是“shadowed”;参见第5章)`Vehicle.drive()`，我们就不会执行“方法多态性”。因此，通过`mixin(..)`调用将对`Vehicle.drive()`的引用进行拷贝，我们可以直接使用`this.drive()`进行访问。选择的标识符重叠 **阴影(shadowing)** 是我们必须使用更复杂的 *显式伪多态性* 方法的*原因* 。

在具有相对多态性的面向类语言中，`Car`和`Vehicle`之间的链接只在类定义的顶部建立一次，这使得只需要在一个地方维护这种关系。

但由于JavaScript的特殊性，显式伪多态(由于阴影！) **在每个需要这种（伪）多态参考的函数中** 创建了脆弱的手动/显式链接。这会显着增加维护成本。而且，虽然显式伪多态可以模拟“多重继承”的行为，但它只会增加复杂性和脆弱性。

这些方法的结果通常更复杂，更难以阅读，并且难以维护代码。**应尽可能避免明确的伪多态性** ，因为在大多数方面成本超过了收益。

#### Mixing Copies

回想一下上面的`mixin(..)`实用程序：

```js
// vastly simplified `mixin()` example:
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// only copy if not already present
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}
```

现在，我们来研究一下`mixin(..)`是如何工作的。它迭代`sourceObj`（在我们的示例中是`Vehicle`）的属性，如果在`targetObj`（在我们的示例中是`car`）中没有该名称的匹配属性，它将生成一个拷贝。因为我们是在初始对象存在的情况下进行拷贝，所以我们要小心不要拷贝存在的目标属性。

如果我们在指定`Car`特定内容之前先进行拷贝，我们可以忽略`targetObj`上的检查，但是这样做比较笨拙，效率也比较低，所以通常不太推荐:

```js
// alternate mixin, less "safe" to overwrites
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		targetObj[key] = sourceObj[key];
	}

	return targetObj;
}

var Vehicle = {
	// ...
};

// first, create an empty object with
// Vehicle's stuff copied in
var Car = mixin( Vehicle, { } );

// now copy the intended contents into Car
mixin( {
	wheels: 4,

	drive: function() {
		// ...
	}
}, Car );
```

无论是哪种方法，我们都明确地将`Vehicle`的非重叠内容复制到`Car`中。“mixin”这个名字来自于另一种解释任务的方式: `Car`拥有 **混合** 的`Vehicle`内容，就像你在你最喜欢的饼干面团中加入巧克力屑。

由于拷贝操作的结果，`Car`将在一定程度上与`Vehicle`分开操作。如果在`Car`上添加属性，则不会影响`Vehicle`，反之亦然。

**注意：** 这里略读了一些小细节。即使在拷贝之后，这两个对象仍有一些微妙的方式可以“影响”彼此，例如它们都共享对公共对象（如数组）的引用。

由于这两个对象也共享对其通用功能的引用，这意味着，**即使是从一个对象到另一个对象手动复制函数（也称为mixin），也不会 *实际上模拟* 面向类语言中出现的从类到实例的真正复制。**

Javascript函数不能真正地被复制（以标准、可靠的方式），所以你最终得到的是对同一个共享函数对象的 **重复引用** （函数是对象；请参见第3章）。例如，如果你通过在共享函数对象(如`ignition()`)之上添加属性来修改其中一个共享函数对象，那么`Vehicle`和`Car`都将通过共享引用受到“影响”。

显式mixins是JavaScript中的一个很好的机制。但它们似乎比实际上更强大。实际上，和 **对每个对象只定义两次属性** 相比，将属性从一个对象复制到另一个对象并没有多大好处。特别是考虑到我们刚才提到的函数对象引用的细微差别。

如果你显式地将两个或多个对象混合到目标对象中，你可以 **部分地模拟** “多重继承”的行为，但是如果从多个源复制相同的方法或属性，则无法直接处理冲突。一些开发人员/库已经提出了“后期绑定”技术和其他奇特的解决方案，但是从根本上说，这些“技巧”通常比回报更费力(而且性能更差!)

注意只使用显式mixin，它实际上有助于使代码更具可读性，如果你发现它使得代码更难以跟踪，或者如果你发现它在对象之间创建了不必要或笨拙的依赖关系，那么就要避免使用这种模式。

**如果正确使用mixin比在使用它们之前变得更加 *困难* ** ，那么您可能应该停止使用mixin。事实上，如果你必须使用一个复杂的库/实用程序来计算所有这些细节，这可能表明你正在以更难的方式处理它，也许是不必要的。在第6章中，我们将尝试提取一种更简单的方法，在不受干扰的情况下实现期望的结果。

#### Parasitic Inheritance

这种显式混合模式的变体，在某些方面是显式的，在另一些方面是隐式的，称为“寄生继承”，主要由Douglas Crockford推广。

以下是它的工作原理：

```js
// "Traditional JS Class" `Vehicle`
function Vehicle() {
	this.engines = 1;
}
Vehicle.prototype.ignition = function() {
	console.log( "Turning on my engine." );
};
Vehicle.prototype.drive = function() {
	this.ignition();
	console.log( "Steering and moving forward!" );
};

// "Parasitic Class" `Car`
function Car() {
	// first, `car` is a `Vehicle`
	var car = new Vehicle();

	// now, let's modify our `car` to specialize it
	car.wheels = 4;

	// save a privileged reference to `Vehicle::drive()`
	var vehDrive = car.drive;

	// override `Vehicle::drive()`
	car.drive = function() {
		vehDrive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	};

	return car;
}

var myCar = new Car();

myCar.drive();
// Turning on my engine.
// Steering and moving forward!
// Rolling on all 4 wheels!
```

如你所见，我们首先从`Vehicle`“父类”(对象)中复制定义，然后混合“子类”(对象)定义(根据需要保留父类引用)，并将这个组合的对象`car`作为子类实例传递。

**注意：** 当我们调用`new Car()`时，将创建一个新对象并由`Car`的`this`引用(参见第2章)。但由于我们不使用该对象，而是返回我们自己的`car`对象，因此最初创建的对象将被丢弃。因此，可以在没有`new`关键字的情况下调用`Car()`，并且上面的功能将是相同的，但不会浪费对象创建/垃圾收集。

### Implicit Mixins

如前所述，隐式mixin与显式伪多态性密切相关。同样，它们也带有同样的警告。

考虑这个代码：

```js
var Something = {
	cool: function() {
		this.greeting = "Hello World";
		this.count = this.count ? this.count + 1 : 1;
	}
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
	cool: function() {
		// implicit mixin of `Something` to `Another`
		Something.cool.call( this );
	}
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 (not shared state with `Something`)
```

使用`Something.cool.call(this)`，可以在“构造函数”调用(最常见)或方法调用(此处显示)中发生，我们基本上“借用”函数`Something.cool()`并在`Another`的上下文中调用它（通过`this`的绑定;参见第2章）而不是`Something`。最终结果是`Something.cool()`所做的赋值将应用于`Another`对象而不是`Something`对象。

所以，有人说我们“混合”了`Something`s的行为和（或混入） `Another`行为。

虽然这种技术似乎利用了`this`重新绑定功能的优势，但它是脆弱的`Something.cool.call( this )`调用, 这个调用不能被转换成一个相对的(因而更灵活的)引用，你应该 **谨慎地使用** 。通常，**尽可能避免这样的结构** 来保持更清晰和更易维护的代码。

## Review (TL;DR)

类是一种设计模式。许多语言提供的语法支持自然的面向类的软件设计。JS也有类似的语法，但它的行为与你习惯使用其他语言的类的行为 **非常不同** 。

**类意味着拷贝。**

在实例化传统类时，会在类和实例之间拷贝行为。继承类时，也会发生从父级到子级的行为拷贝。

多态性(在具有相同名称的继承链的多个级别上具有不同的函数)似乎暗示了从子到父的引用相对链接，但它仍然只是拷贝行为的结果。

JavaScript **不会自动** 在对象之间创建拷贝（如类所示）。

mixin模式(包括显式和隐式)通常用于 *某种程度上* 模拟类的拷贝行为，但这通常会导致难看和脆弱的语法，比如显式伪多态性(`OtherObj.methodName.call(this, ...)`)，这通常会导致难于理解和维护代码。

显式mixin也不完全与类拷贝相同，因为对象(和函数!)只复制了共享引用，而没有复制对象/函数本身。不注意这种细微差别是各种陷阱的根源。

一般来说，在JS中伪造类往往会给未来的编码埋下比解决当前实际问题更多的地雷。