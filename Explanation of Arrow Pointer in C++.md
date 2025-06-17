

The arrow pointer is a weird and unusual thing within C++ that over the past year has left me often confused. 

This symbol `->` refers to when you have a pointer that is pointing to an object and you want to access data from within that object. Whether it be a variable within the class or something else. 

From what I remember it's a way to simplify the usual `*` that is used to access a value that is stored within the pointer. Because if you wanted to get the value of something within a pointer object, combining that with a dot can be confusing and you have to use a specific syntax. So using the `->` is easier and is why it was developed. 


Take this source code as an example. 

`#include <iostream>


`class A
`{
    `public:
        `int num = 1;
`};


`int main() {
    `// Write C++ code here
    
    A b;
    
    A *c = nullptr;
    
    c = &b;
    std::cout << "Address:" << "\n" << std::endl;
    std::cout << &b.num << std::endl;
    std::cout << &c->num << std::endl;
    std::cout << "Value:" << "\n" << std::endl;
    std::cout << b.num << std::endl;
    std::cout << c->num << std::endl;

	`return 0;`
`}


So in this source code we made a class called `A` and within that class we have an integer variable titled num which holds the value of 1. 

Within our main source code, we create an object `B` which holds the variable num. 

We then create a `nullptr` (an empty pointer since uninitialized pointers lead to unpredictable behavior) within the variable/object `c` that uses that `A` class as a data type. Essentially telling us that it is looking for objects that are using the class `A` structure.

Then we assign our `c` object to the address of a since objects only point to addresses. 

Then we do some tests by seeing if they match the same address using the `&` character which in C++ is used for showing a variable/object's address. 

Run this part of the code and you should see that the addresses of both the num var will be the same. Because that's what we're doing, we're accessing that num var and displaying its addresses in memory. 

Since that's essentially what a pointer does in concept. It points to whatever other matching data type object/variable. Meaning it also takes the address and value of whatever pointer is pointing at. Meaning if the original variable/object's value/address its pointing at changes, it will also match that. And vice versa, if you change the value within the pointer object/variable, the original variable will change that. 

