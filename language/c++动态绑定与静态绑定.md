c++静态绑定与动态绑定的区别：基类指针访问子类实例，是在编译时确定（静态绑定）还是在运行时确定（动态绑定）。

# CRTP（Curiously Recurring Template Pattern）用于实现c++的静态绑定

CRTP + 类的继承 可以实现类的多态（使用基类的指针，访问子类的实例）。

CRTP把派生类作为基类的模板参数。

由于基类指针指向对象的类型在编译时就可以确定，所以这种方式称为静态绑定。

```cpp
template<class T>
class Base
{
	// methods within Base can use template to access member of Derived
	void interface() {
		static_cast<T*>(this)->implement();
	}
};

class Derived: public Base<Derived>
{
	void implement() {
		cout << "Derived implement." << endl;
	}
};
```

# 虚函数用于实现c++的动态绑定

虚函数 + 类的继承 可以实现类的多态（使用基类的指针，访问子类的实例）。

每次使用基类指针执行虚函数的时候，系统会检查基类指针实际指向的实例类型。然后调用对应类型的虚函数实现，这一步需要通过查询虚函数表来实现。

由于基类指针指向对象的类型在运行时候才确定，而不是在编译时就确定，所以这种方式称为动态绑定。

```cpp
class Base
{
	// methods within Base can be overrided by Derived
	void interface() {
		cout << "Base implement." << endl;
	}
};

class Derived: public Base
{
	void implement() {
		cout << "Derived implement." << endl;
	}
};
```
