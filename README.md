# The Power of Macros (in C)
### A demonstration of the capabilities of macros in C and how/when to use them

First off, what's a macro?

A macro is sort of like a "find and replace" feature that C provides. Its syntax is roughly like this:
```C
#define <text-to-be-replaced> <text-to-replace-with>
```

They are typically used for defining constants. E.g.
```C
#define c 299792458 // speed of light
#define MAX_LINE_LENGTH 1000
```

They allow us to use more human-readable symbols in place of raw numbers or expressions.

### ***"So what's so cool about macros?"***

## Save memory and time

You may already be wondering _"why would I use a #define when I can just use an actual const?"_, like this:
```C
const int c = 299792458; 
const int MAX_LINE_LENGTH = 1000;
```

The reason/s is because #defines aren't variables; they're just text to be replaced. What's more, this find-and-replace happens BEFORE compile-time by the preprcessor. So if you use ```#define c 299792458```, though your code may look like this:
```C
printf("The speed of light is %dm/s.\n", c);
```

All the compiler sees is:
```C
printf("The speed of light is %dm/s.\n", 299792458);
```
which ultimately makes your code more memory-efficient (since no variable has to be created) and time-efficient (since there's no variable value to access; the compiler already sees the value).

Though that's a negligible benefit of macros. One of their more cooler capabilities is to:

## Create new "keywords" and constructs
Lets admit it... ```and``` is more readable than ```&&```; ```or``` is more readable than ```||```. If you come from any language with ```and``` and ```or```, you may miss having actual-English keywords. But now you can have them back with macros! It's as simple as:
```C
#define and &&
#define or ||
```
So you can now write C code that looks like this:
```C
if (random_mark > 50 and random_mark <= 100)
    printf("You passed!\n");
else if (random_mark < 0 or random_mark > 100)
    printf("Error: Invalid mark\n");
else
    printf("You failed.\n");
```

and it's totally valid; will compile without warning or error. Also note that the "and" in "random_num" isn't replaced. Like normal find-and-replace, macros only replace the text if it's not surrounded on either side by an alphanumeric charachter.

And that's not all. Do you miss the more expressive constructs like ```until``` and ```unless``` from languages like Ruby? Well macros can provide those too!
```C
#define until(condition) while (!(condition))
#define unless(condition) if (!(condition))
```
Yup, macros can take paramaters! So now you can write code like this:
```C
int i = 0;
until (i == 10) {
    printf("i = %d\n", i);
    i++;
}

unless (i == 0)
    printf("Works similarily to the 'unless' statements in Ruby!\n");
```

Macros that take paramaters are called "function-like macros" (emphasis on "like", since there are some caveats of using them compared to normal functions). Their syntax is roughly as follows:
```C
#define <macro-name>(<paramater-1>, <paramater-2>, ... <paramater-n>) <text-to-replace-with,-which-may-use-the-paramater/s>
```

Since macros don't have to expand to a syntactically-correct statement on their own (just so long as whatever it expands to makes sense in the context it's used in), macros allow you to create some truly flexible constructs. For example, here's a "foreach" loop reminiscient of Python:

```C
#include <stdio.h>

// "typeof" is a GCC extension btw; acts like the "type" function in Python
#define each(item, array, length) \
    (typeof(*(array)) *p = (array), (item) = *p; p != (array)+(length); p = p+1, (item) = *p)

int main()
{
    int numbers[] = {4, 2, 99, -3, 54};
    for each (number, numbers, 5)
	printf("number = %d\n", number);

    return 0;
}
```

Although I don't usually use these constructs myself (it can be confusing for others trying to read your program), they MAY make the code more readable to you at least, so use with discretion.

But let me get back to the point about function-like macros, because that's where the REAL power of macros comes from.

## Simulate type-generic functions
As you may have noticed, macros don't require you to specify the type of their paramaters. Since C doesn't have templates or method overloading, macros are the closest thing you can get to having generic functions that work on any data type.

For example, if you've ever needed to use the absolute-value function in your C code, you may have been dissapointed to find that there's actually 2 absolute value functions; ```abs``` for int, and ```fabs``` for doubles. It just seems so... unsatisfying to have to use different functions for the same thing!

However, we can write a generic absolute-value "function" like this:
```C
// Using the ternary operator
#define absolute_value(x) ((x) >= 0 ? (x) : -(x))
```

```absolute_value``` will now wok on any data type (int, float, double, you name it!) for which the ```>=``` and ```-``` operators are defined.

**NOTE:** If you were wondering what's with all the seemingly-unecessary paranthases in the absolute_value macro, it's because macro paramaters AREN'T evaluated before being added in the macro body, which can yield unexpected results. For example, take this simple cube macro:
```C
#define cube(x) x * x * x
int x = cube(3);
```
What do you think the value of ```x``` is? It's 27 (as you'd expect), since ```cube(3)``` expands to ```3 * 3 * 3```. However, what about this case?
```C
int x = cube(2 + 1)
```
Still 27? Nope! Since ```cube(2+1)``` expands to ```2 + 1 * 2 + 1 * 2 + 1```, ```x``` is actually 7 (oops). That's why it's generally a good idea to enclose your macro body (and any paramaters in the body) in paranthases to avoid these kind of operator-precedence problems (some other common pitfalls to watch out for when using macros can be found [here](https://gcc.gnu.org/onlinedocs/cpp/Macro-Pitfalls.html)).

But it is possible to create more complex macros. Say, for instance, we wanted to create a ```sum(array, length)``` macro that could return the sum of an array of any type (i.e. one that works on arrays of ints, doubles, floats, whatever). And what's more, we want the return type to be the SAME as the type of the array. So if it's summing an array of ints, it actually returns an int. If it's summing an array of doubles, it returns a double, etc. Something that behaves like this:

```C
// sum() accepts an array as the 1st paramater and the array's length as the 2nd paramater.
double my_array[] = {3.4, -9.8, 7.6};
printf("sum(my_array) = %lf\n", sum(my_array, 3)); // prints 1.2

int another_array[] = {3, 5, 1, 4};
printf("sum(another_array) = %d\n", sum(another_arry, 4)); // prints 13
```

Is that even possible in C? Yes it is, thanks to _Generics.

```_Generic``` is a feature introduced in C11 (the current standard of the C language) that allows you to make differnt selections based on the type of the argument. A more detailed explanation of _Generics can be found [here](http://www.robertgamble.net/2012/01/c11-generic-selections.html) (which I encourage you to read... it's quite an interesting feature), but the gist of it is that they sort of act like a switch statement for types. To give 1 simple example of how ```_Generic``` works:

```C
#define typename(x) _Generic((x), int: "int", double: "double", char: "char", char *: "string", default: "int")
```

Here, ```typename(x)``` expands to the type of ```x``` as a string (I'm using the phrase "expands to" rather than "returns" here since ```_Generic``` is more like a macro than a function). So ```typename(42)``` expands to ```"int"```, ```typename(4.2)``` expands to ```"double"```, ```typename("Hello World")``` expands to ```"string"```, and anything else whose type isn't covered expands to the value given by ```default``` (in this case, ```"int"```).

So what does this have to do with summing an array? Well, in the same way that ```typename(x)``` expands to different strings based on the type of ```x```, a ```sum(array, length)``` macro can expand to invoke different functions based on the type of ```array```, like so: 
```C
#define sum(array, length) \
    _Generic(*(array), int: sumIntArray, float: sumFloatArray, default: sumDoubleArray)((array), (length))
```

Where ```sumIntArray```, ```sumFloatArray``` and ```sumDoubleArray``` are normal functions to sum arrays of those types (examples of their implementations:)
```C
#include <stdlib.h>

int sumIntArray(const int array[], const size_t length)
{
    int total = 0;
    for (size_t i = 0; i < length; i++)
            total += array[i];
    return total;
}

float sumFloatArray(const float array[], const size_t length)
{
    float total = 0;
    for (size_t i = 0; i < length; i++)
            total += array[i];
    return total;
}

double sumDoubleArray(const double array[], const size_t length)
{
    double total = 0;
    for (size_t i = 0; i < length; i++)
	    total += array[i];
    return total;
}
```

So now our sum macro will invoke the appropriate function (based on the type of the array its given) and return the appropriate type.

Once you've got the hang of creating macros with ```_Generic```, it becomes trivial to create generic macros for other common operations as well (e.g. to return the maximum value in an array).
```C
#define max(array, length) \
    _Generic(*(array), int: maxIntArray, float: maxFloatArray, default: maxDoubleArray)((array), (length))
```

You can also include _Generics inside _Generics. For example, here's a generic power macro that returns an int when both of its arguments are ints and a double otherwise:
```C
#include <math.h>

#define power(x, y) \
    _Generic((x), int: _Generic((y), int: (int) pow(x,y), default: pow(x,y)), default: pow(x,y))
```
which can be useful if precision is a concern and you'd rather use ints when the answer is supposed to be an int and double when it's actually a double.

We now have an ANSI-C compliant way to create generic "functions" in C! The downside of this approach is that you STILL have to write seperate functions for each data type. But at least now that can all be abstracted away behind a single macro (you can put those all away in a seperate sum.h file and then ```#include``` it when needed).
