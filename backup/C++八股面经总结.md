## C++基础概念



## 面向对象相关

### 1. 面向对象三大特性

- 封装，将数据（属性）和操作这些数据的函数（方法）组合在一个类中的过程。隐藏类的内部实现细节，通过控制类的访问级别，限制对类内数据的直接访问，提供了更好的安全性和可维护性。
- 继承，是一个派生类从一个基类哪里获取其属性和方法的过程。通过继承共享代码的类层次结构，减少重复代码，提高代码的复用性和可维护性。C++中继承也可以通过访问修饰符限制访问权限。
- 多态，见下题。

### 2. 介绍下多态

多态，同一个接口可以由不同的对象以不同的方式进行实现和响应的能力。

- 静态多态（编译时），函数重载、运算符重载，类模板、函数模板
- 动态多态（运行时），虚函数（动态绑定）

### 3. 虚函数的机制

- 利用虚函数，基类指针指向基类对象的时候使用基类的成员，指向派生类对象的时候就是用派生类的成员。
- 在类有虚函数的情况下，编译阶段会为类创建一个虚函数表（vtable，多个类共享，属于类的静态成员，在程序只读数据段）
- 每个类的实例有一个指针指向虚函数表（vptr），这个表存储了类的虚函数的地址，通过函数的索引访问对应的虚函数。
- 每个类的派生类继承了基类的虚函数表，并在派生类中拓展和重写虚函数，派生类的虚函数表包含基类的虚函数（根据 override 情况看是否替换），以及自己可能的新增的虚函数。


### 4. 什么函数（不）可以是虚函数

####  模板函数不能是虚函数

```C++
class Animal{
  public:
      template<typename T>
      virtual void make_sound(){
        //...
      }
};
```

会编译报错，模板函数的能被实例化成多少个函数需要所有的编译单元编译完成后才能知道，而虚函数表是在一个类编译完成时就确定的。

#### 构造函数不能是虚函数

构造函数是负责初始化类的对象，每个类都应该有自己的构造函数。在派生类构造时也会调用基类的构造函数用于初始化基类的成员。另外，从虚函数表的机制，在调用构造函数时，对象还没有完全创建和初始化，虚函数表可能尚未设置，在构造函数中使用虚函数表会导致未定义行为。

#### 基类析构函数需要是虚函数

实现多态时，基类指针可能访问的是派生类的对象，此时析构对象，如果基类的析构函数没有定义为虚函数，则只有基类的析构函数会被调用，导致派生类的资源可能不会被释放。如果基类析构函数是虚函数，则析构时会优先调用派生类的析构函数，再调用基类的析构函数，确保对象被正确销毁。

#### inline 函数可以是虚函数吗

虚函数可以是内联函数，但是虚函数表现多态性时不能内联。inline virtual 唯一可以内联的时候是编译器知道所调用的对象是哪个类，在编译期具有实际对象而不是对象的指针或引用时才能发生。

### 5. 纯虚函数

```C++
virtual int A() = 0;
```

纯虚函数是一种特殊的虚函数，在基类中不能对虚函数给出有意义的实现，留给派生类去实现，相当于一个接口。带纯虚函数的类叫做抽象类，不能直接生成对象，只能被继承并重写其虚函数后才能被使用。

- 如果类声明了虚函数，那么这个函数就是实现的了，哪怕是空实现。而纯虚函数只是个声明，需要派生类去实现。
- 虚函数在派生类中可以不重写（override），但纯虚函数必须在派生类中实现才能实例化派生类。
- 虚函数的类继承接口的同时也继承了父类的实现。

### 6. 操作符的重载以及哪些操作符不能重载

操作符重载是一种特殊的函数重载，通过重载操作符让自定义类型也能使用内置类型相似的语法和行为。

有些操作符是不能被重载的

- 成员选择操作符 `.`
- 展开操作符 `::`
- 条件运算发 `?:`
- sizeof
- typeid

### 7. 虚继承解决什么问题，原理是什么

多重继承的二义性是指当一个类从多个父类继承同名成员时，可能会产生歧义，编译器无法确定应该访问哪个成员。可以通过限定作用域符 `::` 指明特定父类成员，或者是在派生类中覆盖同名成员函数，并指明使用哪个父类实现。

当然还有一个解决方法就是虚继承。

```C++
    A
   / \
  B   C
   \ /
    D
class A { public: int value; };
class B : virtual public A {};
class C : virtual public A {}; 
class D : public B, public C {};
```

当一个派生类通过不同路径继承来自同一个基类时，使用虚继承可以确保只有一个基类子对象被创建。

虚继承的原理是通过虚基类表指针和虚基类表实现的，虚基类表存储的是虚基类与本类的偏移地址，通过偏移地址就可以找到虚基类成员。

### 8. 介绍下模板以及特化

泛型编程，通过模板来实现，分为函数模板（以及类中成员函数的模板）和类模板

模板特化是指在一定条件下重写模板的机制，分为全特化和偏特化（其中全局模板函数不支持偏特化）

#### 全特化

全特化是指对模板的某种特定类型的完全重新定义

```C++
template <>
class Stack<bool> {
private:
	std::vector<int> elements;
public:
	void push(const bool& element) {
		elements.push_back(element ? 1 : 0);
	}
	void pop();
	bool top() const {
		return elements.back() == 1;
	};
	bool empty() const {};
};
```

#### 偏特化

偏特化是指保留某些模板参数未特化，而对其他模板参数进行特化

偏特化可以分为

- 参数数量偏特化
- 范围偏特化（指针和const）

```C++
template <typename T1, typename T2>
class MyClass {
};

// 参数数量偏特化
template <typename T>
class MyClass<T, int> {
};

// 范围偏特化（指针和const）
template <typename T1, typename T2>
class MyClass<T1*, T2*> {
};

template <typename T1, typename T2>
class MyClass<T1 const, T2 const> {
};
```

函数模板不支持偏特化，如果强行写偏特化会重载

```C++
template<typename T1, typename T2>
void testfun(T1 t1, T2 t2) {
	cout << "模板函数" << endl;
}

// 全特化1， 不会被重载
template<>
void testfun(char t1, string t2) {
	cout << "全特化" << endl;
}

// 全特化2， 会被重载
template<>
void testfun(int t1, string t2) {
	cout << "全特化" << endl;
}

// 偏特化，重载
template<typename T2>
void testfun(int t1, T2 t2) {
	cout << "偏特化" << endl;
}

// 不支持const范围偏特化

// 指针偏特化
template<typename T1, typename T2>
void testfun(T1* t1, T2* t2) {
	cout << "指针偏特化" << endl;
}
```

#### 模板模板参数

```C++
template<typename T,
		template<typename T>
			class Container
		>
class YYY
{
private:
	Containter<T> c;
};

template<typename T>
using MyList = list<T, allocator<T>>;

YYY<string, list> mylist1; // error
YYY<string, MyList> myList2; // correct
```

#### 可变的模板参数

`typename... Types` 表示是一个 pack，可以通过递归调用

```C++
void print()
{
}

template<typename T, typename... Types>
void print(const T& firstArg, const Types&... args)
{
	cout << firstArg << endl;
	print(args...);
}
```

## 参考资料

1. https://csguide.cn/cpp/intro.html
2. https://github.com/UKMeng/CppInterview
