---
layout:     post
title:      "GitHub Swift Style Guide"
subtitle:   "仅对 GitHub 原文档进行不太优质翻译"
date:       2014-11-05 12:00:00
author:     "Bruce"
header-img: "img/post-bg-08.jpg"
---


**本文档旨在达到以下目标：**

1. 更加严格，减少程序员错误的可能性；
2. 提供缩进透明度；
3. 减少冗长的东西；
4. 减少关于美学的争论；

### 空格

- 使用 Tab 键而非 Space 键
- 每句代码结束后进行换行
- 自由使用垂直空格来把代码划分为逻辑块
- 不要在句子结尾使用空格
	- 甚至不要对空行进行缩排
	
### 更倾向于在只读属性和 subscript 时隐藏`get`

尽可能在只读属性计算和只读 subscript 时隐藏`get`的使用。

正确的写法：

	var myGreatProperty: Int {
		return 4
	}
	
	subscript(index: Int) -> T {
		return objects[index]
	}
	
错误的写法：

	var myGreatProperty: Int {
		get {
			return 4
		}
	}
	
	subscript(index: Int) -> T {
		get {
			return objects[index]
		}
	}
	
说明：这样做的意图是使得第一个版本足够清晰，使用更少的代码来达到结果。

### 始终明确地指定访问控制顶层的定义

顶层功能，类型，变量需要始终明确接入控制说明符：

	public var whoopsGlobalState: Int
	internal struct TheFez {}
	private func doTheThings(things: [Thing]){}
	
在适当情况下，可以通过以下方式访问隐藏控制：

	internal struct TheFez {
		var owner: Person = Joshaber()
	}
	
说明：在很少的情况下在顶层定义中使用说明符`internal`，使用这一决定需要经过明确的思考。使用一个定义后，再次使用相同的控制说明符仅仅是复制，默认情况通常是合理的。

### 当指定类型时，通常是冒号和标识符相关联

当指定标识符类型时，通常会在标识符后面立即加上冒号，然后空格隔开并开始写入类型名称。

	class SmallBatchSustainableFairtrade: Coffee { ... }
	
	let timeToCoffee: NSTimeInterval = 2
	
	func makeCoffee(type: CoffeeType) -> Coffee { ... }
	
说明：说明符的类型表示有事件与标识符关联，需要放置标识符。

### 仅在需要时明确的使用`self`进行引用

当使用`self`接入参数或方法，默认情况下隐藏`self`引用：

	private class History {
	    var events: [Event]
	
	    func rewrite() {
	        events = []
	    }
	}
	
仅在需要时包含这个键时使用，例如：闭包内或者参数名冲突时：

	extension History {
	    init(events: [Event]) {
	        self.events = events
	    }
	
	    var whenVictorious: () -> () {
	        return {
	            self.rewrite()
	        }
	    }
	}
	
说明：这使得`self`的语义在闭包内更易捕获，并且避免其他部分冗长。

### 更倾向于使用 `struct` 代替 `class`

仅在功能必须使用`class`（例如标识符或 deinitializers）,其他地方通过`struct`代替。

在继承时使用`class`并不是一个好的选择，因为协议会提供多种形态，并且通过构成的提供导致实现可被复用。

`class`层级的列子：

	class Vehicle {
	    let numberOfWheels: Int
	
	    init(numberOfWheels: Int) {
	        self.numberOfWheels = numberOfWheels;
	    }
	
	    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
	        return pressurePerWheel * numberOfWheels
	    }
	}
	
	class Bicycle: Vehicle {
	    init() {
	        super.init(numberOfWheels: 2)
	    }
	}
	
	class Car: Vehicle {
	    init() {
	        super.init(numberOfWheels: 4)
	    }
	}
	
可以重构为以下形式：

	protocol Vehicle {
	    var numberOfWheels: Int { get }
	}
	
	func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
	    return pressurePerWheel * vehicle.numberOfWheels
	}
	
	struct Bicycle: Vehicle {
	    let numberOfWheels = 2
	}
	
	struct Car: Vehicle {
	    let numberOfWheels = 4
	}
	
说明：值类型比较简单，易于推理，通过`let`达到预期的行为。

### 使`final`类作为默认

类是以`final`进行开始，作为唯一可改变的允许子类确认需要继承来被识别。在这个实例中，按照同样的规则尽可能使用`final`。

说明：构成通常是最好的继承，

### 省略类型参数

当与接收机连接时，方法的类型参数可以省略接收类型参数。

例子：

	struct Composite<T> {
	    …
	    func compose(other: Composite<T>) -> Composite<T> {
	        return Composite<T>(self, other)
	    }
	}
	
可渲染为：

	struct Composite<T> {
	    …
	    func compose(other: Composite) -> Composite {
	        return Composite(self, other)
	    }
	}
	
说明：意图是删除冗余的类型参数，并且当返回的类型需要不同类型的参数时形成显著的对比。