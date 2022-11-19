## 多态

多态分为两类：

- 静态多态：函数重载和运算符重载属于静态多态，主要是在复用函数名称
- 动态多态：派生类和虚函数实现运行时的多态

静态多态和动态多态的区别：

- 静态多态的函数地址早绑定 - 变异的时候确定函数地址
- 动态多态的函数地址晚绑定 - 运行阶段确认函数地址

下面一个例子说明了地址早绑定的概念，也就是说代码里已经指明了传递进来的是一个什么样的类，那么下游调用的时候就会去找这个类下面的对应方法

```c++
#include <iostream>
using namespace std;

class Animal {
public:
    void speak() {
        cout << "animal is speaking..." << endl;
    }
};

class Cat: public Animal {
public:
    void speak() {
        cout << "cat is speaking..." << endl;
    }
};
// 这种方法就是地址早绑定，在编译阶段就已经确定了函数的地址
// 相当于Animal &animal = cat;
void doSpeak(Animal &animal) {
    animal.speak();
}

void test01() {
    Cat cat;
    doSpeak(cat);
}

int main() {
    test01();
}
```

如果想去实现多态，也就是上面例子中调用Cat的speak方法，就需要实现地址晚绑定，在运行的时候确定函数地址。

实现1：使用虚函数实现地址晚绑定，在父类的函数前添加virtual关键字

```c++
#include <iostream>
using namespace std;

class Animal {
public:
    // virtual是指定一个虚函数
    virtual void speak() {
        cout << "animal is speaking..." << endl;
    }
};

class Cat: public Animal {
public:
    void speak() {
        cout << "cat is speaking..." << endl;
    }
};
// 这种方法就是地址早绑定，在编译阶段就已经确定了函数的地址
// 相当于Animal &animal = cat;
void doSpeak(Animal &animal) {
    animal.speak();
}

void test01() {
    Cat cat;
    doSpeak(cat);
}

int main() {
    test01();
}
```

动态多态的满足条件：

- 有继承的关系。
- 子类要重写父类的**虚函数**（virtual修饰的方法）。重载和重载的区别：重载的函数名同但是参数不同，重写是所有都相同，包括参数列表、函数名。子类可以不需要加virtual

动态多态的使用：

- 父类的**指针或者引用**执行子类对象

### 多态的原理剖析

首先，我们看一下上面实现虚函数对类的大小的影响

```c++
#include <iostream>
using namespace std;

class Animal {
public:
    // virtual是指定一个虚函数
    virtual void speak() {
        cout << "animal is speaking..." << endl;
    }
};

class Cat: public Animal {
public:
    void speak() {
        cout << "cat is speaking..." << endl;
    }
};

void doSpeak(Animal &animal) {
    animal.speak();
}

void test01() {
    // 在不实现需函数的情况下，一个空的类的大小为1,
    // 实现了虚函数之后其大小变为了4
    cout << "size of Animal: " << sizeof(Animal) << endl;
}

int main() {
    test01();
}
```

这里我们可以看到函数的大小从1变成了4

这里占用了4个字节的就是指向这个函数的指针。这个指针是指向虚函数表的(vftable - virtual function table)一个虚函数表指针(vfptr - virtual funcation pointer)

虚函数表内记录了虚函数的地址。

![image-20221119184417890](D:\01Learn_cpp\LearnCPP\images\C++黑马视频教学\image-20221119184417890.png)

在子类不进行函数重写的时候，子类继承的过程中会将这个虚函数指针也记录下来

但是当子类重写了父类中的虚函数，子类中的虚函数表内部会替换成子类的虚函数地址。当父类的指针或引用指向对象的时候就发生了多态

```c++
Animal &animal = cat;
animal.speak; 
```

上面这种形式也就走到了Cat类中重写的对应虚函数

![image-20221119184958641](D:\01Learn_cpp\LearnCPP\images\C++黑马视频教学\image-20221119184958641.png)

![image-20221119185321179](D:\01Learn_cpp\LearnCPP\images\C++黑马视频教学\image-20221119185321179.png)

重写前：

![image-20221119185624269](D:\01Learn_cpp\LearnCPP\images\C++黑马视频教学\image-20221119185624269.png)

重写后：

![image-20221119185644457](D:\01Learn_cpp\LearnCPP\images\C++黑马视频教学\image-20221119185644457.png)

 ### 多态的优势

- 代码组织结构清晰
- 可读性强
- 利于前期和后期的扩展以及维护

案例：分别使用普通写法和多态写法实现计算器

```c++
// 使用传统方法实现计算器
// 不符合开闭原则
class Calculator {
public:
    int m_Num1;
    int m_Num2;

    int getResult(string oper) {
        // 这种场景，只要扩展新功能就需要添加else if分支
        // 这样是违反开闭原则的（对扩展开放，对修改关闭）
        if (oper == "+") {
            return m_Num1 + m_Num2;
        } else if (oper == "-") {
            return m_Num1 - m_Num2;
        } else if (oper == "*") {
            return m_Num1 * m_Num2;
        } else if (oper == "/") {
            return m_Num1 / m_Num2;
        }
    }
};
```

```c++
#include <iostream>
using namespace std;

class Calculator {
public:
    int m_Num1;
    int m_Num2;

    int getResult(string oper) {
        // 这种场景，只要扩展新功能就需要添加else if分支
        // 这样是违反开闭原则的（对扩展开放，对修改关闭）
        if (oper == "+") {
            return m_Num1 + m_Num2;
        } else if (oper == "-") {
            return m_Num1 - m_Num2;
        } else if (oper == "*") {
            return m_Num1 * m_Num2;
        } else if (oper == "/") {
            return m_Num1 / m_Num2;
        }
    }
};

// 利用多态使用计算器

// 实现计算器抽象类
class AbstractCalculator {
public:
    int m_Num1;
    int m_Num2;
    virtual int getResult()
    {
        return 0;
    }
};

// 加法计算器
class AddCalculator: public AbstractCalculator {
public:
    int getResult() {
        return m_Num1 + m_Num2;
    }
};

// 减法计算器
class SubstractCalculator: public AbstractCalculator {
public:
    int getResult() {
        return m_Num1 - m_Num2;
    }
};

// 乘法计算器
class MultiplyCalculator: public AbstractCalculator {
public:
    int getResult() {
        return m_Num1 * m_Num2;
    }
};

// 除法计算器
class DivideCalculator: public AbstractCalculator {
public:
    int getResult() {
        return m_Num1 / m_Num2;
    }
};

void test01() {
    // 多态的条件
    // 父类的指针或引用指向子类对象

    // 加法计算
    AbstractCalculator *abc = new AddCalculator; // 创建加法计算器对象，用父类指针指向他
    abc->m_Num1 = 10;
    abc->m_Num2 = 20;

    cout << abc->m_Num1 << " + " << abc->m_Num2 << " = " << abc->getResult() << endl;
    // new放在堆区，需要手动释放
    delete abc;

    // 减法计算
    // 注意这里已经命名过了，所以只需要指针重新指向就可以了，不需要重新命名
    abc = new SubstractCalculator;
    abc->m_Num1 = 10;
    abc->m_Num2 = 20;

    cout << abc->m_Num1 << " + " << abc->m_Num2 << " = " << abc->getResult() << endl;
    // new放在堆区，需要手动释放
    delete abc;

    // 乘法计算
    abc = new MultiplyCalculator;
    abc->m_Num1 = 10;
    abc->m_Num2 = 20;

    cout << abc->m_Num1 << " + " << abc->m_Num2 << " = " << abc->getResult() << endl;
    // new放在堆区，需要手动释放
    delete abc;

    // 除法计算
    abc = new DivideCalculator;
    abc->m_Num1 = 10;
    abc->m_Num2 = 20;

    cout << abc->m_Num1 << " + " << abc->m_Num2 << " = " << abc->getResult() << endl;
    // new放在堆区，需要手动释放
    delete abc;
}

int main() {
    test01();
}
```

使用多态后，可以很清楚看到抽象类对接口的定义，结构更好。如果有新增部分出错，组织结构更清晰方便快速定位。比如出现了乘法错误了，那么我只需要看乘法的部分就可以了

对于后期新增扩展和维护，如果需要新增一个功能，只需要新增类即可，不需要修改原有类的方法。

### 纯虚函数和抽象类

在多态中，通常父类中虚函数的实现是没有意义的，主要都是调用子类重写的内容

因此可以将虚函数改为**纯虚函数**

纯虚函数的语法：`virtual 返回值类型 函数名 (参数列表) = 0;`

当类中有了纯虚函数，这个类就称为**抽象类**

#### 抽象类的特点

- 无法实例化对象
- 子类必须重写抽象类中的纯虚函数，否则也属于抽象类

总结下也就是说上面的抽象函数只是引入了抽象函数列表和抽象函数指针。这里引入的纯抽象类才是python中meta=abc.Abstract以及@abstractmethod类似的操作

```c++
// 报错场景演示

#include <iostream>
using namespace std;

class Base {
public:
    // 有了一个纯虚函数，因此变成了抽象类
    virtual void func() = 0; // 定义纯虚函数

};

class PublicChild: public Base {
};

void test01() {
    // // 这两个都会因为实例化抽象类而报错
    // Base base1;
    // new Base;

    // 子类没有重定义纯虚函数，因此也无法进行正常的实例化
    PublicChild public_child1;
    new PublicChild;
}

int main() {
    test01();
}
```

```c++
// 正常场景
#include <iostream>
using namespace std;

class Base {
public:
    // 有了一个纯虚函数，因此变成了抽象类
    virtual void func() = 0; // 定义纯虚函数

};

class PublicChild: public Base {
public:
    // 重写纯虚函数之后，可以正常实例化
    void func() {
        cout << "func() is called.." << endl;
    }
};

void test01() {
    PublicChild *public_child = new PublicChild;
    public_child->func();
}

int main() {
    test01();
}
```

### 多态案例2 -- 制作饮品

案例描述：

制作饮品的大致流程为：煮水-冲泡-倒入杯中-加入辅料

```c++
/*
制作饮品的案例
*/

#include <iostream>
using namespace std;

class AbstractDrinking {
public:
    virtual void Boil() = 0;        // 煮水
    virtual void Brew() = 0;        // 冲泡
    virtual void PourInCup() = 0;   // 倒入杯中
    virtual void PutSomething() = 0;  // 加入辅料
    void makeDrink() {
        Boil();
        Brew();
        PourInCup();
        PutSomething();
    }
};

class Coffee: public AbstractDrinking {
    virtual void Boil() {
        cout << "煮星巴克的自来水" << endl;
    };
    virtual void Brew() {
        cout << "冲泡咖啡" << endl;
    };
    virtual void PourInCup() {
        cout << "倒入咖啡杯" << endl;
    };
    virtual void PutSomething() {
        cout << "加入糖和牛奶" << endl;
    };
};

class Tea: public AbstractDrinking {
    virtual void Boil() {
        cout << "煮小罐茶大师的洗脚水" << endl;
    };
    virtual void Brew() {
        cout << "冲泡茶叶" << endl;
    };
    virtual void PourInCup() {
        cout << "倒入搪瓷杯" << endl;
    };
    virtual void PutSomething() {
        cout << "加入枸杞" << endl;
    };
};

void doWork(AbstractDrinking *abs) {
    abs->makeDrink();
    delete abs; // 注意堆区数据手动开辟手动释放
}

void test01() {
    // 制作咖啡
    doWork(new Coffee);
    cout << "-----------------" << endl;
    // 制作茶
    doWork(new Tea);
}

int main() {
    test01();
}
```

### 虚析构和纯虚析构

多态在使用的时候，如果子类有属性开辟到了堆区，那么父类指针在释放的时候是无法对子类在堆区的数据进行析构delete对应堆区数据的

解决方法：将父类中的析构函数改为**虚析构**或者纯虚析构

虚析构和纯虚析构的区别：如果是纯虚析构，那么该类属于抽象类，无法实例化

虚析构语法：

`virtual ~类名() {};`

纯虚析构语法：

`virtual ~类名() {} = 0;`

#### 虚析构

```c++
// 虚析构和纯虚析构
#include <iostream>
#include <string>
using namespace std;

class Animal {
public:
    virtual void speak() = 0;

    Animal() {
        cout << "Animal's strucated..." << endl;
    }

    ~Animal() {
        cout << "Animal's destrucated..." << endl;
        }
};

class Cat: public Animal {
public:
    string *m_Name;
    Cat(string name) {
        cout << "Cat's strucated..." << endl;
        m_Name = new string(name); // 这里有一个堆区的属性，应在Cat的析构函数中对它释放
    }

    ~Cat() {
        if (m_Name != NULL) {
            cout << "Cat's destrucated..." << endl;
            delete m_Name;
            m_Name = NULL;
        }
    }
    virtual void speak() {
        cout << "cat " << *m_Name << " is speaking..." << endl;
    }
};

void test01() {
    Animal *animal = new Cat("Tom");
    animal->speak();
    delete animal;
}

int main() {
    test01();
}
```

上面代码的打印结果是

```
Animal's strucated...
Cat's strucated...
cat Tom is speaking...
Animal's destrucated...
```

这里可以看到Cat本身的析构函数是没有被调用到的。这里应该先析构子类，后析构父类才对。这里也就是说我们子类的指针没有释放掉，导致了内存泄漏。

父类指针在析构的时候不会调用到子类析构函数，这也就导致了子类如果有堆区数据，就会初夏内存泄漏问题。

这个时候将父类的析构函数转化为虚析构，就正常调用了

```c++
// 虚析构和纯虚析构
#include <iostream>
#include <string>
using namespace std;

class Animal {
public:
    virtual void speak() = 0;

    Animal() {
        cout << "Animal's strucated..." << endl;
    }

    virtual ~Animal() {
        cout << "Animal's destrucated..." << endl;
        }
};

class Cat: public Animal {
public:
    string *m_Name;
    Cat(string name) {
        cout << "Cat's strucated..." << endl;
        m_Name = new string(name); // 这里有一个堆区的属性，应在Cat的析构函数中对它释放
    }

    ~Cat() {
        if (m_Name != NULL) {
            cout << "Cat's destrucated..." << endl;
            delete m_Name;
            m_Name = NULL;
        }
    }
    virtual void speak() {
        cout << "cat " << *m_Name << " is speaking..." << endl;
    }
};

void test01() {
    Animal *animal = new Cat("Tom");
    animal->speak();
    delete animal;
}

int main() {
    test01();
}
```

这个时候打印结果就变成了期待的

```
Animal's strucated...
Cat's strucated...
cat Tom is speaking...
Cat's destrucated...
Animal's destrucated...
```

#### 纯虚析构

析构函数就算是转化到了纯虚函数，也会在析构的时候被调用到。因此纯虚析构也是需要代码实现的。

因此他的声明和实现都要做。

```c++
// 虚析构和纯虚析构
#include <iostream>
#include <string>
using namespace std;

class Animal {
public:
    virtual void speak() = 0;

    Animal() {
        cout << "Animal's strucated..." << endl;
    }

    virtual ~Animal() = 0;
};

Animal::~Animal(){
    cout << "Animal's destrucated by pure abstract destructor..." << endl;
}

class Cat: public Animal {
public:
    string *m_Name;
    Cat(string name) {
        cout << "Cat's strucated..." << endl;
        m_Name = new string(name); // 这里有一个堆区的属性，应在Cat的析构函数中对它释放
    }

    ~Cat() {
        if (m_Name != NULL) {
            cout << "Cat's destrucated..." << endl;
            delete m_Name;
            m_Name = NULL;
        }
    }
    virtual void speak() {
        cout << "cat " << *m_Name << " is speaking..." << endl;
    }
};

void test01() {
    Animal *animal = new Cat("Tom");
    animal->speak();
    delete animal;
}

int main() {
    test01();
}
```

总结：

- 虚析构或者纯虚析构都是用来解决通过父类指针释放子类对象
- 如果子类中没有堆区数据，可以不写虚析构或者纯虚析构
- 拥有纯虚析构函数的类也属于抽象类，需要在子类重写

