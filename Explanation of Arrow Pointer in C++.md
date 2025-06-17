

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

Since that's essentially what a pointer does in concept. It points to whatever other matching data type object/variable. Meaning it also takes the address and value of whatever pointer is pointing at. Meaning if the original variable/object's value its pointing at changes, it will also match that. And vice versa, if you change the value within the pointer object/variable, the original variable will change that. They are one in the same and is why pointer's where kind of invented. 

The primary purpose of pointers can be for many reasons. One being you want to change the original variable/object if you are going through things like functions or other transformations outside of that variable/object's scope. Another big reason is that it typically saves a lot of memory. When you have another variable that is initialized with another variable. For example, `int a = 1;`, `int b;`, `b = a;`, you are copying over all of that information into a new variable and its stored into a new address within memory. This is fine for smaller projects but when you are working on larger applications, this can eat up a lot of memory and slow down the program. So making a pointer in most cases would be a better option. 

Now look at this source code.

`#include <iostream>


`class A
`{
    `public:
        `int num = 1;
`};


`int main() {
    `// Write C++ code here`
    
    A b;
    
    A *c = nullptr;
    
    c = &b;
    std::cout << "Address:" << "\n" << std::endl;
    std::cout << &b.num << std::endl;
    std::cout << &c->num << std::endl;
    std::cout << "Value:" << "\n" << std::endl;
    std::cout << b.num << std::endl;
    std::cout << c->num << std::endl;
    std::cout << "New Value Assigned From Pointer: 15: " << std::endl;
    c->num = 15;
    std::cout << b.num << std::endl;
    std::cout << c->num << std::endl;
    std::cout << "Address After Pointer Assigned New Value for Num Var: " <<            std::endl;
    std::cout << &b.num << std::endl;
    std::cout << &c->num << std::endl;
    return 0;
`}


All we did was change the num variable from 1 to 15 using the object pointer `c`. Now run the code and you should see that the original object `b`'s num variable also changed. As well as look at the addresses after the change. They remain the same as before we changed num's value to 15. So as you can see, `b` and pointer `c` are one in the same. 

So all in all, the `->` (arrow) pointer in C++ is a way for a pointer object to access its members that where created from a class. 
