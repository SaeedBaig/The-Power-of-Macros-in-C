# The-Power-of-Macros-in-C-
A demonstration of the capabilities of macros in C and how/when to use them

First off, what's a macro?

A macro is a sort-of-like a "find and replace" feature that C provides, and its syntax is roughly like this:
```C
#define <text-to-be-replaced> <text-to-replace-with>
```

They are typically used for defining constants, to improve readability of code. E.g.
```C
#define c 299792458 // speed of light
#define MAX_LINE_LENGTH 1000
```

They allow us to use more human-readable symbols in place of raw numbers or expressions.

***"So what's so cool about macros?***

## Save memory and time

At this point you may already be wondering... why would I use a #define when I can just use an actual const?, like this:
```C
const int c = 299792458; 
const int MAX_LINE_LENGTH = 1000;
```

The reason/s is because #defines aren't variables; they're just text to be replaced. What's more, this find-and-replace happens BEFORE compile-time by the preprcessor. So though your code may look like this:
```C
#include <stdio.h>
#define c 299792458

int main()
{
        printf("The speed of light is %dm/s.", c);
        return 0;
}
```

All the compiler sees is:
```C
#include <stdio.h>

#define c 299792458

int main()
{
        printf("The speed of light is %dm/s.", 299792458);
        return 0;
}
```

which ultimately saves memory (since no variable has to be created) and saves time (since there's no variable value to access).

However, those benefits are negligible. A cooler capability of them is to:

## Create new "keywords" and constructs
Lets admit it... "and" is more readable than "&&"; "or" is more readable than "||". If you come from a language like Python or Ruby, you may miss having actual-English keywords. But now you can have them back with macros! It's as simple as:
```C
#define and &&
#define or ||
```
Yup, macros don't have to be strictly syntactically-correct statements; they can be almost any text you want. So you can now write C code that looks like this:
```C
	if (random_mark > 50 and random_mark <= 100) {
          puts("You passed!");
	} else if (random_mark < 0 or random_mark > 100){
          puts("Error: Invalid mark");
	} else {
		      puts("You failed.");
	}
```

and it's totally valid; will compile without warning or error. Also note that the "and" in "random_num" isn't replaced. Like normal find-and-replace, macros only replace the text if it's not surrounded on either side by an alphanumeric charachter. So no need to worry there!

And that's not all. Do you miss the more expressive constructs like "until" and "unless" from languages like Ruby. Well macros can provide those too!

```C
#include <stdio.h>

#define until(condition) while (!(condition))
#define unless(condition) if (!(condition))

int main()
{
        int i = 0;
	      until (i == 10) {
	              printf("i = %d\n", i);
		            i++;
	      }

	      unless (i == 0)
                puts("Works similarily to a normal 'if' statement!");
	      return 0;
}
```
Yup, macros can take paramaters too! Macros that take paramaters act are called "function-like macros" (emphasis on the word "like" because, there are some caveats of using them compared to normal functions). Their syntax is roughly as follows:
```C
#define <macro name>(<paramater 1>, <paramater 2>, ... <paramater n>) <text-to-replace with, which may use the paramater/s>
```

Since macros don't have to expand to a syntactically-correct statement on their own (just so long as whatever it expands to makes sense in the context it's used in), macros allow you to create some truly flexible constructs. For example, here's a "foreach" loop reminiscient of Python:

```C
#include <stdio.h>

// "typeof" is a GCC extension btw; acts like the "type" function in Python
#define foreach(item, array, length) \
for (typeof(*array) *p = array, item = *p; p != array+length; p = p+1, item = *p)

int main()
{
        int numbers[] = {4, 2, 99, -3, 54};
	      foreach (number, numbers, 5) {
		            printf("number = %d\n", number);
	      }

	      return 0;
}
```

Although I personally don't usually use these constructs myself (it can be confusing for others trying to read your program), they MAY make the code more readable to you at least, so use with discretion.

But let me get back to the point about function-like macros. You may still be wondering "well, why would I use a function-like macro when I can just use an actual function?", and the answer to that is:

## Simulate type-generic functions
As you may have noticed, macros don't require you to enter specify the TYPE of its paramaters, and this can be useful for creating generic "functions" in C that work on any data type.

For example, if you've ever needed to use an absolute-value function in your C code, you may have been dissapointed to find that there's actually 2 absolute value functions; "abs" for int, and "fabs" for doubles. It just seemed so... unsatisfying to have to use different functions on different data types when both do conceptually the same thing!

However, since we know macros can take paramaters, we can now write our own absolute-value "function" like this:
```C
// Using the ternary operator
#define abs_val(x) ((x) >= 0 ? (x) : -(x))
```

abs_val will now wok on any data type, (int, float, double, you name it!) for which the ">=" and "-" operators are defined.

Note the prevlaent use of paranthases in macros (surrounding all the macro's arguments, and the macro itself) are recommended to avoid getting the wrong answer due to operator-precedence problems in weird situations. For example, take this simple cube macro:

**NOTE:** If you were wondering what's with all the seemingly-unecessary paranthases in the abs_val macro, it's because macro paramaters AREN'T evaluated before being added in the macro body, which can yield unexpected results. For example, take this simple cube macro:
```C
#define cube(x) x * x * x
int x = cube(3);
```
What do you think the value of "x" is? It's 27 (as you'd expect), since ```cube(3)``` expands to ```3 * 3 * 3```. However, what about this case?
```C
int x = cube(2 + 1)
```
Still 27? Nope! Since ```cube(2+1)``` expands to ```2 + 1 * 2 + 1 * 2 + 1```, x is actually 7 (oops). This is why enclosing your macro body (and any paramaters in the body) in paranthases is generally a good idea; to avoid these kind of operator-precedence problems (remember the emphasis on "like" in "function-like"? Some other common pitfalls to watch out for when using macros can be found [here](https://gcc.gnu.org/onlinedocs/cpp/Macro-Pitfalls.html)).

But it is possible to create more complex macros. Say, for instance, we wanted to create a sum macro that could return the sum of an array of any type (i.e. one that works on arrays of ints, doubles, floats, whatever). And what's more, we want the return type to be the SAME as the type of the array. So if it's summing an array of ints, it actually returns an int. If it's summing an array of doubles, it returns a double, etc. Something that behaves like this:

```C
// sum() accepts an array as the 1st paramater and the array's length as the 2nd paramater.
double my_array[] = {3.4, -9.8, 7.6};
printf("sum(my_array) = %lf\n", sum(my_array, 3)); // prints 1.2

int another_array[] = {3, 5, 1, 4};
printf("sum(another_array) = %d\n", sum(another_arry, 4)); // prints 13
```

Is that even possible in C? Yes it is, thanks to _Generics.

_Generic was a new feature introduced in C11 (the current standard of the C language) that allows you to make differnt selections based on the type of the argument. A more detailed explanation of _Generics can be found [here](http://www.robertgamble.net/2012/01/c11-generic-selections.html) (which I encourage you to read... it's quite a powerful feature), but the gist of it is that they sort of act like a switch statement for types. To give 1 simple example of how _Generic works:

```C
#define typename(x) _Generic((x), int: "int", double: "double", char: "char", char *: "string", default: "int")
```

Here, typename() expands to the type of it's paramater as a string (I'm using the phrase "expands to" rather than "returns" here, since it's more accurate to think of _Generic() as a macro rather than a function). So typename(42) returns "int". typename(4.2) returns "double". typename("Hello World") returns "string" and anything else whose type isn't covered returns the value of ```default``` (in this case, "int").

So what does this have to do with my summing an array? Well, in the same way that typename() expands to different strings based on the type of it's argument, a sum() macro can expand to invoke different functions based on the type of the array, like so: 
```C
#define sum(array, size) _Generic(*array, int: sumIntArray, float: sumFloatArray, default: sumDoubleArray)(array, size)
```

Where "sumIntArray", "sumFloatArray" and "sumDoubleArray" are normal functions to sum arrays of those types (examples of their implementations: 
```C
#include <stdlib.h>

int sumIntArray(const int array[], const size_t size)
{
        int total = 0;
        for (size_t i = 0; i < size; i++)
		            total += array[i];
	      return total;
}

float sumFloatArray(const float array[], const size_t size)
{
        float total = 0;
	      for (size_t i = 0; i < size; i++)
		            total += array[i];
	      return total;
}

double sumDoubleArray(const double array[], const size_t size)
{
        double total = 0;
	      for (size_t i = 0; i < size; i++)
		            total += array[i];
	      return total;
}
```

So now our sum macro will invoke the appropriate function (based on the type of the array its given) and return the appropriate type.

So now we have an ANSI-C compliant way to create "generic" functions in C! The downside of this approach is that you STILL have to write seperate functions for each data type. But at least now that can all be extracted away behinf 1 single generic "sum" macro (you can probably put those away in a seperate sum.h file and then call as needed from your own code).

