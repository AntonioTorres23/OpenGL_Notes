
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
        
        void get_global_var_variables()
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
    A.get_global_var_variables();
    
    
    return 0;
}
```



So within this function we have a class called Test. Within Test we have 3 variables called `tyme`, `greeting`, and `flt`.  Then, we have a constructor that requires us to pass 3 arguments in order to create a Test object. Within those arguments we also named them the same as the 3 variables we declared previously. So within this class we have two names that are the same, we have an actual member object variable, and we have a constructor variable. How is the compiler going to know the difference between which one? It can't and it will lead to 

If you run this without 