



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





























