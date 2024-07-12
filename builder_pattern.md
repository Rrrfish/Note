# 建造者模式



建造者模式是为了解决创建类时的繁琐过程，它可以屏蔽掉创建类的内部细节，只需**按照使用者的想法，通过`Builder`一步步装配**。

# 示例

```java
// com/example/demo1/Student

public class Student {
    private int id;
    private String name;
    private int age;
    private String gender;

    private Student(int id, String name, int age, String gender) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    public String toString() {
        return "id:" + id + ", name:" + name + ", age:" + age + ", gender:" + gender;
    }

    public static class Builder {
        private int id;
        private String name;
        private int age;
        private String gender;

        public Builder id(int id) {
            this.id = id;
            return this;
        }

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder gender(String gender) {
            this.gender = gender;
            return this;
        }

        public Student build() {
            return new Student(id, name, age, gender);
        }
    }
}

// com/example/demo1/Main
public class Main {
    public static void main(String[] args) {
        Student student = new Student.Builder()
                .id(1)
                .age(15)
                .gender("female")
                .name("Aha")
                .build();
        System.out.println(student.toString());
    }
}
```



# 为什么`Builder`要设计成静态内部类？

首先看什么是内部类

## Java内部类

> 内部类是一个编译时的概念，一旦编译成功，就会成为**完全不同的两类**。 对于一个名为outer的外部类和其内部定义的名为inner的内部类。编译完成后出现`outer.class`和`outer$inner.class`两类。

内部类主要有以下几类：

- 成员内部类
- 局部内部类
- 静态内部类
- 匿名内部类



### 成员内部类

作为外部类的一个**成员**存在，与外部类的属性、方法并列。

内部成员类的要点就是，**想创建内部类必须要有一个外部类的实例再进行创建**！

不可以有静态成员（方法和变量都不行）。

访问成员内部类的几个途径：

1. 外部类的静态方法访问内部类
2. 外部类的外部访问内部类

这两种方法其实差不多。

```java
package com.example.demo1;

public class Outer {
    private String field;

    public Outer(String field) {
        this.field = field;
    }
    
    public class Inner {
        private String field;
        private String differentField;

        public Inner(String field) {
            this.field = field;
        }
        
        public void call() {
            System.out.println("I'm inner, my field is " + field);
			//在内部类中访问内部类自己的变量直接用变量名
			System.out.println(field);
			//在内部类中访问内部类自己的变量也可以用this.变量名
			System.out.println(this.field);
			//在内部类中访问外部类中与内部类同名的实例变量用外部类名.this.变量名
			System.out.println(Outer.this.field);
			//如果内部类中没有与外部类同名的变量，则可以直接用变量名访问外部类变量
			System.out.println(differentField);
        }
    }
    
    public static void callInner() {
        Outer out = new Outer("outer field", "outer different field");
        Inner inner = out.new Inner("inner field");
        inner.call();
    }
}
```

在外部类的静态方法中访问内部类时，需要用**外部类的实例**创建内部类。`out.new Inner("inner field")`。

```java
	public static void main(String[] args) {
		Outer out = new Outer();
		Outer.Inner inner = out.new Inner();
		inner.call();
	}
```

在外部类的外部访问内部类时，也需要使用外部类的实例来创建内部类，唯一不同的是，内部类的类名展示时还需要有外部类的前缀。

```java
Inner inner = out.new Inner();  //通过外部类的静态类创建内部类
Outer.Inner inner = out.new Inner();   //通过外部类外部的方法创建内部类
```



### 局部内部类

在**方法中定义的内部类**称为局部内部类。

与局部变量类似，局部内部类**不能有访问说明符**，由于局部内部类的作用域被限制在这个块中，因此外部代码无法直接访问它，也就无需（也无法）定义访问说明符（如 `public`、`private` 等）。

可以访问外部类的局部变量(即方法内的变量)，但是变量必须是**final**的，和此外围类所有的成员。

不可以有静态成员（方法和变量都不行）。

```java

public class Outer {
    private String field;
    private String differentField;

    public Outer(String field, String differentField) {
        this.field = field;
        this.differentField = differentField;
    }

    public void callInner() {
        String innerField = "InnerField";
        final int i = 3;
        
        class Inner {
            private String field;
            // public static String name;  不可以有静态成员
            public Inner(String field) {
                this.field = field;
            }
            public void call() {
                System.out.println("I'm inner, my field is " + field);
                //可以访问外部类的局部变量(即方法内的变量)，但是变量必须是final的
                System.out.println("I can reach the final local varible: " + i + "in the method");
            }
        }
        Inner inner = new Inner(innerField);
        inner.call();
    }
}
```



```java
public static void main(String[] args) {
    Outer outer = new Outer("outer field", "outer different field");
    outer.callInner();
}
```



### 静态内部类（嵌套类）

> 如果你不需要内部类对象与其外围类对象之间有联系，那你可以将内部类声明为static。这通常称为嵌套类（nested class）。想要理解static应用于内部类时的含义，你就必须记住，普通的内部类对象隐含地保存了一个引用，指向创建它的外围类对象。然而，当内部类是static的时，就不是这样了。

- 要创建嵌套类的对象，并不需要其外围类的对象。（`Outer.Inner in = new Outer.Inner();`）
- 不能从嵌套类的对象中访问非静态的外围类对象。

由于静态内部类的加载机制,决定了他可以使用来处理**单例模式**。

嵌套类**可以有静态成员**。静态内部类**不能访问外部类的非静态成员**(包括非静态变量和非静态方法)。



#### 静态内部类和接口

静态内部类可以作为接口的一部分：正常情况下接口内部不可以有除了抽象方法以外的任何代码，但在JAVA 8以后，接口内部可以声明静态内部类的静态方法（可以有方法体）。

示例

```java
public interface Vehicle {
    // 定义一个静态内部类
    static class Car {
        private String color;
        
        public Car(String color) {
            this.color = color;
        }
        
        public void displayColor() {
            System.out.println("The color of the car is: " + color);
        }
    }
    
    // 定义一个静态方法，这在Java 8及以上版本的接口中是允许的
    static void displayMessage() {
        System.out.println("Vehicle interface message.");
    }
}

public class Test {
    public static void main(String[] args) {
        // 直接使用接口内部的静态类创建对象
        Vehicle.Car myCar = new Vehicle.Car("Red");
        myCar.displayColor();
        
        // 调用接口中的静态方法
        Vehicle.displayMessage();
    }
}
```



### 匿名内部类（一种特殊的局部内部类）

匿名内部类就是没有名字的内部类。匿名内部类可以很方便的定义**回调**。

#### 什么情况下需要使用匿名内部类？

如果满足下面的一些条件，使用匿名内部类是比较合适的：

- 只用到类的一个实例。
- 类在定义后马上用到。
- 类非常小（SUN推荐是在4行代码以下）
- 给类命名并不会导致你的代码更容易被理解。

在使用匿名内部类时，要记住以下几个原则：

- 匿名内部类一般不能有构造方法。
- 匿名内部类不能定义任何静态成员、方法和类。
- 匿名内部类不能是public,protected,private,static。
- 只能创建匿名内部类的一个实例。
- 一个匿名内部类一定是在new的后面，用其**隐含实现一个接口或实现一个类**。
- 因匿名内部类为局部内部类，所以局部内部类的所有限制都对其生效。



如果你有一个匿名内部类，它要使用一个在它的外部定义的对象，编译器会要求其参数引用是**final** 型的。
```java
public class Parcel8 {
	public static void main(String[] args) {
		Parcel8 p = new Parcel8();
		Destination d = p.dest("Tanzania");
	}

	// Argument must be final to use inside
	// anonymous inner class:
	public Destination dest(final String dest) {
		return new Destination() {
			private String label = dest;

			public String readLabel() {
				return label;
			}
		};
	}
}
```



**`JDK8`之后**允许声明是不显示标注final，但是它仍然是隐式的final。

```java
String name = "Dog";

name = "cat";

new Animal() {
    @Override
    public void yell() {
        System.out.println(name + ": bark! bark!");  
        // 报错。
        // Variable 'name' is accessed from within inner class, needs to be final or effectively final.
        // 将name = "cat"注释掉即可正常通过编译。
    }
}.yell();
```



#### 回调函数

匿名内部类非常适合使用于函数回调的情况。
```java
public class CallBack {

	public static void main(String[] args) {
		CallBack callBack = new CallBack();
		callBack.toDoSomethings(100, new CallBackInterface() {
			public void execute() {
				System.out.println("我的请求处理成功了");
			}
		});

	}

	public void toDoSomethings(int a, CallBackInterface callBackInterface) {
		long start = System.currentTimeMillis();
		if (a > 100) {
			callBackInterface.execute();
		} else {
			System.out.println("a < 100 不需要执行回调方法");
		}
		long end = System.currentTimeMillis();
		System.out.println("该接口回调时间 : " + (end - start));
	}
}
public interface CallBackInterface {
	void execute();
}
```







#### 构造器

匿名内部类是没有构造器的，不过可以手动做一个“假构造器”。
```java
// Animal
public interface Animal {
    void yell();
}

// AnimalFactory
public class AnimalFactory {
    String dogName = "Dog";
    String catName = "Cat";

    public Animal createDog() {
        return new Animal() {
            @Override
            public void yell() {
                System.out.println("I'm a " + dogName);
            }
        };
    }

    public Animal createCat() {
        return new Animal() {
            @Override
            public void yell() {
                System.out.println("I'm a " + catName);
            }
        };
    }
}

// Main
public class Main {
    public static void main(String[] args) {
        AnimalFactory factory = new AnimalFactory();
        Animal dog = factory.createDog();
        Animal cat = factory.createCat();

        dog.yell();
        cat.yell();
    }
}
```

打印结果
```
I'm a Dog
I'm a Cat
```



**注意**

```java
// 在AnimalFactory内部创建这个方法
public Animal createWhat(String what) {
        return new Animal() {
            @Override
            public void yell() {
                System.out.println("I'm " + what + "???");
            }
        };
    }

// Main
public class Main {
    public static void main(String[] args) {
        AnimalFactory factory = new AnimalFactory();

        String name = "haha";
        name = "what";

        Animal what = factory.createWhat(name);
    }
}
```

这里name不是final也不会报错，这是因为name并没有被匿名内部类直接使用，而是使用了`createWhat`的**形参name**，如果改成下面这样就会报错。
```java
public Animal createWhat(String what) {
        what = what.toLowerCase();   // what被修改，不是effectively final了
        return new Animal() {
            @Override
            public void yell() {
                System.out.println("I'm " + what + "???");  // 此行what报错
            }
        };
    }
```



## 从多层嵌套类中访问外部

一个内部类被嵌套多少层并不重要，它能透明地访问所有它所嵌入的外围类的所有成员

```java
class MNA {
	private void f() {
	}

	class A {
		private void g() {
		}

		public class B {
			void h() {
				g();
				f();
			}
		}
	}
}

public class MultiNestingAccess {
	public static void main(String[] args) {
		MNA mna = new MNA();
		MNA.A mnaa = mna.new A();
		MNA.A.B mnaab = mnaa.new B();
		mnaab.h();
	}
}
```

可以看到在`MNA.A.B`中，调用方法g()和f()不需要任何条件（即使它们被定义为private）。

## 内部类的重载问题

```java
class Egg {
	private Yolk y;


	public Egg() {
		System.out.println("New Egg()");
		y = new Yolk();
	}

	protected class Yolk {
		public Yolk() {
			System.out.println("Egg.Yolk()");
		}
	}
}


public class BigEgg extends Egg {
	public static void main(String[] args) {
		new BigEgg();
	}

	public class Yolk {
		public Yolk() {
			System.out.println("BigEgg.Yolk()");
		}
	}
}
```

输出结果:
`New Egg() Egg.Yolk()`

缺省的构造器是编译器自动生成的，这里是调用基类的缺省构造器。你可能认为既然创建了`BigEgg`的对象，那么所使用的应该是被“重载”过的Yolk，但你可以从输出中看到实际情况并不是这样的。

这个例子说明，当你继承了某个外围类的时候，内部类并没有发生什么特别神奇的变化。这两个内部类是完全独立的两个实体，各自在自己的命名空间内。

若**明确地继承某个内部类**
```java
// 父类
package com.example.demo1;

public class 李妃 {
    太子 son = new 太子();

    public 李妃() {
        System.out.println("我孩子要出生了");
    }

    public void 找孩子() {
        System.out.println("我的孩子呢？");
        son.yell();
    }

    protected class 太子 {
        public 太子() {
            System.out.println("我是太子！");
        }

        public void yell() {
            System.out.println("我真是太子！");
        }
    }

    public void 换太子(太子 newGuy) {
        son = newGuy;
    }
}


// 子类
package com.example.demo1;

public class 可怜李妃 extends 李妃 {
    public 可怜李妃() {
        换太子(new 狸猫());
    }

    protected class 狸猫 extends 太子 {
        public 狸猫() {
            System.out.println("我是狸猫！");
        }

        @Override
        public void yell() {
            System.out.println("你孩子没了");
        }
    }

    public static void main(String[] args) {
        可怜李妃 li = new 可怜李妃();
        li.找孩子();
        System.out.println(li.son);
    }
}
```

打印结果
```
我是太子！
我孩子要出生了
我是太子！
我是狸猫！
我的孩子呢？
你孩子没了
com.example.demo1.可怜李妃$狸猫@5acf9800
```

这里的`换太子()`方法通过`狸猫`对象向上转型让新版的外部类的成员变量换成了新的内部类。可以看到调用`找孩子()`时，`狸猫`的`yell()`被调用了。

## 内部类的继承问题

因为内部类的构造器要用到其外围类对象的引用，所以在你继承一个内部类的时候，事情变得有点复杂。问题在于，那个“秘密的”外围类对象的引用必须被初始化，而在被继承的类中并不存在要联接的缺省对象。要解决这个问题，需使用专门的语法来明确说清它们之间的关联：
```java
class WithInner {
	class Inner {
		Inner() {
			System.out.println("this is a constructor in WithInner.Inner");
		}

		;
	}
}


public class InheritInner extends WithInner.Inner {
	// ! InheritInner() {} // Won't compile
	InheritInner(WithInner wi) {
		wi.super();
		System.out.println("this is a constructor in InheritInner");
	}


	public static void main(String[] args) {
		WithInner wi = new WithInner();
		InheritInner ii = new InheritInner(wi);
	}
}
```

输出结果为：

> `this is a constructor in WithInner.Inner this is a constructor in InheritInner`

可以看到，`InheritInner` 只继承自内部类，而不是外围类。但是当要生成一个构造器时，缺省的构造器并不算好，而且你不能只是传递一个指向外围类对象的引用。此外，你必须在构造器内使用如下语法：

```java
enclosingClassReference.super();
```





## 为什么要使用内部类？为了解决什么问题？

1. 封装性
2. 逻辑分组
3. 实现多重继承的效果
4. 支持闭包
5. 匿名内部类可以很方便的定义回调



## 为什么`Builder`要设计成静态内部类

扯了这么远，回到一开始的问题，为什么Builder要设计成静态内部类？

1. 如果`Builder`设计为普通的成员内部类。那么在创建`Builder`时还需有一个外部类的实现。需要多构造一个，比较浪费。

   ```java
   Builder itemBuilder = new Item().new Builder();  // 这里创建了一个冗余的Item的实例
   Item item = itemBuilder.build(someField);        // 这里又创造了一个Item实例
   ```

2. 设计成静态内部类时，`Builder`无法访问外部类的私有成员，只能通过向它开放的方法（如构造函数）进行外部类的新建。封装性好。

# 参考文章

[一篇文章让你彻底了解Java内部类](https://github.com/Ccww-lx/JavaCommunity/blob/master/doc/javabase/%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E8%AE%A9%E4%BD%A0%E5%BD%BB%E5%BA%95%E4%BA%86%E8%A7%A3Java%E5%86%85%E9%83%A8%E7%B1%BB.md)







