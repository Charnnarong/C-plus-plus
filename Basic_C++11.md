# Lambda in C++ 11

แลมดา คือการเขียน anonymous function แบบ inline. ทำให้โค๊ดอ่านเข้าใจได้ง่ายมากยิ่งขึ้น

#### Syntax

```cpp
	[](){}
```
[]  คือ lambda capture block. 

ตัวอย่าง 

ถ้าไม่มีพารามิเตอร์สามารถไม่ใส่วงเล็บก็ได้ หรือจะใส่ก็ได้
```cpp
	auto notify = []{std::cout<<"Hello C++11 Lambda"<<std::endl;};
	notify();
```

ถ้าต้องการใส่พารามิเตอร์ให้แลมดา

```cpp
	auto add = [](int a, int b){std::cout<<a + b <<std::endl;};
	add(5,6);
```

ถ้าต้องการให้แลมดารีเทิร์นค่ากลับคืนมา ก็ระบุชนิด return type ที่หลังวงเล็บ

```cpp
	auto power = [](int a)->int{return a*a;};
	cout<<power(7);
```

แต่ถ้าไม่ระบบุว่าต้องการให้รีเทิร์นค่าแบบใด คอมไพเลอร์จะทำการจะพยายามหาว่าจะต้องคืนค่าชนิดใด ( duduction process)

```cpp
	auto mul = [](double a , int b){return a * b;}; //return as double;
	cout<<sizeof(mul(3,4))<<endl; 
```

[] แสดงว่าแลมดามีการเก็บค่าบางอย่างในบล๊อกสโคปทีมันอยู่เอาเข้าไปด้วย

```cpp
	string name = "Hello Lambda";
	auto info = [name](){cout<<name<<endl;};
	info();
```

หมาความว่า name สามารถเห็นได้เมื่อมองจากบอดี้ของแลมดาเอง ถ้า [] ไม่มีอะไรใส่เข้าไปคือไม่มีการแคปเจอร์ค่าใดๆทั้งสิ้น แต่อย่างตัวอย่างข้างบน คือ name จะทำการแคปแบบ by value

#### การทำงาน

จากโค๊ดตัวอย่างนี้ 

```cpp
	int main()
	{
		string name = "Hello Lambda";
		auto info = [name](){cout<<name<<endl;};
		info();

		return 0;
	}

```

คอมไพเลอร์ทำการเปลี่ยนแลมดาไปเป็นฟังก์ชั่นอะไรอย่างหนึ่ง เช่นใน main ฟังก์ชั่น เมื่อคอมไพล์ก็จะได้แบบนี้

```c++-objdump
int main()
{
   100401125:	55                   	push   %rbp
   100401126:	53                   	push   %rbx
   100401127:	48 83 ec 48          	sub    $0x48,%rsp
   10040112b:	48 8d ac 24 80 00 00 	lea    0x80(%rsp),%rbp
   100401132:	00 
   100401133:	e8 f8 01 00 00       	callq  100401330 <__main>


	string name = "Hello Lambda";
   100401138:	48 8d 45 bf          	lea    -0x41(%rbp),%rax
   10040113c:	48 89 c1             	mov    %rax,%rcx
   10040113f:	e8 3c 01 00 00       	callq  100401280 <std::allocator<char>::allocator()>
   100401144:	48 8d 55 bf          	lea    -0x41(%rbp),%rdx
   100401148:	48 8d 45 b0          	lea    -0x50(%rbp),%rax
   10040114c:	49 89 d0             	mov    %rdx,%r8
   10040114f:	48 8d 15 db 1e 00 00 	lea    0x1edb(%rip),%rdx        # 100403031 <std::piecewise_construct+0x1>
   100401156:	48 89 c1             	mov    %rax,%rcx
   100401159:	e8 2a 01 00 00       	callq  100401288 <std::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string(char const*, std::allocator<char> const&)>
   10040115e:	48 8d 45 bf          	lea    -0x41(%rbp),%rax
   100401162:	48 89 c1             	mov    %rax,%rcx
   100401165:	e8 26 01 00 00       	callq  100401290 <std::allocator<char>::~allocator()>
	auto info = [name]()
	{cout<<name<<endl;};
   10040116a:	48 8d 55 b0          	lea    -0x50(%rbp),%rdx
   10040116e:	48 8d 45 a0          	lea    -0x60(%rbp),%rax
   100401172:	48 89 c1             	mov    %rax,%rcx
   100401175:	e8 1e 01 00 00       	callq  100401298 <std::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string(std::string const&)>
	info();
   10040117a:	48 8d 45 a0          	lea    -0x60(%rbp),%rax
   10040117e:	48 89 c1             	mov    %rax,%rcx
   100401181:	e8 4a ff ff ff       	callq  1004010d0 <main::{lambda()#1}::operator()() const>

	return 0;
   100401186:	bb 00 00 00 00       	mov    $0x0,%ebx
```

จะเห็นว่าก่อน `return 0;` จะมีการเรียกแลมดาขึ้นมาใช้งาน ซึ่งแลมดานั้นจะถูกสร้างเช่น

```c++-objdump

00000001004010d0 <main::{lambda()#1}::operator()() const>:
int main()
{


	string name = "Hello Lambda";
	auto info = [name]()
   1004010d0:	55                   	push   %rbp
   1004010d1:	48 89 e5             	mov    %rsp,%rbp
   1004010d4:	48 83 ec 20          	sub    $0x20,%rsp
   1004010d8:	48 89 4d 10          	mov    %rcx,0x10(%rbp)
	{cout<<name<<endl;};
   1004010dc:	48 8b 45 10          	mov    0x10(%rbp),%rax
   1004010e0:	48 89 c2             	mov    %rax,%rdx
   1004010e3:	48 8b 0d 76 1f 00 00 	mov    0x1f76(%rip),%rcx        # 100403060 <__fu0__ZSt4cout>
   1004010ea:	e8 79 01 00 00       	callq  100401268 <std::basic_ostream<char, std::char_traits<char> >& std::operator<< <char, std::char_traits<char>, std::allocator<char> >(std::basic_ostream<char, std::char_traits<char> >&, std::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)>
   1004010ef:	48 8b 15 7a 1f 00 00 	mov    0x1f7a(%rip),%rdx        # 100403070 <.refptr._ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_>
   1004010f6:	48 89 c1             	mov    %rax,%rcx
   1004010f9:	e8 72 01 00 00       	callq  100401270 <std::ostream::operator<<(std::ostream& (*)(std::ostream&))>
   1004010fe:	90                   	nop
   1004010ff:	48 83 c4 20          	add    $0x20,%rsp
   100401103:	5d                   	pop    %rbp
   100401104:	c3                   	retq   
   100401105:	90                   	nop

```

ค่าที่แคปเจอร์มาจะถูกมาสร้างเป็น data member ของฟังก์ช่นนั้นๆ เช่น `string name = "Hello Lambda";` ที่อยู่ใน `<main::{lambda()#1}::operator()() const>`

จาก main สมมุติเราประกาศ `string name = "Hello Lambda";` เป็น `const string name = "Hello Lambda";` ฟังก์ชั่นที่ถูกสร้างจากแลมดาก็จะทำการแคปเจอรแบบ const ด้วยเช่นกัน นั่นก็คือ `const string name = "Hello Lambda";` 

และถ้าต้องการให้ค่าที่แคปเจอร์เข้ามาสามารถเปลี่ยนค่าได้ก็ใช้คีย์เวิร์ด mutable หลังวงเล็บ

```cpp
	int main()
	{
		string name = "Hello Lambda";
		auto info = [name]()mutable{name ="I have changed"; cout<<name<<endl;};
		info();
		cout<<name<<endl;

		return 0;
	}

```

แม้ว่าแลมดาจะสามารถเปลี่ยนค่าของ `name` ได้แต่ว่า บรรทัดสุดท้าย `cout<<name<<endl;` ก็ไม่ได้เปลี่ยนค่าของ name แต่อย่างใด เพราะแลมดาไปทำการเปลี่ยนค่าที่แคปเจอร์เข้ามา

และถ้าต้องการเปลี่่ยนค่าจริงๆที่แคปเจอร์มาก็ให้แคปแบบ reference 

```cpp
	int main()
	{
		string name = "Hello Lambda";
		auto info = [&name]()mutable{name ="I have changed"; cout<<name<<endl;};
		info();
		cout<<name<<endl;

		return 0;
	}

```

ถ้าแคปแบบ reference แล้วก็ไม่ต้องใช้ mutable ก็ได้

```cpp
	auto info = [&name](){name ="I have changed"; cout<<name<<endl;};
```

นอกจากนี้เรายังสามารถแคปเจอร์ค่าทั้งหมดในบล๊อกที่มันอยู่ได้ด้วย 

###### แบบ 1 แคปทั้งหมด by value [=]
```cpp
	int main()
	{
		string name = "Hello Lambda";
		int age = 12;
		double gpa = 92.3;

		auto info = [=]()
		{	cout<<name<<endl;
			cout<<age<<endl;
			cout<<gpa<<endl;};

		info();
		cout<<"----after lambda execution---\n";
		cout << name << endl;
		cout << age << endl;
		cout << gpa << endl;

		return 0;
	}
```

###### แบบ 2 แคปทั้งหมด by reference [&]

```cpp
	int main()
	{
		string name = "Hello Lambda";
		int age = 12;
		double gpa = 92.3;

		auto info = [&]()
		{	name = "changed";
			age = 24;
			gpa = 49.99;};

		info();
		cout << "----after lambda execution---\n";
		cout << name << endl;
		cout << age << endl;
		cout << gpa << endl;

		return 0;
	}

```

นอกจากนี้ยังสามารถแคปแบบเลือกเอาว่าจะเลือกตัวไหนเป็นแบบ by value หรือ by reference ที่เหลือก็แคปแบบ default ซึ่งค่า default นี้มีได้สองแบบคือ & กับ = . ให้ใส่ค่า default ไว้ข้างหน้าสุด ใน[] เช่น

แคปทุกอย่างเป็น reference ยกเว้น age ให้เป็นแบบ by value

```cpp
	auto info = [&, age]()
	{	name = "changed";
		//age = 24;
		gpa = 49.99;};
```

แคปทุกอย่างเป็น by value ส่วน gpa เป็นแบบ reference

```cpp
	auto info = [=, &gpa]()
	{	//name = "changed";
		//age = 24;
		gpa = 49.99;};
```

#### Syntax ต่อ ref 

```cpp

	[]()->{}
	[ capture-list ] ( params ) mutable(optional) exception attribute -> ret { body }	
	
```
[http://en.cppreference.com/w/cpp/language/lambda](http://en.cppreference.com/w/cpp/language/lambda)


#### ไม่ต้องการ return ค่า int ไม่ต้องการ return ค่า double , string , float etc. แต่ต้องให้ได lambda กลับคืนมา ?


ก่อนอื่นก็เขียนโปรแกรมง่ายๆกันก่อน

```cpp
#include <iostream>

using namespace std;

int squareIt(int x)
{
	return [x]{return x*x;}();
}

int main()
{

	cout << squareIt(4);

	return 0;
}

```

ถ้าเราไม่ทำการ execute แลมดาใน squareIt เราก็จะได้

```cpp
	int squareIt(int x)
	{
		return [x]{return x*x;};
	}
```

ซึ่งมันผิด เพราะ `return [x]{return x*x;};` ไม่ใช่ค่า int แต่เป็นแลมดา เรารู้ล่ะว่ามันรับ int เข้าไป และก็รีเทิร์น int ออกมา ดังนั้นสามารถเขียนใหม่ได้เป็น


```cpp
	#include <iostream>
	#include <functional>
	using namespace std;

	function<int(void)> squareIt(int x)
	{
		return [x]{return x*x;};
	}


	int main()
	{

		auto sqit = squareIt(5);
		cout<<sqit();

		return 0;
	}

```

`function<int(void)>` return ตัว lambda ออกมา นั้นก็คือ  `return [x]{return x*x;};` ซึ่งแลมดานี้ ไม่รับอะไรเลย แต่คืนค่า int ออกมา `return [x]{return x*x;};` 


#### ฟังก์ชั่นที่รับ lambda เข้าไปเพื่อเอาไปใช้ทีหลัง หรือเรียนกว่าการเรียกกลับ หรือ callback

```cpp
	#include <iostream>
	#include <functional>
	using namespace std;

	//template <class T& , class R& , class L& >
	int arithematic (int a, int b , const function<int(int ,int )> &callback)
	{
		return callback(a,b);
	}


	int main()
	{

		auto plus = [](int a,int b)->int{return a+b;};
		auto minus = [](int a,int b){return a-b;};

		cout<<arithematic(1,2,plus)<<endl;
		cout<<arithematic(3,10,minus)<<endl;
		cout<<arithematic(4,5,[](int a , int b)->int{return a * b;});

		return 0;
	}


```

##### ตัวอย่างทีทำงานกับ STL Algorithm

```cpp
	#include <iostream>
	#include <vector>
	#include <algorithm>
	using namespace std;

	int main()
	{
		vector<int> x
		{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

		int greatherThanTwo = count_if(x.begin(),x.end(),[](int x_item){	return x_item > 2;});
		cout << greatherThanTwo << endl;

		return 0;
	}

```

## References

[http://en.cppreference.com/w/cpp/language/lambda](http://en.cppreference.com/w/cpp/language/lambda)
[http://en.cppreference.com/w/cpp/utility/functional/function](http://en.cppreference.com/w/cpp/utility/functional/function)
[http://stackoverflow.com/questions/3867276/can-the-type-of-a-lambda-expression-be-expressed](http://stackoverflow.com/questions/3867276/can-the-type-of-a-lambda-expression-be-expressed)
[http://stackoverflow.com/questions/9620098/explicit-return-type-of-lambda](http://stackoverflow.com/questions/9620098/explicit-return-type-of-lambda)


