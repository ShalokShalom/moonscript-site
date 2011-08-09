target: reference/index
template: reference
--
MoonScript is a programming language that compiles to
[Lua](http://www.lua.org). This guide expects the reader to have basic
familiarity with Lua. For each code snippet below, the MoonScript is on the
left and the compiled Lua is on right right.

## Assignment

Unlike Lua, there is no `local` keyword. All assignments to names that are not
already defined will be declared as local to the scope of that declaration. If
you wish to create a global variable it must be done using the `export`
keyword.

    hello = "world"
    a,b,c = 1, 2, 3


## Update Assignment

`+=`, `-=`, `/=`, `*=`, `%=` operators have been added for updating a value by
a certain amount have been added. They are aliases for their expanded
equivalents.

    x = 0
    x += 10

## Comments 

Like Lua, comments start with `--` and continue to the end of the line.
Comments are not written to the output.

    -- I am a comment

## Literals & Operators

MoonScript supports all the same primitive literals as Lua and uses the same
syntax. This applies to numbers, strings, booleans, and `nil`.

MoonScript also supports all the same binary and unary operators. Additionally
`!=` is as an alias for `~=`.

## Function Literals

All functions are created using a function expression. A simple function is
denoted using the arrow: `->`

    my_function = ->
    my_function() -- does nothing


The body of the function can either by one statement placed directly after the
arrow, or it can be a series of statements indented on the following line:

    func_a = -> print "hello world"

    func_b = ->
      value = 100
      print "The value:", value

If a function has no arguments, it can be called using the `!` operator,
instead of empty parentheses. For example, calling the two functions from
above:

    func_a!
    func_b!

Functions with arguments can be created by preceding the arrow with a list of
argument names in parentheses:

    sum = (x, y) -> print "sum", x + y

Functions can be called by listing the values of the arguments after the name
of the variable where the function is stored. When chaining together function
calls, the arguments are applied to the closest function to the left.

    sum 10, 20
	print sum 10, 20

	a b c "a", "b", "c"

In order to avoid ambiguity in when calling functions, parentheses can also be
used to surround the arguments. This is required here in order to make sure the
right arguments get sent to the right functions.

    print "sum 1:", sum(10, 20), "sum 1:", sum(30, 40)

Functions will coerce the last statement in their body into a return statement,
this is called implicit return:

    sum = (x, y) -> x + y
    print "The sum is ", sum 10, 20

And if you need to explicitly return, you can use the `return` keyword:

    sum = (x, y) -> return x + y

Just like in Lua, functions can return multiple values. The last statement must
be a list of values separated by commas:
    
    mystery = (x, y) -> x + y, x - y
    a,b = mystery 10, 20

### Fat Arrows

Because it is an idiom in Lua to send the object as the first argument when
calling a method, a special syntax is provided for functions which
automatically includes a `self` argument.

    func = (num) => self.value + num

### Argument Defaults

It is possible to provide deafult values for the arguments of a function. An
argument is determined to be empty if it's value is `nil`. Any `nil` arguments
that have a default value will be replace before the body of the function is run.

    my_function = (name="something", height=100) ->
      print "Hello I am", name
      print "My height is", height

An argument default value expression is evaluated in the body of the function
in the order of the argument declarations. For this reason default values have
access to previously declared arguments.

    some_args = (x=100, y=x+1000) ->
      print x + y

### Considerations

Because of the expressive parentheses-less way of calling functions, some
restrictions must be put in place to avoid parsing ambiguity involving
whitespace.

The minus sign plays two roles, a unary negation operator and a binary
subtraction operator. In order to force subtraction a space must be placed
after the `-` operator. In order to force a negation, no space must follow
the `-`. Consider the examples below.

    a = x - 10
    b = x-10
    c = x -y

The precedence of the first argument of a function call can also be controlled
using whitespace if the argument is a literal string.In Lua, it is common to
leave off parentheses when calling a function with a single string literal.

When there is no space between a variable and a string literal, the
function call takes precedence over any following expressions. No other
arguments can be passed to the function when it is called this way.

Where there is a space following a variable and a string literal, the function
call acts as show above. The string literal belongs to any following
expressions (if they exist), which serves as the argument list.

    x = func"hello" + 100
    y = func "hello" + 100

## Table Literals

Like in Lua, tables are delimited in curly braces.

    some_values = { 1, 2, 3, 4 }

Unlike Lua, assigning a value to a key in a table is done with `:` (instead of
`=`).

    some_values = {
      name: "Bill",
      age: 200,
      ["favorite food"]: "rice"
    }

The curly braces can be left off if a single table is being assigned.

    profile = 
      height: "4 feet",
      shoe_size: 13,
      favorite_foods: {"ice cream", "donuts"}

Newlines can be used to delimit values instead of a comma (or both):

    values = {
      1,2,3,4
      5,6,7,8
      name: "superman"
      occupation: "crime fighting"
    }

When creating a single line table literal, the curly braces can also be left
off:

    my_function dance: "Tango", partner: "none"

    y = type: "dog", legs: 4, tails: 1

## Table Comprehensions

Table comprehensions provide a quick way to iterate over a table's values while
applying a statement and accumulating the result.

The following creates a copy of the `items` table but with all the values
doubled.

    items = { 1, 2, 3, 4 }
    doubled = [item * 2 for i, item in ipairs items]

The items included in the new table can be restricted with a `when` clause:

    slice = [item for i, item in ipairs items when i > 1 and i < 3]

Because it is common to iterate over the values of a numerically indexed table,
an `*` operator is introduced. The doubled example can be rewritten as:

    doubled = [item * 2 for item in *items]

The `for` and `when` clauses can be chained as much as desired. The only
requirement is that a comprehension has at least one `for` clause.

Using multiple `for` clauses is the same as using nested loops:

    x_coords = {4, 5, 6, 7}
    y_coords = {9, 2, 3}

    points = [{x,y} for x in *x_coords for y in *y_coords]

### Slicing

A special syntax is provided to restrict the items that are iterated over when
using the `*` operator. This is equivalent to setting the iteration bounds and
a step size in a `for` loop.

Here we can set the minimum and maximum bounds, taking all items with indexes
between 1 and 5 inclusive:
    
    slice = [item for item in *items[1:5]]

Any of the slice arguments can be left off to use a sensible default. In this
example, if the max index is left off it defaults to the length of the table.
This will take everything but the first element:

    slice = [item for item in *items[1:]]

If the minimum bound is left out, it defaults to 1. Here we only provide a step
size and leave the other bounds blank. This takes all odd indexed items: (1, 3,
5, ...)

    slice = [item for items in *items[::2]]

## For Loop

There are two for loop forms, just like in Lua. A numeric one and a generic one:

    for i = 10, 20
      print i

    for k = 1,15,2 -- an optional step provided
      print k 

    for key, value in pairs object
      print key, value
    
The slicing and `*` operators can be used, just like with table comprehensions:

    for item in *items[2:4]
      print item

A shorter syntax is also available for all variations when the body is only a
single line:

    for item in *items do print item

    for j = 1,10,3 do print j

A for loop can also be used an expression. The last statement in the body of
the for loop is coerced into an expression and appended to an accumulating
table if the value of that expression is not nil.

Doubling every even number:

    doubled_evens = for i=1,20
      if i % 2 == 0
        i * 2
      else
        i

Filtering out odd numbers:
    
    my_numbers = {1,2,3,4,5,6}
    odds = for x in *my_numbers
      if x % 2 == 1 then x

For loops at the end of a function body are not accumulated into a table for a
return value (Instead the function will return `nil`).  Either an explicit
`return` statement can be used, or the loop can be converted into a list
comprehension.

    func_a -> for i=1,10 do i
    func_b -> return for i=1,10 do i

    print func_a! -- prints nil
    print func_b! -- prints table object

This is done to avoid the needless creation of tables for functions that don't
need to return the results of the loop.

## While Loop

The while loop also comes in two variations:

    i = 10
    while i > 0
      print i
      i -= 1

    while running == true do my_function!

Like for loops, the while loop can also be used an expression. Additionally,
for a function to return the accumlated value of a while loop, the statement
must be explicitly returned.

## Conditionals

    have_coins = false
    if have_coins
      print "Got coins"
	else
      print "No coins"

A short syntax for single statements can also be used:

	have_coins = false
    if have_coins then print "Got coins" else print "No coins"


Because if statements can be used as expressions, this can able be written as:

	have_coins = false
	print if have_coins then "Got coins" else "No coins"

Conditionals can also be used in return statements and assignments:

    is_tall = (name) ->
      if name == "Rob"
        true
      else
        false

    message = if is_tall "Rob"
      "I am very tall"
    else
      "I am not so tall"

    print message -- prints: I am very tall


## Line Decorators

For convenience, the for loop and if statement can be applied to single
statements at the end of the line:

    print "hello world" if name == "Rob"

And with basic loops:

    print "item: ", item for item in *items

## Object Oriented Programming

A simple class:

    class Inventory
      new: =>
        @items = {}

      add_item: (name) =>
        if @items[name]
          @items[name] += 1
        else
          @items[name] = 1

A class is declared with a `class` statement followed by a table-like
declaration where all of the methods and properties are listed.

The `new` property is special in that it will become the constructor.

Notice how all the methods in the class use the fat arrow function syntax. When
calling methods on a instance, the instance itself is sent in as the first
argument. The fat arrow handles the creation of a `self` variable.

The `@` prefix on a variable name is shorthand for `self.`. `@items` becomes
`self.items`.

Creating an instance of the class is done by calling the name of the class as a
function.

    inv = Inventory!
    inv\add_item "t-shirt"
    inv\add_item "pants"

Because the instance of the class needs to be sent to the methods when they are
called, the '\' operator is used.

All properties of a class are shared among the instances. This is fine for
functions, but for other types of objects, undesired results may occur.

Consider the example below, the `clothes` property is shared amongst all
instances, so modifications to it in one instance will show up in another:

    class Person
      clothes: {}
      give_item: (name) =>
        table.insert @clothes, name

    a = Person!
    b = Person!

    a\give_item "pants"
    b\give_item "shirt"

	-- will print both pants and shirt
    print item for item in *a.clothes

### Inheritance

The `extends` keyword can be used in a class declaration to inherit the
properties and methods from another class.

    class BackPack extends Inventory
      size: 10
      add_item: (name) =>
        if #@items > size then error "backpack is full"
        super name

Here we extend our Inventory class, and limit the amount of items it can carry.
The `super` keyword can be called as a function to call the function of the
same name in the super class. It can also be accessed like an object in order
to retrieve values in the parent class that might have been shadowed by the
child class.

### Types

Every instance of a class carries its type with it. This is stored in the
special `__class` property. This property holds the class object. The class
object is what we call to build a new instance. We can also index the class
object to retrieve class methods and properties.

    b = BackPack!
    assert b.__class == BackPack

    print BackPack.size -- prints 10


## Export Statement

Because, by default, all assignments to variables that are not lexically visible will
be declared as local, special syntax is required to declare a variable globally.

The export keyword makes it so any following assignments to the specified names
will not be assigned locally.

	export var_name, var_name2
	var_name, var_name3 = "hello", "world"


This is especially useful when declaring what will be externally visible in a
module:

	-- my_module.moon
	module "my_module", package.seeall
    export print_result

	length = (x, y) -> math.sqrt x*x + y*y

	print_result = (x, y) ->
	  print "Length is ", length x, y

	-- main.moon
	require "my_module"

	my_module.print_result 4, 5 -- prints the result

	print my_module.length 6, 7 -- errors, `length` not visible


## Import Statement

Often you want to bring some values from a table into the current scope as
local variables by their name. The import statement lets us accomplish this:

    import insert from table

The multiple names can be given, each separated by a comma:

    import C, Ct, Cmt from lpeg

Sometimes a function requires that the table be sent in as the first argument
(when using the `\` syntax). As a shortcut, we can prefix the name with a `\`
to bind it to that table:

    -- some object
    my_module =
        state: 100
        add: (value) =>
            self.state + value

    import \add from my_module

    print add(22) -- equivalent to calling my_module:get(22)

## With Statement

A common pattern involving the creation of an object is calling a series of
functions and setting a series of properties immediately after creating it.

This results in repeating the name of the object multiple times in code, adding
unnecessary noise. A common solution to this is to pass a table in as an
argument which contains a collection of keys and values to overwrite. The
downside to this is that the constructor of this object must support this form.

The `with` block helps to alleviate this. It lets us use a bare function and
index syntax in order to work with the object:

    with Person!
      .name = "Oswald"
      \add_relative my_dad
      \save!
      print .name

The `with` statement can also be used as an expression which returns the value
it has been giving access to.

    file = with File "favorite_foods.txt"
      \set_encoding "utf8"

Or...

    create_person = (name,  relatives) ->
      with Person!
        .name = name
        \add_relative relative for relative in *relatives

    me = create_person "Leaf", {dad, mother, sister}

## The Using Clause; Controlling Destructive Assignment

*This isn't implemented yet*

While lexical scoping can be a great help in reducing the complexity of the
code we write, things can get unwieldy as the code size increases. Consider
the following snippet:

    i = 100

    -- many lines of code...

    my_func = ->
        i = 10
        while i > 0
            print i
            i -= 1

    my_func()

    print i -- will print 0


In `my_func`, we've overwritten the value of `i` mistakenly. In this example it
is quite obvious, but consider a large, or foreign code base where it isn't
clear what names have already been declared.

It would be helpful to say which variables from the enclosing scope we intend
on change, in order to prevent us from changing others by accident.

The `using` keyword lets us do that. `using nil` makes sure that no closed
variables are overwritten in assignment. The `using` clause is placed after the
argument list in a function, or in place of it if there are no arguments.

    i = 100

    my_func = (using nil) ->
        i = "hello" -- a new local variable is created here

    my_func()
    print i -- prints 100, i is unaffected


Multiple names can be separated by commas. Closure values can still be
accessed, they just cant be modified:

    tmp = 1213
    i, k = 100, 50

    my_func = (add using k,i) ->
        tmp = tmp + add -- a new local tmp is created 
        i += tmp
        k += tmp

    my_func(22)
    print i,k -- these have been updated

