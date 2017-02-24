



##第三章
string 
vector
迭代器
bitset
using std::cin
using namespace std;



#string初始化方式
1. string s1;
2. string s2(s1);
3. string s3("value");
4. string s4(n, 'c');




#读取
string s
1.cin >> s;

```c++
int main()
{
	string line;
	while(getline(cin, line))
		cout << line <<endl;
	return 0;
}
```


#string的操作
	s.empty()
	s.size()
	s[n]
	s1+s2
	s1 = s2
	v1 == v2
	!=, <, <=, >, >=

s.size()返回的是string::size_type类型
不要把size的返回值赋值给int变量

头文件后缀 
```
#include<cname>
#include<name.h>
```
cname是c++版本，而name.h是c版本 c++源自C，cname头文件定义的名字都定义在命名空间std内，而.h版本中的名字却不是这样子

\#include<vector>
using std::vector;

vector<int> ivec;

#vector的初始化方式
1. vector<int> ivec1;
2. vector<int> ivec2(ivec1);
3. vector<int> ivec3(10, 1);
4. vector<int> ivec4(10);


#vector的操作
1. v.empty();
2. v.size();
3. v.push_back(t);
4. v[n];
5. v1 = v2;
6. v1 == v2;
7. != < <= > >=

size()返回的是vector<T>::size_type


#iterator 迭代器

vector<string>::iterator iter;
iter.begin();
iter.end();

#迭代器的自增和解饮用
\*iter
++iter

```C++
for(vector<int>::iterator iter=ivec.begin(); iter!=ivec.end(); iter++)
	*iter = 0;
```

vector<string>::const_iterator 不能对const_iterator类型进行赋值，返回的是const值

const vector<string>::iterator 可以赋值 但是不能自增

迭代器支持算数运算
iter+ n
iter1 - iter2 得到的difference_type的signed类型 

#任何改变vector长度的操作都会使已经存在的迭代器失效！！！！！

----------------
####bitset#####
----------------





















##第7章 函数
1. 给函数传递参数
2. 从函数返回值
3. 内联函数
4. 类成员函数
5. 重载函数
6. 函数指针

函数不能返回另一个函数或者内置数组类型，但是可以返回指向函数的指针。 int \*foo_bar()


如果形参具有非引用类型，则复制实参的值；如果形参为引用类型，则它只是实参的别名。

引用形参 void swap(int i, int j)()   void swap(int &i, int &j)()

应该将不需要修改的引用形参定义为const引用，普通的非const引用形参在使用时不太灵活。这样子的形参既不能用const对象初始化，也不能用字面值或产生右值的表示式实参初始化



int \*&v1 v1是一个引用，指向int型对象



如果是数组形参的话，最好是定义为指针，明确表示操纵的是指向数组元素的指针，而不是数组本身。
如果形参定义成数组形式，可以写长度  编译器会忽略。
void printValue(const int ia[10]){}


void f(const int \*){}
void f(int (&arr)[10]) 这时候编译器会检查实参和形参的长度大小


int& arr[10]
int (&arr)[10]

void printValus(int matrix[][10], int rowSize)

int main(int argc, char \*\*argv){}

可变形参
void foo(parm_list, ...);
void foo(...);




#千万不要返回局部对象的引用

返回引用的函数可以返回一个左值
```C++
char &get_val(string &str, string::size_type ix) 
{
	return str[ix];
}

int main()
{
	string s("hello");
	get_val(s, 0)  = 'H';
	cout<< s<< endl;
	return 0;
}
```

默认实参
string screenInit(string::size_type height=24,
				  string::size_type width=80,
				  char background = ''){}

默认实参只能用来替换函数调用缺少的尾部实参，即如果给width实参了，那么height必须也有实参。


指定默认实参只能一次，通常是在函数生命的时候指定默认实参

#局部对象
自动对象
在函数里面创建的局部变量 如果是未初始化的内置类型局部变量，其初值是不确定的。

静态局部对象  static 

内联函数 inline  内联函数应该放在头文件中定义

类的成员函数
函数原型必须在类中定义 但是函数体可以在类中也可以在类外定义
成员函数可以访问该类的private成员

除了static成员函数之外，每个成员函数都有一个额外的、隐含的形参this，调用成员函数时，形参this初始化为调用函数的对象的地址。

bool same_isbn(const Sales_item &rhs) const {}
const成员函数 改变了形参this的类型 指向const对象，常量成员函数不能修改调用该函数的对象

类的构造函数
构造函数的初始化列表  Sales_item():unit_sold(0), revenue(0.0) {}

构造函数的初始化列表和构造函数的函数体

编译器会合成默认构造函数

重载函数
重载只是会根据形参的个数和类型进行区分，不会根据返回类型和默认实参进行区分，const类型对于非引用形参是不区分的，但是对于引用形参是区分的 如：
int func(int * a)    int func(const int\* a)

局部声明的函数会屏蔽外层的同名函数而不是重载


重载函数的函数匹配，精确匹配优先于需要类型转换的匹配 


函数指针
bool (\*pf)(const string &, const string &)

typedef简化函数指针的定义
typedef bool (\*cmpFcn)(const string &, const string &)

函数指针形参
void useBigger(const string &, const string &, 
							   bool(const string &, const string&))
等价于
void useBigger(const string &, const string &,
							   bool (*)(const string &, const string &))

返回指向函数的指针
int (\*ff(int))(int \*, int)    ff(int)是一个函数  这个函数返回一个函数指针 int (\*)(int \*, int)

指向重载函数的指针   指针的类型必须与重载函数精确匹配





###标准IO库

istream  ostream cin  cout  cerr >> <<


         iostrea


如果函数有基类类型的引用形参时，可以给函数传递派生类型的引用对象

wchar_t读取宽字符 相应的也会有库来处理 如wiostream wfstream wstringstream

IO对象不可复制和赋值，因此函数形参和返回值不能不能为流类型，必须传递或者返回指向该对象的指针或者引用。

可以通过条件状态函数获取流的状态信息，来判断一个流是否是有效的。如  s.eof() s.bad()等
if(cin) {
	/*do something*/
}

while(cin>>word) {}
if直接检查cin流状态是否有效  而while检查条件表达式返回的流。

可以调用tie函数把输入流和输出流绑定到一起。
cin.tie(&cout)
cin.tie(0)

文件模式
in 打开文件做读操作
out 打开文件做写操作
app 在每次写之前找到文件尾
ate 打开文件后立即将文件定位到文件尾
trunc 打开文件时清空已经存在的文件流
binary 以二进制形式进行文件操作
模式是文件的属性 不是流的属性


