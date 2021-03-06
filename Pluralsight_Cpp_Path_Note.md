## Directory
> [Learn How to Program with C++](#1)<br>
> [C++ Fundamentals including C++ 17](#2)<br>
> [C++ Fundamentals - Part 2](#3)<br>
> [C++11 Language Features](#4)<br>
> [C++ Unit Testing Fundamentals Using Catch](#5)<br>
> [Beautiful C++: STL Algorithms](#6)<br>

<h2 id="1">Learn How to Program with C++</h2>

### 1. const of member function  
never change value of member variables  
> ref: 7.6 2:15

<h2 id="2">C++ Fundamentals including C++ 17</h2>

### 1. default constructor
```
// only in header
Person() = default;
```
> ref: 5.6 2:08

### 2. namespace
never use namespace in header file  
> ref: 5.11 1:21

### 3. enum class
variable can repeat
```
enum class FileError
{
	notfound,
	ok
};

enum class NetworkError
{
	disconnected,
	ok
};
```
> ref: 5.14 1:59

### 4. pragma once
```
// include in header
#pragma once
```
> ref: 5.17 1:17

### 5. warning level
right-click on the project -> Properties -> C/C++ -> General -> Warning Level  
> ref: 8.5 2:29

### 6. null pointer
nullptr
> ref: 10.2 4:33

### 7. const
```
// can not change i through cri, but i is changeable
int i = 3;
int const & cri = i;
cri = 6; // not feasible
i = 6; // feasible
```
```
// can not assign a reference to a const variable
int const ci = 3;
int & ncri = ci; // not feasible
```
```
// can not past a const to a non-const reference function
int DoubleInt(int& x) { // int& x need to be const
  return x * 2;
}
int DoubleTen = DoubleInt(10); // not feasible 
```
```
// const object can not call a non-const member function
int GetNumber() {return arbitrarynumber;} // in header, function need to be const

Person const Kate("Kate", "Gregory", 234);
int KateNumber = cKate.GetNumber(); // not feasible
```
> ref: 10.6 0:00

```
// const reference: can't use the const to change the value
int const & ri = 7;
ri = 5; // not feasible
```

```
// const pointer: can't change it to point somewhere else
int * const cpi = &i;
cpi = &something; // not feasible
```

```
// pointer to a const: can't change the value of target
int const * cpi = &i;
*cpi = 2; // not feasible
```

```
// can't change it to point somewhere else or use it to change the value of target
int const * const crazy;
```
> ref: 10.7 0:09

### 8. memory management
```
// Person.h 
class Person
{
private:
  std::string firstname;
  std::string lastname;
  int id;
  Item* item;

public:
  Person(std::string first, std::string last, int number);
  ~Person();
  Person(Person const & p);
  Person& operator=(Person const & p);
  std::string GetName() const;
  void SetName(std::string first, std::string last) { firstname = first; lastname = last;}
  void distributeItem();
};
```
```
// Person.cpp
Person::Person(string first,string last, int number) : firstname(first),lastname(last), id(number), item(nullptr){}

Person::~Person() {
  delete item;
}

Person::Person(Person const & p) {
  firstname = p.firstname;
  lastname = p.lastname;
  id = p.id;
  item = new Item(p.GetName());
}

Person& Person::operator=(Person const & p) {
  if (&p == this) {
    return *this;
  }
  delete item;
  firstname = p.firstname;
  lastname = p.lastname;
  id = p.id;
  item = new Item(p.GetName());
  return *this;
}

string Person::GetName() const {
  return firstname + " " + lastname;
}

void Person::distributeItem() {
  delete item;
  item = new Item(GetName());
}
```
```
// item.h item.cpp
class Item
{
private:
  std::string ofWhom;
public:
  Item(std::string name);
  ~Item();
  std::string GetName() const {return ofWhom;}
};

Item::Item(string name) : ofWhom(name) {
  cout << "Constructor of " << ofWhom << "'s item."<< endl;
}

Item::~Item() {
  cout << "Destructor of " << ofWhom << "'s item." << endl;
}
```
```
// main.cpp
int main() {
  Person P1("X", "L", 27385522);
  P1.distributeItem(); // delete nullptr -> constructor(xl)
  P1.SetName("X", "W");
  P1.distributeItem(); // destructor(xl) -> constructor(xw)
  Person P2 = P1; // constructor(xw)
  return 0; // P2: destructor(xw) -> P1: destructor(xw)
}
```
```
// result
/*
Constructor of X L's item.
Destructor of X L's item.
Constructor of X W's item.
Constructor of X W's item.
Destructor of X W's item.
Destructor of X W's item.
*/
```
> ref: 11.5 0:00

### 9. shared_ptr
```
// Person.h 
#include <memory> // add

class Person
{
private:
  std::string firstname;
  std::string lastname;
  int id;
  // Item* item;
  std::shared_ptr<Item> item; // add

public:
  Person(std::string first, std::string last, int number);
  // ~Person();
  // Person(Person const & p);
  // Person& operator=(Person const & p);
  std::string GetName() const;
  void SetName(std::string first, std::string last) { firstname = first; lastname = last;}
  void distributeItem();
};
```
```
// Person.cpp
Person::Person(string first,string last, int number) : firstname(first),lastname(last), id(number){} // do not need item(nullptr)

// do not need destructor, copy constructor or copy assignment constructor

string Person::GetName() const {
  return firstname + " " + lastname;
}

void Person::distributeItem() {
  // delete item;
  // item = new Item(GetName());
  item.reset(); // add
  item = std::make_shared<Item>(GetName()); // add
}
```
```
// main.cpp
int main() {
  Person P1("X", "L", 27385522);
  P1.distributeItem(); // constructor(xl)
  P1.SetName("X", "W");
  P1.distributeItem(); // destructor(xl) -> constructor(xw)
  Person P2 = P1; // count++
  return 0; // P2: count-- -> P1: destructor(xw)
}
```
```
// result
/*
Constructor of X L's item.
Destructor of X L's item.
Constructor of X W's item.
Destructor of X W's item.
*/
```
> ref: 11.6 0:00

### 10. virtual
As soon as you have one virtual function, make sure you mark your destructor as virtual also
> ref: 12.4 7:43

### 11. polymorphism vs slicing
```
string f1(Person const & p) { return p.GetName(); }
string f2(Person p) { return p.GetName(); }

SocialPerson P2("X", "W", 0, "@Not Known");
string name;
name = f1(P2); // polymorphism
// X W @Not Known
name = f2(P2); // slicing
// X W
```
> ref: 12.5 4:15

<h2 id="3">C++ Fundamentals - Part 2</h2>

### 1. function pointer
```
// function pointer
typedef int(*HELPER)(int i);
HELPER h;
```
> ref: 4.3 3:33

```
// pointer to member function
typedef int (Person::*HELPER)() const;

int useFunctionPointer(Person const & p) {
  HELPER h;
  h = std::rand() % 2 ? &Person::GetId : &Person::GetRandom;
  return (p.*h)();
}

int main() {
  Person P("Xiao", "Luo", 27385522);
  for (int i = 0; i < 10; i++) {
    cout << useFunctionPointer(P) << endl;
  }
}
```
> ref: 4.4 1:16

<h2 id="4">C++11 Language Features</h2>

### 1. decltype
```
vector<int> a;
decltype(a) b;
```
> ref: 1.5 0:46

```
template<typename T1, typename T2>
auto multiple(T1 x, T2 y) -> decltype(x * y) {
  return x * y;
}
// return type can't be decltype(x * y), x and y aren't in scope yet
```
> ref: 1.6 1:00

### 2. lambda
```
[](int i) -> double {
  return i > 10 ? 0.0 : double(i);
}
```

<h2 id="5">C++ Unit Testing Fundamentals Using Catch</h2>

### 1. catch
Download [catch.hpp](https://github.com/philsquared/Catch)
```
#define CATCH_CONFIG_MAIN
#include "catch.hpp"

TEST_CASE("This is a test name") {
  REQUIRE(result == 0);
  // .....
}
```
> ref: 2.7 0:34

<h2 id="6">Beautiful C++: STL Algorithms</h2>

### 1. count & count_if
```
// all_of any_of none_of
int cnt = count_if(begin(v), end(v), [](auto elem) { return elem % 2 == 0; });
```
> ref: 3.6 3:39

### 2. find & find_if
```
// find_if_not find_end search_n adjacent_find
vector<int> v{ 18, 7, 22, 10, 690, 2 };
vector<int> primes{ 2, 3, 5, 7, 11, 13, 17, 19 };
auto res = find_first_of(begin(v), end(v), begin(primes), end(primes));

vector<int> subsequence{ 22, 10 };
res = search(begin(v), end(v), begin(subsequence), end(subsequence));
```
> ref: 3.8 2:41

### 3. equal & mismatch
```
vector<int> v1{ 18, 7, 22, 10, 69, 0, 2 };
vector<int> v2{ 18, 7, 22, 10, 690, 2 };
equal(begin(v1), end(v1), begin(v2), end(v2)); // 0

auto it = mismatch(begin(v1), end(v1), begin(v2), end(v2));
cout << *(it.first) << " " << *(it.second) << endl; // 69 690
cout << it.first - begin(v1) << endl; // 4
```
> ref: 5.3 3:32

### 4. accumulate
```
vector<int> v{ 18, 7, 22, 10, 69, 0, 2 };
int sum = accumulate(begin(v), end(v), 0, [](int sum, int curr) { return curr % 2 == 0 ? sum + curr : sum; }); // 52
```
> ref: 5.5 4:15

