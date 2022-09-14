## part1

member template function could not be virtual

stl avoids inheritance and dynamic binding

stl is generic programming not oriented-object



## part2

### case1

template class can be specialized outside and redefine its member

In this case, container \<char> no longer has  `increase` member function, but has `uppercase` instead.

container <user_defined class> , needs to overload some functions `xx operator++()`, `ostream& operator<<(ostream &os, xx t)`

```c++
template <class T>
class constainer {
public:
    explicit container (const T & arg): element (arg) {}
    T increase() {++element;}
private:
    T element;
};

template <> class container <char> {
public:
    explicit container (char arg): element (arg) {}
    char uppercase() {}
private:
    char element;
}
```



### case2

partial specialization

```c++
template <class T, class U>
struct Class {
  Class() {cout << "Class <T,U>\n";}
};

template <class T>
struct Class <T,T>{
    Class() {cout << "Class <T,T>\n";}
};

template <class T>
struct Class <T,int>{
    Class() {cout << "Class <T,int>\n";}
};

template <class T, class U>
struct Class <T*, U*>{
    Class() {cout << "Class <T*,U*>\n";}
};

int main() {
    Class <int, int> a;   // error, ambigious! solution: speicialize another function
}
```



### case3

default template parameter

```c++
template <class T = int>
class stack {
public:
    //..
private:
    vector<T> s;
};

int main() {
    stack <> sta;
}
```



### case4

```c++
template <class T, int N> // typed param T, non-type param N
class memblock {
public:
    void setmem(int x, T value);
    T getmem(int x);
private:
    T memblock[N] = {0};
}

template <class T, int V> 
T add (const T &n) {
    return n + V;
}

int main() {
    memblock <int, 5> my_ints;
    int i;
    i = add<int, 6> (10);
    cout<<i<<endl; // i = 16;
    return 0;
}
```



### case5

template template

container is a template class, needs to add template in the front, since we do not know which value type is stored in the container

```c++
template <class T, template <class ...> class container = deque>
class stack {
public:
    void push(const T &t) {s.push_back(t);}
    void pop() {s.pop_back();}
    T top() {return s.back();}
private:
    container<T> s;
};

int main() {
    stack<string> s1;
    stack<int, list> s2; 
}
```



### case6

macros: c generic programming

evil 

difference between macro and template

```c++
#define MIN(i, j) (((i) < (j)) ? (i) : (j))

template<T>
T min(const T &i, const T &j) {
    return i < j ? i : j;
}

int main() {
    int a = 10, b = 20;
    cout << "min = " << min(a++, b++) << endl;
    cout << "MIN = " << MIN(a++, b++) << endl;
    // equals to (((a++) < (b++)) ? (a++) : (b++))
    cout << "a = " << a << " b = " << b << endl;
}

/*
output:
min = 10	// after comparision do the ++
MIN = 12
a = 13 b = 22
*/
```



### case7

Template functions and member functions are only instantiated when they are used. Some older C++ compliers had problems instantiating all the templates properly. One way to prevent linker and compilation errors is to use "explicit instantiation", which provides each template argument when invoking a template function. For example, consider the following small template function.

```c++
template <class T>
void print_data_type (const T& a) {
    cout << typeid(a).name() << endl;
}

template <class T>
struct example_class {
    example_class () {}
    T result (T t) { return t; }
    void add (const T &t) {}
}; 

template void print_data_type<int> (const int &);
template void print_data_type<double> (const double &);

// explicit instantiate a template class constructor
template example_class<int>::example_class();

// explicit instantiate a entire class
template struct example_class<double>;

// explicit instantiate member functions
template example_class<string>::example_class();
template void example_class<string>::add(const string &);
template string example_class<string>::result(string);

int main() {
    example_class<int> ai;
    example_class<double> ad;
    print_data_type(ai);
    print_data_type(ad);
}

/*
output:
13example_classIiE
13example_classIdE
*/
```



### case8

template recursion function

```c++
template <class T>
T adder(T v) {
    cout << __PRETTY_FUNCTION__ << "\n";
    return v;
}

// variadic templates takes the first T, and arbitrary number of arguments 
template <class T, class ... Args>
T adder(T first, Args ... args) {
    cout << __PRETTY_FUNCTION__ << "\n";
    return first + adder(args ...);
}

int main() {
    long sum = adder(1, 2, 3, 8, 7);
    cout << "sum = " << sum << endl;
    string s1 = "C++", s2 = " ", s3 = "is", s4 = " ", s5 = "cool!";
    string string_sum = adder(s1, s2, s3, s4, s5);
    cout << "string sum = " << string_sum << endl;
}

/*
T adder(T, Args ...) [with T = int; Args = {int, int, int, int}]
T adder(T, Args ...) [with T = int; Args = {int, int, int}]
T adder(T, Args ...) [with T = int; Args = {int, int}]
T adder(T, Args ...) [with T = int; Args = {int}]
T adder(T) [with T = int]
sum = 21
*/
```



### case9

A **`constexpr`** function is one whose return value is computable at compile time when consuming code requires it. Consuming code requires the return value at compile time to initialize a **`constexpr`** variable, or to provide a non-type template argument. When its arguments are **`constexpr`** values, a **`constexpr`** function produces a compile-time constant. When called with non-**`constexpr`** arguments, or when its value isn't required at compile time, it produces a value at run time like a regular function. (This dual behavior saves you from having to write **`constexpr`** and non-**`constexpr`** versions of the same function.)

```c++
const long long factorial_20 = 2432902008176640000LL;

long long factorial(const long long val, const long long sum=1) {
    if(val > 1) {
        return factorial(val-1, sum*val);
    }
    else {
        return sum;
    }
}

constexpr long long cfactorial(const long long in, const long long sum=1) {
    // cout << __PRETTY_FUNTION__ << endl; //not allowable, since it is not const
    if(in > 1) {
        return cfactorial(in-1, sum*in);
    }
    else {
        return sum
    }
}

enum{ compileTimeGenratedValue = cfactorial(20)};
// if factorial_20 is not right, it would failed!
// because it runs in compile-time!
static_assert(compileTimeGeneratedValue == factorial_20, "error");

/**
 * Define a "compile-time" factorial template using a generic
 * metafunction calls (inherits from) itself.
 */
template<long long LLn, long long LSum = 1>
struct Factorial
       : Factorial<LLn - 1, LLn * LSum>{};

/**
 * This specialized metafunction has a value and does not inherit.
 */
template<long long LSum> // take LSum as a wild card
struct Factorial<1, LSum> {
    enum { value = LSum };
};
```



### case10

`forward<class T>(arg)`: Returns an *rvalue reference* to arg if arg is not an *lvalue reference*.

```c++
template<class T,
		 template <class ...> class sequential_container = vector>
struct container_wrapper {
    template<class ...Args>
	explicit container_wrapper(Args&&... args): container_(forward<Args>(args)...) {}
    
    sequential_container<T> container_;
};

int main() {
    const char* shapes[3] = {"Circle", "Triangle", "Square"};
    container_wrapper<string> b1(3, "words");	// forward right value reference
    container_wrapper<const char*, deque> b2(shapes, shapes+3);
    cout << b1 << endl;
    cout << b2 << endl;
    return 0;
}
```



### case11

Args handle for key-value pairs

```c++
template <template<class, class...> class container,
	class valueType,
	class... Args>
void print_container(ostream &os, const container<valueType, Args...>& objs) {
    os << __PRETTY_FUNCTION << endl;
    for(auto const &obj: objs) {
        os << obj << ' ;
    }
    os << endl;
}
```

