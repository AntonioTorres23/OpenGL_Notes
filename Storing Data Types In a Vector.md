
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

So as we can see, you can store those structure objects we created previously and add them to a vector that is created with that same structure as its type. As well as index it to access a specific one in particular and 