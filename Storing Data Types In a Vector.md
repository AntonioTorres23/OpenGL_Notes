
Now vectors are quite interesting because they are so versatile when it comes to what it can store. C++ vectors can store things that you previously though where not possible with something such as an array. For example, you can store objects and structures within a vector. 

Take this source code for example

```
// Online C++ compiler to run C++ program online
#include <iostream>
#include <string>
#include <vector>

using namespace std; 


struct hello {
  int a = 0;
  std::string hello = "hello";
};

std::vector<hello> helvec {
    
};

int main() {
    
    hello a;
    hello b;
    hello c;
    cout << a.hello << std::endl;
    helvec.push_back(a);
    helvec.push_back(b);
    helvec.push_back(c);
    unsigned int siz = helvec.size();
    cout << siz << endl;
    cout << helvec[1].hello << endl;
    
    return 0;
}
```



Pretty simple, we create a struct called hello and assign different variables/data types to it. Then we create a vector that specifically holds that data type called `helvec`.  We just make it empty for now within the global scope. 

Within the main function though, we create 3 variables/objects that hold the hello structure, a, b, and c. 

After this we add the 3 objects using the `push_back` built in function to the `helvec` vector. 

Next we grab the size of the vector to show 3 of the structures are held within it. This adds up to our `push_back` function that we did previously since we added 3 of the hello objects we created previously to that vector. 

Lastly, we use indexing to go to the second object stored in the vector (b) and access the hello variable which contains the string hello and display it to default output using the built-in `cout` function. 

So as we can see, you can store those structure objects we created previously and add them to a vector that is created with that same structure as its type. As well as index it to access a specific one in particular and retrieve its members. 

Now structures are the same as classes in a way, but here is a version of this code with class implementation. 


```
#include <iostream>
#include <string>
#include <vector>

class Test
{
    public:
    
        int integer = 1;
        std::string greeting = "hello";
        float flt = 2.5;
};

std::vector<Test> test_vec 
{
    
};

int main() {
    // Write C++ code here
    Test A;
    Test B;
    B.integer = 2;
    B.greeting = "goodbye";
    B.flt = 3.2;
    test_vec.push_back(A);
    test_vec.push_back(B);
    std::cout << "Size Of Vector: " << test_vec.size() << std::endl;
    std::cout << "Integer Class Member Stored Within Vector: " << std::endl;
    std::cout << "Object A: " << test_vec[0].integer << std::endl;
    std::cout << "Object B: " << test_vec[1].integer << std::endl;
    std::cout << "String Class Member Stored Within Vector: " << std::endl;
    std::cout << "Object A: " << test_vec[0].greeting << std::endl;
    std::cout << "Object B: " << test_vec[1].greeting << std::endl;
    std::cout << "Float Class Member Stored Within Vector: " << std::endl;
    std::cout << "Object A: " << test_vec[0].flt << std::endl;
    std::cout << "Object B: " << test_vec[1].flt << std::endl;

    return 0;
}
```

So in this program we create a class called Test. Within test there are members of various datatypes. We create a vector that takes in the type Test called `test_vec` and it is empty upon its initialization. 

In the main function, we create two Test objects called A, and B. We then change the members in B prior to pushing it back as to differentiate between the two when we index them in the vector. Then we use the `push_back` function to put our 2 Test objects in the `test_vec` vector. 

Next we do a bunch of indexing of each object and seeing what values are in each member. Remember we pushed A first so A will be 0 and B will be 1. If you run this you will see the differences in them and how we can access the members within the vector. 

As you can see vectors can store some pretty complex things. Making them super versatile. 

How does this apply to OpenGL. Much of our model and mesh header files happen to contain vectors that house structures within a vector, allowing us to store many different items and access them within one place. For example, you may have a Vertex structure that holds things like vertex positions, normals, and texture coordinates. But you have multiple 