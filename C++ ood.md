## C++ 复健

### function pointer

程序运行期间，每个函数都会占用一段连续的内存空间。

函数指针： 指向函数的指针

#### case1

```c++
int (*pf)(int , char);
```



#### case2

可以用一个原型匹配的函数名字给一个函数指针赋值

```c++
void PrintMin(int a, int b) {
    if(a < b) printf("%d", a);
    else pritnf("%d", b);
}
int main() {
    void(* pf)(int, int);
    int x = 4, y =5;
    pf = PrintMin;	// 函数指针的赋值
    pf(x,y);
    return 0;
}
```



#### case3

qsort可以用函数指针对任意类型数组进行排序

base: 数组起始位置

nelem: 数组元素个数

width: 每个元素的大小 

pfCompare: 排序规则

```c++
void qsort(void * base, int nelem, unsigned int width, int(* pfCompare)(const void*, const void*));
```



```c++
int MyCompare(const void * elem1, const void * elem2) {
    unsigned int *p1, *p2;
    p1 = (unsigned int *) elem1;
    p2 = (unsigned int *) elem2;
    return (*p1 % 10) - (*p2 % 10);
}

#define NUM 5
int main() {
    unsigned int an[NUM] = {8, 123, 11, 11, 10, 4};
    qsort(an, NUM, sizeof(unsigned int), MyCompare);	// 注意：函数指针可以用函数名赋值
    return 0;
}
```





### inline

函数调用是有时间开销的。编译器处理对内联函数的调用语句是，是将整个函数的代码插入到调用语句处。（调用函数需要用到栈）

把简单函数写成`inline`

```c++
inline int Max(int a, int b) {
    if(a > b) {
        return a;
    }
    return b;
}

int k = Max(n1, n2);
// inline will convert to
/*
if(n1 > n2) {
	tmp = n1;
}
else {
	tmp = n2;
}
k = tmp;
*/
```





### explicit

no type conversion in construction



### reference &

#### case1

```c++
double a = 4, b= 5;
double & r1 = a;
double & r2 = r1;
r2 = 10;
cout << a << endl;	// output: 10
r1 = b;
cout << a << endl;	// output: 5
```



#### case 2

```c++
int n = 4;
int & SetValue() { return n; }
int main()
{
	SetValue() = 40; // <==> n = 40;
	cout << n; // output: 40
	return 0;
}
```



#### case3

```c++
int n = 100;
const int & r = n;	// r type is const int &
// const int & r = (int) n;		ok
// int & n = (const int (&)) r;		compile error
r = 200;	// compile error cannot modify const
n = 300;	// that's ok
```





### const

#### case1

```c++
const int MAX_VAL = 23;		// define a constant
// better to use const than define in c++ because we can check the const type of variable 
```



#### case2

```c++
int n; int * p2;
const int * p = & n; 	// define a const ptr cannot modify the value it points to 
* p = 5;	// compile error
n = 4;		// ok
p = & m:	// ok, we did not change the value it points to
p = p2;		// ok
p2 = p;		// compile error

// can be used to protect the value in a function
void MyPrintf( const char * p) {
    strcpy(p, "this");	// compile error
    printf("%s", p);	// ok
}
```



#### case3

```c++
int n = 100;
const int & r = n;	// r type is const int &
// const int & r = (int) n;		ok
// int & n = (const int (&)) r;		compile error
r = 200;	// compile error cannot modify const
n = 300;	// that's ok
```





### new

#### case1

```c++
int * pn;
pn = new int;	// assign a variable
* pn = 5;	
```



#### case2

```c++
int * pn;
int i = 5;
pn = new int[i * 20];	// assign an array
pn[0] = 20;	

new T;		// return T *
new T[n]; 	// return T *
```



### delete

#### case1

```c++
int * p = new int;
* p = 5;
delete p;
delete p;	// compile error
```



#### case2

```c++
int * p = new int[20];
p[0] = 1;
delete [] p;
// delete p is ok but space are still used
```







### constructor

对象生成时构造函数自动被调用。对象一旦生成就再也不能在其上执行构造函数。

无参构造函数不一定存在。

```c++
class Test {
	public:
		Test(int n) {};
		Test(int n , int m) {};
		Test() {};
};
Test array1[3] = {1, Test(1,2)};	// 三个元素分别用三个不同的constructor初始化
```





### copy constructor

#### case1

```c++
class Complex {
	double real, imag;
};
Complex c1;			//调用缺省无参构造函数
Complex c2(c1);		//调用缺省的复制构造函数，将c2初始化为c1
```



#### case2

```c++
class Complex {
public:
	double real, imag;
	Complex() {}
	Complex( const Complex & c) {
		real = c.real;
		iimag = c.imag;
	}
};
Complex c1;			
Complex c2(c1);		
```



#### case3

```c++
class CSample {
	CSample( CSample c ) {}; 	// error, the parameter must be a reference
};
```



#### case4

```c++
Complex c2(c1);		
Complex c2 = c1;	// equal to the upper one
```



#### case5

```c++
class A {
public:
	A() {};
	A( A & a ) {
		cout << "Copy constructor called" << endl;
	}
};

void Func(A a1) {}		// when we call Func, a1 needs to be init, use copy constructor to construct it (按值传递)
int main() {
	A a2;
    Func(a2);			// a2 is the parameter of the copy constructor to init a1
    return 0;
}

// output: Copy constructor called
```

Using copy constructor costs. Hence, we should use reference to be the parameter. And we want to not to change the reference, we can use const.



#### case6

```c++
class A {
public:
	int v;
	A(int n) { v = n; }
	A(const A & a) {
		v = a.v;
		cout << "Copy constructor called" << endl;
	}
};
A Func() {
    A b(4);
    return b;	// use copy constructor to copy b to return
}
int main() {
    cout << Func().v << endl;
    return 0;
}
//output: copy constructor called\n 4
```





### conversion constructor

convert one Type to the class 

#### case1

```c++
class Complex {
public:
    double real, imag;
    // conversion constructor
    Complex(int i) {
        cout << "IntConstructor called" << endl;
        real = i;
        imag = 0;
    }
    // constructor
    Complex(double r, double i) {
        real = r;
        imag = i;
    }
};

int main()
{
    Complex c1(7, 8);
    Complex c2 = 12;	// conversion constructor
    c1 = 9;		// 9 is automatically a temp Complex by conversion constructor
    cout << c1.real << "," << c1.imag << endl;
    return 0;
}
```





### destructor

One class has only one destructor.

One class can have many constructors. 

Default destructor does nothing.



#### case1

```c++
class String {
private:
	char * p;
public:
	String() {
		p = new char[10];
	}
	~String();
}
String::~String()
{
    delete []p;
}
```



#### case2

```c++
class Ctest {
public:
    ~Ctest() {
		cout << "destructor called" << endl;
    }
};

int main() 
{
    Ctest array[2];
    cout << "End Main" << endl;
    return 0;
}

/*
output:
End Main
destructor called
destructor called
*/
```



#### case3

```c++
Ctest * pTest;
pTest = new Ctest[3];
delete [] pTest;
```



#### case4

```c++
class CMyclass {
public:
    ~CMyclass() {
        cout << "destructor" << endl;
    }
};
CMyclass obj;
CMyclass fun(CMyclass sobj) {
	return sobj;
}
int main() {
	obj = fun(obj);
    // value parameter destruct
    // fun(obj) returns a temporary object destruct
    // obj destruct at the end
    return 0;
}

/*
output:
destructor
destructor
destructor
*/
```



#### case5

```c++
class Demo {
public:
    Demo(int i) {
		id = i;
        cout << "id=" << id << " constructed" << endl;
    }
    ~Demo() {
		cout << "id=" << id << " destructed" << endl;
    }
};

Demo d1(1);
void Func() {
    static Demo d2(2);		// static would not destruct until the programme ends
    Demo d3(3);
    cout << "func" << endl;
}

int main() {
	Demo d4(4);
    d4 = 6;
    cout << "main" << endl;
    {
     	Demo d5(5);		// local variable   
    }
    Func();
    cout << "main ends" << endl;
    return 0;
}

/*
output:
id=1 constructed
id=4 constructed
id=6 constructed
id=6 destructed	// temporaty object
main
id=5 constructed
id=5 destructed
id=2 constructed
id=3 constructed
func
id=3 destructed
main ends
id=6 destructed
id=2 destructed
id=1 destructed
*/
```



#### case6

```c++
int main() {
	A * p = new A[2]
    A * p2 = new A;
    A a;
    delete [] p;
}

/*
destructor used 3 times
p2 does not destruct
*/
```





### this

#### case1

早期c++要先翻译成c才能编译

```c++
class CCar {
public:
	int price;
    void SetPrice(int p);
};
void CCar::SetPrice(int p) {
    price = p;
}
int main()
{
    CCar car;
    car.SetPrice(20000);
    return 0;
}
```



translate->

```c
struct CCar {
	int price;
};

// C does not have member function
// has to define in global
void SetPrice(struct CCar * this, int p) {
	this->price = p;
}

int main()
{
    struct CCar car;
    SetPrice(&car, 20000);
    return 0;
}
```



#### case2

```c++
class Complex {
public:
    double real, imag;
    void Print() {
		cout << real << "," << imag;
    }
    Complex(double r, double i): real(r), imag(i) {}
    Complex AddOne() {
		this->real ++;	// equals to ++real
        this->Print();	// equals to Print()
        return *this;
    }
}

int main()
{
	Complex c1(1, 1), c2(0, 0);
    c2 = c1.AddOne();
    return 0;
}

/*
output
2,1
*/
```



#### case3

```c++
class A {
	int i;
public:
    void Hello() {cout << "hello" << endl;} // actually it has a this ptr as a parameter
};
int main()
{
	A *p = NULL;
    p->Hello();	// Can compile?
}
```



translate ->

```c++
class A {
	int i;
};
void Hello(A * this) {cout << "hello" << endl}
int main()
{
    A *p = NULL;
    Hello(p);
}

/* can compile
output:
hello
*/
```



#### case4

```c++
class A {
	int i;
public:
    void Hello() {cout << i << "hello" << endl;}
};
int main()
{
	A *p = NULL;
    p->Hello();	// compile error! `this` is a nullptr, does not have this->i
}
```



static member function could not use this ptr!

Because statistic member function does not impact specific objects.





### static

#### case1

```c++
class CRectangle {
private:
	int w, h;
    static int nTotalArea;		// static member variable
    static int nTotalNumber;
public:
    CRectangle(int w_, int h_);
    ~CRectange();
    static void PrintTotal();	// static member function
};
```

every object has their own non-static member variable, but all objects share one static member variable.

static member variable is a global member

sizeof would not compute the static member variable.

non-static member function must function on a specific object, but static member function  does not need to function on a specific object.

static member can be visited without object.

static member function is a global function



#### case2

```c++
// 1
CRectangle::PrintTotal();
// 2
CRectangle r;  r.PrintTotal();
// 3
CRectangle *p = &r; p->PrintTotal()
```



#### case3

global variable to record counts and sum of areas vs static member variable

static member variable encapsuled into the class, easy to maintain

```c++
CRectangle::CRectangle(int w_, int h_) {
	w = w_;
    h = h_;
    nTotalNumber ++;
    nTotalArea += w*h;
}
CRectangle::~CRectangle() {
	nTotalNumber --;
    nTotalArea -= w * h;
}
void CRectangle::PrintTotal() {
	cout << nTotalNumber << "," << nTotalArea << endl;
}

// must init the static member variable in the class file
int CRectangle::nTotalNumber = 0;
int CRectangle::nTotalArea = 0;

int main()
{
	CRectangle r1(3,3), r2(2,2);
    // cout << CRectangle::nTotalNumber;	Wrongprivate
}

/*
output
2,13
2,13
*/
```



#### case4

static member function could not visit non-static member variable, and it cannot use non-static member function

```c++
void CRectangle::PrintTotal() {
	cout << w << "," << nTotalNumber << "," << nTotalArea << endl;	// wrong
}
```



#### case5

missing copy constructor, making `nTotalNumber` not correct

```c++
CRectangle::CRectangle(CRectangle & r) {
	w = r.w;
	h = r.h;
	nTotalNumber ++;
	nTotalArea += w * h;
}
```



### member class and enclosing

#### case1

```c++
class CTyre {
private:
    int radius;
    int width;
public:
    // list-initializaion since c++11
    CTyre(int r, int w): radius(r), width(w) {}
    
};

class CEngine {

};

class CCar {
private:
	int price;
    CTyre tyre;			// member class
    CEngine engine;		// member class
public:
    CCar(int p, int tr, int tw);
};

CCar::CCar(int p, int tr, int w): price(p), tyre(tr, w) {};	// important

int main() 
{
	CCar car(20000, 17, 225);
    return 0;
}
```

If we do not implement the construction of CCar, `CCar car;` would compile error, since the compiler does not know how to initialize `car.tyre`. `car.tyre` does not have non parameter construction.

CCar is an enclosing class since it has member class `car.tyre` and `car.engine`. When the CCar initialize, first use construction of `CTyre` and `CEngine`. Then use construction of `CCar`. When the CCar destruct, first use destruction of `CCar`, and then use `CTyre` and `CEngine`.



#### case2

```c++
class A {
public:
    A() {cout << "default" << endl;}
    A(A & a) {cout << "copy" << endl;}
};

class B {A a;};

int main()
{
	B b1, b2(b1);
    return 0;
}

/*
output:
default
copy
*/
```



### const

#### case1

```c++
class Demo {
	...
};
const Demo obj;	// obj cannot be altered
```



#### case2

```c++
class Sample {
public:
    int value;
    void GetValue() const;
    void func() {};
    Sample() {};
};

void Sample::GetValue() const
{
    value = 0	// error! const member function should not change the value
    func();		// error! const member function should not use non const function
}

int main()
{
	const Sample o;
    o.value = 100;	//error! const obj cannot be altered
    o.func();		// error! const obj cannot use non const member function
    o.GetValue();	// ok! can use const function
    return 0;
}

```

PS.

1. `const` can use `static`
2. two member function, name and parameters are the same, but one has const and one does not. `const` one is overload.



#### case3

```c++
class CTest {
private:
    int n;
public:
    CTest() {n = 1};
    int GetValue() const {return n};
    int GetValue() {return 2 * n};
};

int main()
{
	const CTest obj1;
    CTest obj2;
    cout<<obj1.GetValue()<<","<<obj2.GetValue();
    return 0;
}

/*
output
1,2
*/
```





### friends

#### case1

```c++
class CCar;
class CDriver {
public:
    void ModifyCar(CCar * pCar);
    // if not use ptr, need to implement construction of CCar first
};

class CCar {
private:
    int price;
   	friend int MostExpensiveCar(CCar cars[], int total);	// declare friend global function
    friend void CDriver::ModifyCar(CCar *pCar);		// declare friend class member function
}

void CDriver::Modify(CCar * pCar) {
	pCar->price += 1000;
    // since friend can modify private
}

int MostExpensiveCar(CCar cars[], int total) {
	int tmpMax = -1;
    for(int i = 0; i < total; ++i) {
		if(cars[i].price > tmpMax) {
			tmpMax = cars[i].price;
        }
    }
    return tmpMax;
} 
```



#### case2

```c++
class CCar {
private:
    int price;
    friend class CDriver;
};

class CDriver {
public:
    CCar myCar;
    void ModifyCar() {
		myCar.price += 1000;
    }
};
```

友元类之间不能传递，继承



### operator overload

#### case1

```c++
class Complex {
public:
    double real, imag;
    Complex(double r = 0, double i = 0):real(r),imag(i) {}
    Complex operator-(const Complex & c);
};

// overload global function has 运算符目数 parameters
Complex operator+(const Complex & a, const Complex & b) {
	return Complex(a.real+b.real, a.imag+b.imag);
}

// overload member function has 运算符目数-1 parameters
Complex Complex::operator-(const Complex & c) {
    return Complex(real-c.real, imag-c.imag);
}

int main()
{
	Complex a(4,4), b(1,1), c;
    c = a + b;	// equals to c = operator+(a,b);
    c = a - b;	// equals to c = a.operator-(b);
    return 0;
}
```



#### case2

```c++
class Complex {
private:
    double real, imag;
public:
    Complex(double r, double i): real(r), imag(i){};
    Complex operator+(double r);
};
Complex Complex::operator+(double r) {
	return Complex(real+r, imag);
}

Complex c;
c = c + 5;		// ok
c = 5 + c;		// compile error
```

In order to use `c = 5 + c`, need to overload global function.



#### case3

```c++
class Complex {
	double real, imag;
public:
    Complex(double r, double i):real(r),imag(i){};
    Complex operator+(double r);
    friend Complex operator+(double r, const Complex & c);	// c.real and c.imag are private
}

Complex operator+(double r, const Complex & c) {
    return Complex(c.real + r, c.imag);		
}
```







### = overload

#### case1

```c++
class String {
private:
    char * str;
public:
    String(): str(new char[1]) {str[0] = 0;}
    const char * c_str() {return str;}
    String & operator = (const char * s);
    string::~String() {delete[] str;}
}

String & String::operator=(const char * s) {
    delete[] str;
    str = new char[strlen(s)+1];
    strcpy(str, s);
    return *this;
}

int main()
{
	String s;
    s = "Good Luck,";	// equals to s.operator=("Good Luck,");
    cout << s.c_str()<<endl;
    String s2 = "hello!";	// compile error! construction with parameter char* does not exist
    s = "Shenzhou 8!";
    cout << s.c_str()<<endl;
	return 0;
}
```



#### case2

```c++
class String {
private:
    char * str;
public:
    String(): str(new char[1]) {str[0] = 0;}
    const char * c_str() {return str;}
    String & operator = (const char * s) ;
    string::~String() {delete[] str;}
}

String & String::operator=(const char * s) {
    delete[] str;
    str = new char[strlen(s)+1];
    strcpy(str, s);
    return *this;
}

int main()
{
	S1 = "this";
    S2 = "that";
    // default = operator function is a deep copy
    // if we do not overload, S1.str would points to where S2.str points to.
    // may delete S2.str twice compile error
    // change S1 would also change S2
    // hence = overload is important
    S1 = S2;
}
```





#### case3

```c++
String s;
s = "Hello";
s = s;		// compile error! we first delete s, and not new for deleted s

// hence = overload needs to change
String & String::operator=(const char * s) {
    if(this == &s) {
        return * this;
    }
    delete[] str;
    str = new char[strlen(s)+1];
    strcpy(str, s);
    return *this;
}
```



#### case4

```c++
// void String::operator=(const char * s) ?
// String String::operator=(const char * s) ?
a = b = c;		// a.operator=(b.operator=(c))

// if we use String not String &, a would not change
(a = b) = c;	// (a.operator=(b)).operator=(c);
```



####  case5

copy constructor is the same as = overloaded operator. default copy constructor is deep copy. we need to use shallow copy

```c++
String(String & s) {
	str = new char[strlen(s.str)+1];
    strcpy(str, s.str);
}
```





### >> << overload

#### case1

```c++
class CStudent {
    public:
    int nAge;
};


// we cannnot write a member function
// have to use global function
ostream & operator<<(ostream & o, const CStudent & s) {
    o << s.nAge;
    return o;	// need to return ostream to continue to print
}

int main()
{
	CStudent s;
    s.nAge = 5;
    cout<<s<<endl;		// want to overload can output s.nAge
    return 0;
}
```



#### case2

```c++
class Complex {
private:
    double real;
    double imag;
public:
    Complex(double r, double i):real(r),imag(i) {};
    friend ostream & operator<<(ostream & o, const Complex & c);
    friend istream & operator>>(istream & i, const Complex & c);
};

ostream & operator<<(ostream & o, const Complex & c) {
    o << c.real << "+" << c.imag << "i";
    return o;
}

istream & operator>>(istream & i, const Complex & c) {
	string s;
    i >> s;
    int pos = s.find("+", 0);
    c.real = stof(s.substr(0, pos));
    c.imag = stof(s.substr(pos+1, s.size()-2-pos));
    return o;
}
```



### Type overload

#### case1

```c++
class Complex {
private:
    double real, imag;
public:
    Complex(double r=0, double i=0): real(r),imag(i) { };
    // Type overload function return type is itself! not need to write double in the front
    operator double() {return real;}
};
int main()
{
    Complex c(1.2, 3.4);
    cout << (double)c << endl;
    double n = 2 + c;	// it can be compiled because c can be converted to double	double n = 2 + c.`operator double()`;
    cout << n;
}

/* output:
1.2
3.2
*/
```



### ++ -- overload

#### case1

```c++
// 前置运算符作为一元运算符重载
// member function
T & opeartor++();

// global function
T1 & operator++(T2);
```



#### case2

```c++
// 后置运算符作为二元运算符重载，多写一个没用的参数
// member function
T operator++(int);

// global function
T1 operator++(T2, int);
```



#### case3

前置++比后置++快，因为后置++会新建一个对象。

```c++
class CDemo {
private:
    int n;
public:
    CDemo(int i=0):n(i){ }
    CDemo & operator++();		// ++i return &i in c++
    							// ++a = 1; // ok
    CDemo opeartor++(int);		// i++ return the value of i
    							// a++ = 1; // compile error
    operator int() {return n;}
    friend CDemo & operator--(CDemo &);
    friend CDemo operator--(CDemo &, int);
};

CDemo & operator++() 
{
    ++n;
	return *this;    
}

CDemo CDemo::opeartor(int k)
{
    CDemo tmp(*this);
    n++;
    return tmp;
}

CDemo & operator--(CDemo & d) 
{
	d.n--;
    return d;
}

CDemo & operator--(CDemo & d, int) 
{
    CDemo tmp(d);
    d.n--;
    return tmp;
}

int main()
{
	CDemo d(5);
    cout << (d++) <<",";
}
```





### operator overload caveat

1. c++ 不允许定义新的运算符
2. 重载后运算符的含义应该符合日常习惯
3. 运算符重载不改变运算符的优先级
4. `.` , `.*`, `::`, `?:`, `sizeof` 不能被重载
5. `()`, `[]`, `->`, `=` 这些运算符重载函数必须是类成员函数





### Inheritance

- base class
- derived class

**derived class can not access the private variable in base class**

**the size of derived class = the size of base class + itself**

**the same function in the base class and derived class can be different**

**A derived class should also be the base class**



#### case1

```c++
class CStudent {
private:
    string name;
    string id;
    char gender;
    int age;
public:
    void PrintInfo();
    void SetInfo(const string & name_, const string & id_, int age_, char gender_);
    string GetName() {return name;}
};

class CUndergraduateStudent: public CStudent {
private:
    string Department;
public:
    void QualifiedForBaoyan() {
        cout << "Qualified" <<endl;
    }
    // derived c
    // override
    void PrintInfo() {
        CStudent::PrintInfo();
        cout << "Department: " << department << endl;
    }
};
```



#### case2

```c++
class CPoint
{
    double x, y;
};

// 一个圆是一个点？？？
class CCircle:public CPoint
{
    double r;
};

------------------------------------------------------------------------

// 复合关系
class CPoint
{
    double x, y;
    friend class CCircle;
};

class CCircle
{
    double r;
    CPoint center;
}
```



#### case3

复合关系

wrong case1

```c++
class CDog;
class CMaster
{
    CDog dogs[10];
};

class CDog
{
    CMaster m;
}

// sizeof(CMaster) = 10 * sizeof(CDog);
// sizeof(CDog) = sizeof(CMaster)
// So what are the sizes of them?????
```



wrong case2

````c++
class CDog;
class CMaster 
{
  CDog * dogs[10];
};

class CDog
{
    CMaster m;
}

// can be compiled but
// if two dogs have the same master, one dog's master changed then another dog's master needs to be changed
````



wrong case3

```c++
class CMaster;

class CDog
{
    CMaster* pm;
}

class CMaster 
{
  CDog dogs[10];	// 狗是主人的固有属性吗？	
};
```



correct case

```c++
class CDog;
class CMaster 
{
  CDog * dogs[10];
};

class CDog
{
    CMaster * pm;
}
// ptr = 知道关系
```





### Override

派生类可以定义一个和基类成员同名的成员，称为覆盖

defaul access derived class member

To access base class member, need to use ::

#### case1

```c++
class base {
    int j;
public:
    int i;
    void func();
}

class derived:public base {
public:
    int i;		// in fact is not good to write the same variable in base and derived class
    void access();
    void func();
}

void derived::access()
{
	j = 5;			// compile erorr! private
    i = 5;			// derived.i
    base::i = 5;	// base.i
    func();			//derived.func()
    base::func();	// base.func()
}

derived obj;
obj.i = 1;			// derived.i
obj.base::i = 1;	// base.i
```





### protected

![image-20220520143430243](C:\Users\28562\AppData\Roaming\Typora\typora-user-images\image-20220520143430243.png)



#### case1

```c++
class Father {
	private: int nPrivate;
    public: int nPublic;
    protected: int nProtected;
};

class Son: public Father {
    void AccessFather() {
        nPublic = 1;	//ok
        nPrivate = 1;	// wrong
        nProtected = 1; // ok
        Son f;
        f.nProtected = 1;	// wrong, f is not *this
    }
};

int main()
{
    Father f;
    Son s;
    f.nPublic = 1;
    s.nPublic = 1;
    f.nProtected = 1;	// error
    f.nPrivate = 1;		// error
    s.nProtected = 1;	// error
    s.nPrivate = 1;		// error
    return 0;
}
```



### derived class construction

在执行一个派生类的构造函数之前，总是先执行基类的构造函数。

1. 显式调用基类构造函数：初始化列表
2. 隐式调用：自动调用基类无参构造函数

#### case1

```c++
class Bug {
private:
    int nLegs;
    int nColor;
public:
    int nType;
    Bug(int Legs, int color):nlegs(legs),ncolor(color) { };
    void PrintBug() { };
};

class FlyBug: public Bug {
    int nWings:
public:
    FlyBug(int legs, int color, int wings);
};

// error construction
FlyBug::FlyBug(int legs, int color, int wings) {
	nLegs = legs;	// compile error, private cannot access
    nColor = color;	// compile error, private cannot access
    nType = 1;
    nWings = wings;
}

// correct, use initialized list
// cannot use base construction function in the derived construction function
FlyBug::FlyBug(int legs, int color, int wings):Bug(legs, color) {
    nWings = wings;
}
```



#### case2

```c++
class Base {
public:
    int n;
    Base(int i):n(i) {
        cout << "Base " << n <<" constructed" << endl;
    }
    ~Base() {
        cout << "Base " << n << " destructed" << endl;
    }
}

class Derived:public Base {
public:
    // must use initialized list
    // since there is no nonparameter construction function in Base
    Derived(int i):Base(i) {
		cout << "Derived constructed" << endl;
    }
    ~Derived() {
        cout << "Derived destructed" << endl;
    }
};
int main() 
{
    Derived Obj(3);
    return 0;
}

/* output
Base 3 constructed
Derived constructed
Derived destructed
Base 3 destructed
*/
```



#### case3

```c++
class Bug {
private:
    int nLegs;
    int nColor;
public:
    int nType;
    Bug(int legs, int color);
    void PrintBug(){ };
};

class Skill {
public:
    Skill(int n) { }
};

class FlyBug:public Bug {
	int nWings;
    Skill sk1, sk2;
public:
    FlyBug(int legs, int color, int wings);
}

// initialized member class variable
// first base construction
// then sk construction
// last derived construction
FlyBug:FlyBug(int legs, int color, int wings):
	Bug(legs, color), sk1(5), sk2(color), nWings(wings) { }
```





### public inheritance

#### case1

```c++
class base { };
class Derived: public base { };
base b;
derived d;
```

1. derived object can be assigned to base object: `b = d`
2. derived object can be assigned to the reference of base class: `base & br = d;`
3. derived object address can be assigned to base ptr: `base * pb = & d;`

If we use private / protected inheritance, we would not have the upper rules;





### direct base class and indirect bace class

![image-20220521161804165](C:\Users\28562\AppData\Roaming\Typora\typora-user-images\image-20220521161804165.png)

![image-20220521161837599](C:\Users\28562\AppData\Roaming\Typora\typora-user-images\image-20220521161837599.png)



#### case1

```c++
class Base {
public:
    int n;
    Base(int i):n(i) {
        cout << "Base " << n << " constructed" << endl;
    }
    ~Base() {
        cout << "Base " << n << " destructed" << endl;
    }
};

class Derived:public Base {
public:
    Derived(int i):Base(i) {
        cout << "Derived constructed" << endl;
    }
    ~Derived() {
        cout << "Derived destructed" << endl;
    }
};

class MoreDerived:public Derived {
public:
    MoreDerived():Derived(4) {
        cout << "More Derived constructed" << endl;
    }
    ~MoreDerived() {
        cout << "More Derived destructed" << endl;
    }
};

int main()
{
	MoreDerived Obj;
    return 0;
}

/*
output
Base 4 constructed
Derived constructed
More Derived constructed
More Derived destructed
Derived derstructed
Base 4 destructed
*/
```





### virtual function 

#### case1

`virtual` only used in class definition

construction function and static function cannot be `virtual`

`virtual` function can be used in polymorphism

```c++
class base {
	virtual int get();
};

int base::get() { }
```





### polymorphism 多态

#### case1

```c++
class CBase {
public:
    virtual void SomeVirtualFunction() { }
};

class CDerived:public CBase {
public:
    virtual void SomeVirtualFunction() { }
};

int main() {
    CDerived ODerived;
    CBase * p = & ODerived;
    // p would use CDerived funtion
    // if p points to CBase object, would use CBase function
    // this is one of the polymorphism
    p -> SomeVirtualFunction();
    return 0;
}
```



#### case2

```c++
int main() {
    CDerived ODerived;
    CBase * r = ODerived;
    // r would use CDerived funtion
    // if r is the CBase reference, would use CBase function
    // this is one of the polymorphism
    r.SomeVirtualFunction();
    return 0;
}
```



#### case3

![image-20220521163505618](C:\Users\28562\AppData\Roaming\Typora\typora-user-images\image-20220521163505618.png)

```c++
class A {
public:
    virtual void Print() {
		cout << "A::Print" << endl;
    }
};

class B: public A {
public:
    virtual void Print() {
		cout << "B::Print" << endl;
    }
};

class D: public A {
public:
    virtual void Print() {
		cout << "D::Print" << endl;
    }
};

class E: public B {
public:
    virtual void Print() {
		cout << "E::Print" << endl;
    }
};

int main() 
{
    A a; B b; E e; D d;
    A * pa = &a; B * pb = &b;
    D * pd = &d; E * pe = &e;
    
    pa->Print();	// polymorphism function
    pa = pb;
    pa->Print();
    pa = pd;
    pa->Print();
    pa = pe;
    pa->Print();
    return 0;
}

/*
output:
A::Print
B::Print
D::Print
E::Print
*/
```



#### case4

![image-20220523095615026](C:\Users\28562\AppData\Roaming\Typora\typora-user-images\image-20220523095615026.png)



#### case5

在非构造函数，非析构函数的成员函数中调用虚函数，是多态。

```c++
class Base {
public:
    void fun1() {fun2();}	// equals to this->fun2(), fun2 is virtual function(polymorphism) if this is derived ptr, it would use derived function
    virtual void fun2() {cout<<"Base::fun2()"<<endl;}
};

class Derived:public Base {
public:
    virtual void fun2() {cout<<"Derived:fun2()"<<endl;}
};

int main()
{
    Derived d;
    Base* pBase = & d;
    pBase->fun1();
    return 0;
}

/*
output:
Derived:fun2()
*/
```



#### case6

在构造函数和析构函数中调用虚函数，不是多态。编译时即可确定，调用的函数是自己的类或基类中定义的函数，不会等到运行时才决定调用自己还是派生类的函数。

```c++
class myclass {
public:
    virtual void hello() {cout<<"hello from myclass"<<endl;}
    virtual void bye() {cout<<"bye from myclass"<<endl;}
};

class son:public myclass {
public:
    // the same function in derived class and base virtual function
    // even if it has not virtual, it would be the virtual function automatically
    void hello() {cout<<"hello from son"<<endl;}
    son() {hello();}
    ~son(){bye();}
};

class grandson:public son {
public:
    void hello() {cout<<"hello from grandson"<<endl;}	// virtual function
    void bye() {cout<<"bye from grandson"<<endl;}		// virtual function
    grandson() {cout<<"constructing grandson"<<endl;}
    ~grandson() {cout<<"destructing grandson"<<endl;}
}

int main()
{
    grandson gson;
    son *pson;
    pson = & gson;
    pson->hello();	// polymorphism
    return 0;
}

/*
output
hello from son	// gson would first construct son
constructing grandson
hello from grandson
destructing grandson
bye from myclass
*/
```



### 多态的实现原理

动态联编（dynamic binding）：编译时不确定到底调用的是基类还是派生类的函数，运行时才确定。

The concept of dynamic programming is implemented with virtual functions.

#### case1

```c++
class Base {
public:
    int i;
    virtual void Print() {cout << "Base:Print";}
};

class Derived:public Base {
public:
    int n;
    virtual void Print() {cout << "Derived:Print" << endl;}
};

int main() {
    Derived d;
    cout << sizeof(Base) << "," << sizeof(Derived);
    return 0;
}

/*
output:
8,12
?? why all more 4 bytes?
4 bytes used for constructing a virtual function list 
*/
```

虚函数表：每一个有虚函数的类（或有虚函数的类的派生类）都有一个虚函数表，该类的任何对象中都放着虚函数表的指针。虚函数表中列出了该类的虚函数的地址。多出来的4个字节就是用来放虚函数表的地址。缺点：额外时间（查表时间）、空间（存表4byte）开销

多态的函数调用语句被编译成一系列根据基类指针所指向的（或基类引用所引用的）对象中存放的虚函数表的地址，在虚函数表中查找虚函数地址，并调用虚函数的指令。

![image-20220524092257397](C:\Users\28562\AppData\Roaming\Typora\typora-user-images\image-20220524092257397.png)

![image-20220524092613316](C:\Users\28562\AppData\Roaming\Typora\typora-user-images\image-20220524092613316.png)



#### case2

```c++
class A {
public:
    virtual void Func() {cout << "A::Func" << endl;}
};

class B:public A {
public:
    virtual void Func() {cout << "B::Func" << endl;}
};

int main() {
    A a;
    A * pa = new B();
    pa->Func();
    // 64位程序指针为8字节
    // longlong 8字节
    long long * p1 = (long long *) & a;	// 取a的头八个字节，(long long *)
    // 指针修改了变量的某几个部分
    // 前8个字节是 class A 虚函数表的地址
    long long * p2 = (long long *) pa;	// class B 虚函数表的地址
    * p2 = * p1;
    pa->Func();
    return 0;
}
/*
output:
B::Func
A::Func 
*/
```





### virtual destructed function

通过基类的指针删除派生类对象时，通常情况下只调用基类的析构函数。但是，删除一个派生类的对象时，应该先调用派生类的析构函数，然后调用基类的析构函数。

解决办法：把基类的析构函数声明成virtual。

一般来说，一个类如果定义了虚函数，则应该将析构函数也定义成虚函数。或者，一个类打算作为基类使用，也应该将析构函数定义成虚函数。

不允许构造函数是虚函数

#### case1

```c++
class son {
public:
    ~son() {cout << "bye from son" << endl;};
};

class grandson:public son {
public:
    ~grandson() {cout << "bye from grandson" << endl;};
};

int main() {
	son *pson;
    pson = new grandson();
    delete pson;
    return 0;
}

/*
output:
bye from son	// because destructed function is not virtual
*/
```



#### case2

```c++
class son {
public:
    virtual ~son() {cout << "bye from son" << endl;};
};

class grandson:public son {
public:
    ~grandson() {cout << "bye from grandson" << endl;};
};

int main() {
	son *pson;
    pson = new grandson();
    delete pson;
    return 0;
}

/*
output:
bye from grandson
bye from son	
*/
```





### pure virtual function / abstract class

没有函数体的虚函数

包含纯虚函数的类叫抽象类

抽象类只能作为基类来派生新类使用，不能创建抽象类的对象

抽象类的指针和引用可以指向派生类对象



#### case1

```c++
class A {
private:
    int a;
public:
    virtual void Print() = 0;	// pure virtual function
    void fun() {cout << "fun";}
};

A a;	// wrong! A is abstract class, cannot be constructed
A * pa; // ok, can define a abstract class ptr and ref
pa = new A;	// wrong! A is abstract class, cannot be constructed
```





#### case2

在抽象类的成员函数内可以调用纯虚函数，但是在构造函数或析构函数内不能调用纯虚函数。之前说过在构造函数和析构函数调用虚函数不是多态

如果一个类从抽象类派生而来，那么当且仅当它实现了基类中所有的纯虚函数，它才能成为非抽象类。

```c++
class A {
public:
    virtual void f() = 0;
    void g(){
        this -> f();	//ok
    }
    A() {}	// cannnot use f()
};

class B:public A {
public:
    void f() {cout << "B:f()" << endl;}
};

int main() {
	B b;
    b.g();
    return 0;
}

/*
output:
B:f()
*/
```





### io class

![image-20220524104419533](C:\Users\28562\AppData\Roaming\Typora\typora-user-images\image-20220524104419533.png)



#### case1

```c++
#include <iostream>
int main() {
    int x, y;
    cin >> x >> y;
    freopen("test.txt", "w", stdout);	// 将标准输出重定向到txt文件
    if(y == 0)
        cerr << "error." << endl;	// 在屏幕打印错误信息
   	else
        cout << x/y;		// 输出结果到txt
    return 0;
}
```



#### case2

```c++
#include <iostream>
int main() {
    double f;
    int n;
    freopen("t.txt", "r", stdin);	// cin被改为从txt中读取数据
    cin >> f >> n;			// 从文件中读取
    cout << f << "," << n << endl;
    return 0;
}
```



#### case3

判断输入流结束

如果是从文件输入，比如前面有`freopen("some.txt", "r", stdin);`，那么读到文件尾部输入流就算结束

如果从键盘输入（非重定向），则在单独一行输入ctrl+z代表输入流结束EOF

```c++
int x;
// cin 强制转化成 bool
while(cin >> x) {
    ...
}
```



#### case4

```c++
int main() {
    int x;
    char buf[100];
    cin >> x;
    cin.getline(buf, 90);
    cout << buf << endl;
    return 0;
}

/*
1:
input:
12 abcd\n

output:
 abcd\n

2
input:
12\n
output:
\n
```







### 流操纵算子

- 整数流的基数 dec, oct, hex, setbase
- 浮点数的精度 precision, setprecision
- 设置域宽 setw, width
- 用户自定义的流操纵算子

使用流操纵算子需要`#include <iomanip>`



### case1

```c++
int n = 10;
cout << n << endl;
cout << hex << n << "\n"	// 连续起作用
    << dex << n << "\n"
    << oct << n << endl;

/* 
output:
10
a
10
12
*/
```



#### case2

```c++
cout.precision(5);
cout << setprecision(5);	// 连续起作用

double x = 1234567.89;
double y = 12.34567;
int n = 1234567;
int m = 12;
cout << setprecision(6) << x << endl // 默认小数点位置不固定
    << y << endl << n << endl << m;

/*
output:
1.23457e+006		// 最多保留6位，整数部分不足，科学计数法
12.3457				// 不需要科学计数法
1234567
12
*/
```



#### case3

```c++
cout.precision(5);
cout << setprecision(5);	// 连续起作用

double x = 1234567.89;
double y = 12.34567;
int n = 1234567;
int m = 12;
cout << setiosflags(ios::fixed)
    << setprecision(6) << x << endl // 小数点位置固定 (小数点必须出现在浮点数的右边)
    << y << endl << n << endl << m;
	// << resetiosflags(ios::fixed)	取消固定小数点位置输出
/*
output:
1234567.890000		// 最多保留6位，整数部分不足，科学计数法
12.34570				// 不需要科学计数法
1234567
12
*/
```



#### case4

```c++
cin >> setw(4);		// cin.width(5);
int w = 4;
char string[10];
cin.width(5);
while(cin >> string) {
    cout.width(w++);
    cout<<string<<endl;
    cin.width(5);	// 宽度设置有效性是一次性的，每次读入和输出之前都要设置宽度
}

/*
input:
1234567890
output:
1234
 5678	// 输出宽度不足，在前面补空格
    90
*/
```



#### case5

```c++
ostream &tab(ostream &output) {
    return output << '\t';
}
cout << "aa" << tab << "bb" << endl;

/*
output:
aa	bb
*/
```





### file

文件可以用流读写

ifstream ofstream fstream

#### case1

```c++
#include <fstream>
ofstream outFile("clients.dat", ios::out|ios::binary);
```



#### case2

```c++
ofstream fout;
fout.open("test.out", ios::out|ios::binary);

if(!fout) {
	cout << "File open error!" << endl;
}
```



#### case3

对于输入文件，有一个读指针；

对于输出文件，有一个写指针；

对于输入输出文件，有一个读写指针；

标识文件操作的当前位置，该指针在哪里，读写操作就在哪里进行。

```c++
ofstream fout("a1.out", ios::app);	// ios::app append sth to the file
long location = fout.tellp();		// 取得写指针的位置
location = 10;
fout.seekp(location);				// 将写指针移动到第10个字节处
fout.seekp(location, ios::beg);		// 从头数location
fout.seekp(location, ios::cur);		// 从当前位置数location
fout.seekp(location, ios::end);		// 从尾部数location

ifstream fin("a1.in", ios:ate);		//打开文件，定位文件指针到文件尾
long location = fin.tellg();
location = 10L;
fin.seekg(location);
```



#### case4

文本文件读写

```c++
vector<int> v;
ifstream srcFile("in.txt", ios::in);
ofstream destFile("out.txt", ios::out);
int x;
while(srcFile >> x) {
    v.push_back(x);
}
sort(v.begin(), v.end());
for(int i = 0; i < v.size(); ++i) {
    destFile << v[i] << " ";
}
destFile.close();	// if we do not close file, they might not be written to the disk, but still in the memory
srcFile.close();
```



#### case5

`istream& read(char* s, long n)`

`istream& write(const char * s, long n)`

```c++
ofstream fout("some.dat", ios::out|ios::binary);
int x = 120;
fout.write((const char *)(&x), sizeof(int));
fout.close();
ifstream fin("some.dat", ios::in|ios::binary);
int y;
fin.read((char *)& y, sizeof(int));
fin.close();
cout << y << endl;
```



#### case6

```c++
struct Student {
    char name[20];
    int score;
};
int main()
{
    Student s;
    ofstream OutFile("students.dat", ios::out|ios::binary);
    while(cin >> s.name >> s.score) {
        OutFile.write((char *) & s, sizeof(s));
    }
    OutFile.close();
    
    isstream inFile("students.dat", ios::in|ios::binary);
    if(!inFile) {
		cout << "error"<<endl;
        return 0;
    }
    while(inFile.read((char *)& s, sizeof(s))) {
        int readedBytes = inFile.gcount();
        cout << s.name << " " << s.score << endl;
    }
    
    fstream iofile("students.dat", ios::in|ios::out|ios::binary);
    if(!iofile) {
        cout << "error";
        return 0;
    }
    iofile.seekp(2 * sizeof(s), ios::beg);		// 定位写指针到第三个记录
    iofile.write("Mike", strlen("Mike")+1);		// 修改第三个记录的名字
    iofile.seekg(0, ios::beg);	// 定位读指针到开头
    while(iofile.read((char *)& s, sizeof(s))) {
        cout << s.name << " " << s.score << endl;
    }
    iofile.close();
    return 0;
}
```

二进制存储空间比文本文件小

文本文件：

Linux, Unix 的换行符号是 '\n'

Mac OS 的换行符号是 '\r\n'

Windows 的换行符号是 '\r'

二进制文件：

windows会把 '\r\n' 看成一个换行符





### Function Template 函数模板

Genric Programming (泛型程序设计)

#### case1

```c++
template <class T>
void Swap(T & x, T & y) {
    T tmp = x;
    x = y;
    y = tmp;
}
```



#### case2

```c++
template <class T1, class T2>
T2 print(T1 arg1, T2 arg2)
{
	cout << arg1 << " " << arg2 << endl;
    return arg2;
}
```



#### case3

```c++
template<class T>
T MaxElement(T a[], int size) {
    T tmpMax = a[0];
    for(int i = 1; i < size; ++i) {
        if(tmpMax < a[i]) {
			tmpMax = a[i];
        }
    }
    return tmpMax;
}
```



#### case4

```c++
template <class T>
T Inc(T n) {
	return 1 + n;
}
int main()
{
    cout << Inc<double>(4)/2;	// 输出2.5，如果不显式实例化函数模板，会是2
}
```



#### case5

函数模板可以重载

在有多个函数和函数模板名字相同的情况下，编译器如下处理一条函数调用语句

1. 先找参数完全匹配的普通函数
2. 再找参数完全匹配的模板函数
3. 再找实参经过自动类型转换后能够匹配的普通函数
4. 上面都找不到，报错

```c++
template<class T>
int Max(T a, T b) {
	cout << "TemplateMax"<< endl;
    return 0;
}

template<class T, class T2>
int Max(T a, T2 b) {
	cout << "TemplateMax2"<< endl;
    return 0;
}

double Max(double a, double b) {
    cout << "Mymax" << endl;
    return 0;
}

int main() {
	int i = 4;
    int j = 5;
    Max(1.2, 3.4);	// Mymax
    Max(i, j);		// TemplateMax
    Max(1.2, 3);	// TemplateMax2
    return 0;
}
```



#### case6

匹配模板函数时，不进行类型自动转换

```c++
template <class T>
T myFunction(T arg1, T arg2) {
    cout << arg1 << " " << arg2 << "\n";
    return arg1;
}
myFunction(5, 7);
myFunction(5.8, 8.4);
myFunction(5, 8.4);		// error!!!
```





### class template 类模板

类成员模板函数不能作为虚函数

#### case1

```c++
template <class T1, class T2>
class Pair {
public:
    T1 key;
    T2 value;
    Pair(T1 k, T2 v):key(k), value(v) { };
    bool operator < (const Pair<T1, T2> & p) const;
}

template <class T1, class T2>
bool Pair<T1, T2>::operator < (const Pair<T1, T2> & p) const {
    return key < p.key;
}

int main()
{
    // 实例化一个类
    Pair<string, int> student("Tom", 19);
    cout << student.key << " " << student.value;
    return 0;
}
```



#### case2

```c++
Pair<string, int> * p;
Pair<string, double> a;
p = & a;	// wrong
```



#### case3

```c++
template <class T>
class A {
public:
    template<class T2>
    void Func(T2 t) {cout << t;}
};

int main()
{
	A<int> a;
    a.Func('K');
    a.Func("hello");
    return 0;
}
```



#### case4

```c++
template <class T, int size>
class CArray{
    T array[size];
public:
    void Print() {
        for(int i = 0; i < size; ++i) {
            cout << array[i] << endl;
        }
    }
};
CArray<double, 40> a2;
CArray<int, 50> a3;
```



#### case5

类模板从类模板派生

```c++
template <class T1, class T2>
class A {
    T1 v1; T2 v2;
};

template <class T1, class T2>
class B:public A<T2,T1> {
	T1 v3; T2 v4;
};

template <class T>
class C:public B<T,T> {
    T v5;
};

int main() {
    B<int, double> obj1;	// class B<int,double>:public A<double,int>
    C<int> obj2;		
    return 0;
}
```



#### case6

类模板从类模板的实例派生

```c++
template <class T1, class T2>
class A {
    T1 v1;
    T2 v2;
};

template <class T>
class B: public A<int, double> {
	T v;
};

int main() {
	B<char> obj1;		// class B<char> class A<int, double>
    return 0;
}
```



#### case7

类模板从普通类派生

```c++
class A {
    int v1;
};
template <class T>
class B: public A {
    T v;
};
int main() {
    B<char> obj1;
    return 0;
}
```



#### case8

普通类从模板类派生

```c++
template<class T>
class A {
    T v1;
    int n;
};

class B: public A<int> {
    double v;
};

int main() {
	B obj1;
    return 0;
}
```



#### case9

函数模板作为类模板的friend

```c++
template <class T1, class T2>
class Pair {
private:
    T1 key;
    T2 value;
public:
    template <class T3, class T4>
    // cout can access the private values
    friend ostream & operator << (ostream&o, const Pair<T3, T4> & p);
}

template <class T1, class T2>
ostream & operator << (ostream&o, const Pair<T1, T2> & p) {
    o << "(" << p.key << "," << p.value << ")";
    return o;
}
```



#### case10

任何从A模板实例化出来的类，都是任何B实例化出来的类的友元。

```c++
template <class T>
class B {
    T v;
public:
    B(T n):v(n){}
    template <class T2>	// attention!!!
    friend class A;
};

template <class T>
class A {
public:
    void Func() {
        B<int> o(10);
        cout << o.v << endl;
    }
};

int main() {
    A<double> a;
    a.Func();
    return 0;
}
```



#### case 11

类模板中可以定义静态成员，那么从该类模板实例化得到的所有类都包含同样的静态成员。

```c++
class A {
private:
    static int count;
public:
    A() {++count;}
    ~A() {--count;}
    A(A &) {++count;}
    static void PrintCount() {cout << count << endl;}
};

template<> int A<int>::count = 0;
template<> int A<double>::count = 0;
int main() {
    A<int> ia;
    A<double> da;
    ia.PrintCount();
    da.PrintCount();
    return 0;
}

/* 
output:
1
1
*/
```





### string

```c++
string s;
s = 'a';	// ok
getline(cin, s);
cin >> s;

s.at(1) // ~ s[1], but at() will throw an exception
    
string s1("good "), s2("morning!");
s1.append(s2, 3, s2.size());	// append s2 substring to s1

int f1 = s1.compare(s2);	// -1 s1 < s2 0 s1 == s2 1 s1 > s2
// can compare substring

/*
find
rfind
find_first_of
find_last_of
find_first_not_of
find_last_not_of
*/ 

s1 = "hello world";
s1.erase(5); // s1=hello
s1.replace(2,3, "haha");	//hehaha
s1.insert(5,s2);
```





### C++11

#### 统一的初始化方法

```c++
int arr[3]{1,2,3};
vector<int> iv{1,2,3};
map<int,string> mp{{1, "a"}, {2,"b"}};
string str{"Hello World"};
int *p = new int[20]{1,2,3};

struct A {
    int i,j;
    A(int m, int n):i(m), j(n){ }
};

A func(int m, int n) {
    return {m, n};
}

int main() {
    A *pa = new A{3,7};
}
```



#### 成员变量默认初始值

```c++
class B {
public:
    int m = 1234;
    int n;
}
B b;
b.m;	// 1234
```



#### auto

编译器自动判断变量的类型

```c++
auto i = 100;		// int
auto p = new A();	// A*
auto k = 34343LL;	//long long

map<string, int, greater<string>> mp;
for(auto i= mp.begin(); i != mp.end(); ++i)
    cout << i->first << ","<<i->second<<endl;

class A {};
A operator+(int n, const A& a) {
	return a;
}
template <class T1, class T2>
// ->decltype(x+y)==x+y的类型
auto add(T1 x, T2 y)->decltype(x+y) {
	return x+y;
}

auto d = add(100, 1.5);	// double
auto k = add(100, A());	// A
```



#### decltype

```c++
int i;
double t;
struct A {double x;};
const A* a = new A();

decltype(a) x1;		// x1 is A* <-> A* x1;
decltype(i) x2;		// x2 is int <-> int x2;
decltype(a->x) x3; 	// x3 is double
decltype((a->x)) x4 = t; // x4 is double&
// double& x4 = t;
```



#### shared_ptr

`shared_ptr<T> ptr(new T);`

托管一个new运算符返回的指针

此后`ptr`可以像T*类型的指针一样来使用，不必担心释放内存（不需要delete）

多个`shared_ptr`对象可以同时托管一个指针，系统会维护一个托管计数。当没有`shared_ptr`托管该指针时，delete该指针。

`shared_ptr`不能托管指向动态分配的数组的指针，否则程序运行会出错。

##### case1

```c++
#include <memory>
#include <iostream>
using namespace std;
struct A {
    int n;
    A(int v= 0;) n(v){}
    ~A() {cout << n <<"destructor"<<endl;}
};

int main() {
    shared_ptr<A> sp1(new A(2));
    shared_ptr<A> sp2(sp1);		// sp2托管A(2)
    cout << "1)" << sp1->n <<"," << sp2->n << endl;
    shared_ptr<A> sp3;
    A* p = sp1.get();
    cout << "2)" << p->n << endl;
    sp3 = sp1;		// sp3托管A(2)
    cout << "3)" << (*sp3).n << endl;
    sp1.reset();	// sp1放弃托管A(2)
    if(!sp1) {
        cout << "4)sp1 is null" << endl;
    }
    A* q = new A(3);
    sp1.reset(q);
    cout << "5)" <<sp1->n << endl;
    shared_ptr<A> sp4(sp1);
    shared_ptr<A> sp5;
    // sp5.reset(q);	// error
    sp1.reset();
    cout << "before end main" << endl;
    sp4.reset();	// delete A(3)
    cout << " end main"<< endl;
    return 0;	// delete A(2)
}
```



##### case2

```c++
#include <iostream>
#include <memory>
using namespace std;
struct A {
	~A() {cout << "~A" << endl;}
};
int main() {
    A *p = new A();
    shared_ptr<A> ptr(p);
    shared_ptr<A> ptr2;
    ptr2.reset(p);	// 并不增加ptr中对p的托管次数
    // 因为ptr2不知道它托管的和ptr1是一样的
    cout << "end" << endl;
    return 0;
}

/* 
output:
end
~A
~A
之后程序崩溃
因p被delete两次
*/
```



#### nullptr



#### for

```c++
int arr[] = {1,2,3,4,5};
for(int &e: arr) {
    e *= 10;
}

vector<A> st(ary, ary+5);
for(auto & it: st) {
    it.n *= 10;
}
```



#### 右值引用 move

右值：不能取地址的表达式

目的：提高程序的运行效率，减少需要进行深拷贝的对象进行深拷贝的次数

```c++
class A {};
A &r = A();		// A() 是无名变量，是右值
A && r = A();	// r是右值引用
```



##### case1

```c++
class String {
public:
    char * str;
    String():str(new char[1]) {str[0] = 0;}
    String(const char* s) {
        str = new char[strlen(s)+1];
        strcpy(str, s);
    }
    
    // deep copy
    String(const String & s) {
        cout << "copy constructor called" << endl;
        str = new char[strlen(s.str)+1];
        strcpy(str, s.str);
    }
    String & operator=(const String & s) {
        cout << "copy ooperator= called" << endl;
        if(str != s.str) {
            delete[] str;
            str = new char[strlen(s.str)+1];
            strcpy(str, s.str);
        }
        return * this;
    }
    
    // move constructor
    // 改变了参数的内容
    String(String && s):str(s.str) {
        cout << "move constructor called" << endl;
        s.str = new char[1];
        s.str[0] = 0;
    }
    // move assignment
    String & operator=(String && s) {
        cout << "move operator= called" << endl;
        if(str != s.str) {
			delete[] str;
            str = s.str;
            s.str = new char[1];
            s.str[0] = 0;
        }
        return *this;
    }
    ~String() {delete [] str;}
};

// 不在乎a，b被修改
template <class T>
void MoveSwap(T& a, T& b) {
	T tmp(move(a));		// move(a)右值
    a = move(b);		// move(b)右值
    b = move(tmp);		// move(tmp)右值
}

int main()
{
    String s;
    s = String("ok");	// 右值
    cout << "*****" << endl;
    String && r = String("this");
    cout << r.str << endl;
    String s1 = "hello", s2 = "world";
    MoveSwap(s1, s2);
    cout << s2.str << endl;
    return 0;
}

/*
output:
move operator= called
*****
this 
move constructor called
move operator= called
move operator= called
hello
*/
```



#### unordered_map



#### regex



#### lambda

[] ()-> {}

- `[=]` 传值使用外部变量
- `[]` 不使用外部变量
- `[&]` 引用使用外部变量
- `[x, &y]` x传值，y引用
- `[=, &x, &y]` xy引用，其余传值
- `[&, x, y]` xy传值，其余引用

##### case1

```c++
int x = 100, y = 200, z = 300;
cout << [](double a, double b) {return a+b} (1.2, 2.5) <<endl;

auto ff = [=, &y, &z](int n) {
    cout <<x << endl;
    y++;
    z++;
    return n*n;
};
cout << ff(15) << endl;
```



##### case2

```c++
int a[4] = {4, 2, 11, 33};
sort(a, a+4, [](int x, int y)->bool{return x%10 < y % 10;});
for_each(a, a+4, [](int x){cout << x << " ";});
```



##### case3

```c++
vector<int> a{1,2,3,4};
int total = 0;
for_each(a.begin(), a.end(),[&](int & x) {total += x; x *= 2;});

cout << total << endl;
for_each(a.begin(), a.end(), [](int x){cout << x << " ";});
```



##### case4

function<int(int)> 表示返回值int，有一个int参数的函数

```c++
function<int(int)> fib = [&fib](int n) {return n <= 2 ? 1 : fib(n-1) + fib(n-2);};

cout << fib(5) << endl;
```





### 强制类型转换

#### static_cast

自然，低风险转换：int to float， int to char

不能在不同类型的指针之间互相转换，也不能用于整型和指针之间的互相转换，也不能用于不用类型的引用之间的转换。

```c++
class A {
public:
    operator int() {return 1};
    operator char*() {return nullptr;}
};
int main() {
    A a;
    int n ;
    char *p = "New Dragon";
    n = static_cast<int>(3.14);
    n = static_cast<int>(a);
    p = static_cast<char*>(a);
    n = static_cast<int>(p);	// error
    p = static_cast<int>(n);	// error
}
```



#### reinterpret_cast

不用类型的指针之间的转换，不同类型的引用之间的转换，指针和能容纳的下指针的整数类型之间的转换。转换的时候，执行的是逐个比特拷贝的操作。

```c++
class A {
public:
    int i;
    int j;
    A(int n):i(n),j(n){}
};

int main() {
	A a(100);
    // 强行用r引用a
    int &r = reinterpret_cast<int&>(a);
    r = 200; // a.i = 200
    cout << a.i << "," << a.j << endl;
    int n = 300;
    A *pa = reinterpret_cast<A*>(& n);
    pa->i = 400;	// n = 400
    pa->j = 500;	// 不安全，可能导致程序崩溃
    cout << n << endl;	
    long long la = 0x12345678abcdLL;
	// la太长，只取低32位0x5678abcd拷贝给pa
    pa = reinterpret_cast<A*>(la);
    cout << hex << u<< endl; // 5678abcd
    typedef void(* PF1)(int);
    typedef int (*PF2) (int, char*);
    PF1 pf1; 
    PF2 pf2;
    pf2 = reinterpret_cast<PF2>(pf1);
}
```



#### const_cast

const to nonconst

nonconst to const



#### dynamic_cast

用于多态基类的指针或引用，强制转换为派生类的指针或引用，而且能够检查转换的安全性。

```c++
class Base {
public:
    virtual ~Base() {}
};
class Derived:pubic Base{};
int main() {
	Base b;
    Derived d;
    Derived *pd;
    pd = reinterpret_cast<Derived*>(& b);
    // pd不会是nullptr， 无法检查
    if(pd == nullptr) {
        cout << "unsafe" << endl;
    }
    pd = dynamic_cast<Derived*>(&b);
    // &b 不是指向派生类对象，pd = nullptr
    if(pd == nullptr) {
        cout << "unsafe" << endl;
    }
    Base* pb = &d;
    pd = dynamic_cast<Derived*>(pb);
    if(pd == nullptr) {
        cout << "unsafe" << endl;
    }
    return 0;
}
```

