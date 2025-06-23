
The `this` pointer is a pretty unique feature that is often confusing when using C++. It can be used for many different things and applies to our LearnOpenGL code. 

The this pointer is a unique pointer that points to a non-static member function within an object. 

Take this source code for example.

```
#include <iostream>
#include <string>
#include <vector>

class Test
{
    public:
        int tyme; 
        std::string greeting;
        float flt;
        Test(int tyme, std::string greeting, float flt)
        {
            this->tyme = tyme;
            this->greeting = greeting;
            this->flt = flt;
        };
        
        void get_actual_values()
        {
            std::cout << tyme << std::endl;
            std::cout << greeting << std::endl;
            std::cout << flt << std::endl;
        }
};

std::vector<Test> test_vector
{
    
};

int main() {
    // Write C++ code here
    Test A(1, "hello", 1.5);
    test_vector.push_back(A);
    std::cout << test_vector[0].tyme << std::endl;
    std::cout << test_vector[0].greeting << std::endl;
    std::cout << test_vector[0].flt << std::endl;
    std::cout << "Vector Size: " << test_vector.size() << std::endl;
    std::cout << "Global Variables Current Values: " << std::endl;
    A.get_actual_values();
    
    
    return 0;
}
```



So within this function we have a class called Test. Within Test we have 3 members called `tyme`, `greeting`, and `flt`.  Then, we have a constructor that requires us to pass 3 arguments in order to create a Test object. Within those arguments we also named them the same as the 3 members we declared previously. So within this class we have two names that are the same, we have an actual member, and we have a constructor argument. How is the compiler going to know the difference between which one? It can't and it will lead to unusual behavior

If you run this without specifying the this pointer you end up not getting the values that you specified within the constructor. You just get weird output or even no output at all. 

Now incorporate the this pointer, within the this pointer, we are specifying that we want the value that is within the constructor argument. Because remember the `->` means we are grabbing the value of a member that is within a pointer object. So within this function we are grabbing the constructor argument via a this pointer and then assigning it to the actual member within the object itself. Meaning that whatever value that was specified within the constructor argument is now within the previously defined members. 

Now I also stored it within a vector called `test_vector` since a lot of our code within LearnOpenGL has that but within that we can index and grab the member within that specific object. And when we send it to default output, we get the same values that we entered in initially with the constructor. 

I also made a function called `get_actual_values()` which just grabs the actual members of the class and sends them to default output. Run that function and  you get the same exact output as with the ones that where grabbed previously. 

This applies to our LearnOpenGL code since the `model.h` file includes a class called `Model` in which you have actual members defined like the code previously that are the same name as the constructors. So, within the constructor you have to use the this pointer to differentiate between the constructor arguments and the actual members within the class. This is so you can assign the values within the constructor arguments to the actual members within the class.