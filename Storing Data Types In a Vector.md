
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



Pretty simple, we create a struct called hello and assign different variables/data types to it. Then we create a vector that specifically holds that data type called `helvec`.  We just make it empty for now so 