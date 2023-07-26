using namespace std

Per velocizzare cin e cout usare 
```cpp
std::ios::sync_with_stdio(false);
```
### **PRINT**
```C++
std::cout << "";
```
### **CARATTERI SPECIALI**
\\n -> a capo
\\\\-> fai uno \\
\\t -> tab orizzontale
\\" -> inserisci un doppio apice

### **MATH**

|Function|Description|
|---|---|
|abs(x)|Returns the absolute value of x|
|acos(x)|Returns the arccosine of x|
|asin(x)|Returns the arcsine of x|
|atan(x)|Returns the arctangent of x|
|cbrt(x)|Returns the cube root of x|
|ceil(x)|Returns the value of x rounded up to its nearest integer|
|cos(x)|Returns the cosine of x|
|cosh(x)|Returns the hyperbolic cosine of x|
|exp(x)|Returns the value of Ex|
|expm1(x)|Returns ex -1|
|fabs(x)|Returns the absolute value of a floating x|
|fdim(x, y)|Returns the positive difference between x and y|
|floor(x)|Returns the value of x rounded down to its nearest integer|
|hypot(x, y)|Returns sqrt(x2 +y2) without intermediate overflow or underflow|
|fma(x, y, z)|Returns x*y+z without losing precision|

|   |   |
|---|---|
|fmax(x, y)|Returns the highest value of a floating x and y|
|fmin(x, y)|Returns the lowest value of a floating x and y|
|fmod(x, y)|Returns the floating point remainder of x/y|
|pow(x, y)|Returns the value of x to the power of y|
|sin(x)|Returns the sine of x (x is in radians)|
|sinh(x)|Returns the hyperbolic sine of a double value|
|tan(x)|Returns the tangent of an angle|
|tanh(x)|Returns the hyperbolic tangent of a double value|





### **OOP**
costruttore
```C++
class Car {        // The class  
  public:          // Access specifier  
    string brand;  // Attribute  
    string model;  // Attribute  
    int year;      // Attribute  
    Car(string x, string y, int z); // Constructor declaration  
};
```
In C++, there are three access specifiers:

- `public` - members are accessible from outside the class
- `private` - members cannot be accessed (or viewed) from outside the class
- `protected` - members cannot be accessed from outside the class, however, they can be accessed in inherited classes.
```C++
class Vehicle{
	public:
		stuff...;
}
class Car : public Vehicle{        // The class  
	public:          // Access specifier  
	    string brand;  // Attribute  
	    string model;  // Attribute  
	    int year;      // Attribute  
	    Car(string x, string y, int z); // Constructor declaration  
};
```


### **FILES**
Libraria ==fstream==
```C++
#include <iostream>
#include <fstream>
```

|Class|Description|
|---|---|
|`ofstream`|Creates and writes to files|
|`ifstream`|Reads from files|
|`fstream`|A combination of ofstream and ifstream: creates, reads, and writes to files|

### **ERRORI**
- The `try` statement allows you to define a block of code to be tested for errors while it is being executed.
- The `throw` keyword throws an exception when a problem is detected, which lets us create a custom error.
- The `catch` statement allows you to define a block of code to be executed, if an error occurs in the try block.


### **VECTOR**

```cpp
#include <vector>;
std::vector<int> vettore;
```

|Funzione|Description|
|---|---|
|`vettore.push_back()`|Inserisce alla fine|
|`vettore.at()`|Punta al file presnte a quell'indice|
|`vettore.pop_back`|Rimuove l'ultimo elemento del vettore|
|`vettore.size()`|returns the number of elements present in the vector|
|`vettore.clear()`|removes all the elements of the vector|
|`vettore.front()`|returns the first element of the vector|
|`vettore.back()`|returns the last element of the vector|
|`vettore.empty()`|returns **1** (true) if the vector is empty|
|`vettore.capacity()`|check the overall size of a vector|
______
Puoi creare un iterator che è come un puntatore, ma per vettori.

```cpp
vector<int>::iterator iter;
```
 |Funzione|Description|
|---|---|
|`iter = vettore.begin()`|Crea un iterator con il numero all'inizio|
|`iter = vettore.end()`|Crea un iterator con il numero alla fine|
### **STRUCTURE**

```cpp
typedef struct _STRUCTURE_NAME {
  int ID;
  int Age;
} STRUCTURE_NAME, *PSTRUCTURE_NAME;

STRUCTURE_NAME struct1 = { .ID   = 1470,  .Age  = 34};

PSTRUCTURE_NAME structpointer = &struct1; // structpointer is a pointer to the 'struct1' structure

// Updating the ID member
structpointer->ID = 8765;
printf("The structure's ID member is now : %d \n", structpointer->ID);
```
