---
title: "Declaring Variables in Go"
tags: ["go"]
date: 2024-03-01T11:15:00
---

Go as a statically typed language requires the developer to set the type of a variable when declaring one.

```go
var age int
```

Declaring a variable means allocating some space in memory to store some value of a given type.
In Go you start with the keyword `var` followed by the name and its type. In the example above
the variable `age` was given an integer type `int` but was not initialized explicitly. Coming
from a dynamic language such as JavaScript you might have expected that its initial value were
`undefined` but that is not what happens. When the Go compiler sees that kind of declaration it
assigns a "zero" value to the variable. That "zero" value depends on the type. The "zero" value
for integers and floats is the number `0`. A string zero value is an empty value `""` and for pointers
it is `nil`.

The compiler is smart enough to infer the type when you choose another way of declaring a variable.

```go
age := 99
```

The compiler knows that the above expression is a variable declaration because it has the `:=` operator.
This operator servers as both declaration and initialization. Here the keyword `var` and the type is missing
but the right hand operand is a number of value `99`. The compiles looks at it and knows that is an integer
and set its type as `int` during compile time.

In summary, if you don't have a value to initialize a variable with you can go with the `var` declaration. When you
already have some value you can go with the `:=` operator.

There are situations where even having a value available I would still choose the `var` declaration because
that would increase the readability of my code.

```go
city := GetCityLocation()
```

The function `GetCityLocation` might return a location but what exactly is a location in this context?
That function returns a value which is assigned to `city`. `city` is not really a good name for this
variable, probably `cityLocation` would sound better, but as programmers we tend to be lazy when
naming variables and just give them the smallest name possible. In this situation, just by looking at
the code it is hard to know what type the function returns. You would need to rely on your editor to
jump to the function definition and there you would see what type the function returns. To avoid this
extra step you may refactor that code above to something more readable like this:

```go
var city Location = GetCityLocation()
// or
var cityLocation Location = GetCityLocation()
```

I know it is more verbose, but when I look at this code I know that the function returns a `Location` and
this type is some sort of custom type, probably a user defined one, `struct`. If I click on the type
it will take me straight to its definition and that is really helpful because the funtion or type definition
may be an imported one from another package in my project or even a lib my project relies on.

Go gives you the option to declare a variable explicitly by setting its type as well as relying on the compiler
to infer the type by using the `:=` operator followed by a value.
