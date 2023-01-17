# Go Highlevel Summary

# Index

- [Go Highlevel Summary](#go-highlevel-summary)
- [Index](#index)
  - [Credits](#credits)
  - [Go in a Nutshell](#go-in-a-nutshell)
- [Basic Syntax](#basic-syntax)
  - [Hello World](#hello-world)
  - [Operators](#operators)
    - [Arithmetic](#arithmetic)
    - [Comparison](#comparison)
    - [Logical](#logical)
    - [Other](#other)
  - [Declarations](#declarations)
  - [iota](#iota)
  - [Functions](#functions)
    - [Functions As Values And Closures](#functions-as-values-and-closures)
    - [Variadic Functions](#variadic-functions)
  - [Type Conversions](#type-conversions)
  - [Packages](#packages)
  - [Control structures](#control-structures)
    - [If](#if)
    - [Loops](#loops)
    - [Switch](#switch)
  - [Arrays, Slices, Ranges](#arrays-slices-ranges)
    - [Arrays](#arrays)
    - [Slices](#slices)
    - [Operations on Arrays and Slices](#operations-on-arrays-and-slices)
  - [Maps](#maps)
  - [Structs](#structs)
  - [Pointers](#pointers)
  - [Interfaces](#interfaces)
  - [Embedding](#embedding)
  - [Errors](#errors)
- [Concurrency](#concurrency)
  - [Goroutines](#goroutines)
  - [Channels](#channels)
    - [Channel Axioms](#channel-axioms)
  - [Printing](#printing)
  - [Reflection](#reflection)
    - [Type Switch](#type-switch)
- [Snippets](#snippets)
  - [Files Embedding](#files-embedding)
  - [HTTP Server](#http-server)
  - [String Enum with Interface Printing](#string-enum-with-interface-printing)

## Credits

This highlevel overview of Go is a fork from [a8m/golang-cheat-sheet](https://github.com/a8m/golang-cheat-sheet). I modified the content to include what I found difficult and confusing during my study. 

## Go in a Nutshell

* Imperative language
* Statically typed
* Syntax tokens similar to C (but less parentheses and no semicolons) and the structure to Oberon-2
* Compiles to native code (no JVM)
* No classes, but structs with methods
* Interfaces which defines a collection of methods
* No implementation inheritance. There's [type embedding](http://golang.org/doc/effective%5Fgo.html#embedding), though.
* Functions are first class objects.
* Functions can return multiple values
* Has closures
* Pointers, but no pointer arithmetic
* Built-in concurrency primitives: Goroutines and Channels
* Loosely coupled, orthogonality 

# Basic Syntax

## Hello World
File `hello.go`:
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello Go")
}
```
`$ go run hello.go`

## Operators
### Arithmetic
|Operator|Description|
|--------|-----------|
|`+`|addition|
|`-`|subtraction|
|`*`|multiplication|
|`/`|division, returns integer portion when both operands are integer|
|`%`|remainder, only with int type|
|`&`|bitwise and|
|`\|`|bitwise or|
|`^`|bitwise xor. Bitwise not when used as unary operator| 
|`&^`|bitwise clear (and not),  n &^= (1 << pos)|
|`<<`|left shift|
|`>>`|right shift|

The bitwise clear operator is equivalent to `p & (^q)`, where ^q is the bitwise not of q. 

To clear the bit of n at position *pos*, `n &^ (1 << pos)` 

To set a bit of n at position *pos*, `n | (1 << pos)`

### Comparison
|Operator|Description|
|--------|-----------|
|`==`|equal|
|`!=`|not equal|
|`<`|less than|
|`<=`|less than or equal|
|`>`|greater than|
|`>=`|greater than or equal|

### Logical
|Operator|Description|
|--------|-----------|
|`&&`|logical and|
|`\|\|`|logical or|
|`!`|logical not|

### Other
|Operator|Description|
|--------|-----------|
|`&`|address of / create pointer|
|`*`|dereference pointer|
|`<-`|send / receive operator (see 'Channels' below)|

## Declarations
Type goes after identifier!
Go has the following built-in types.
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte, alias for uint8   (**no char type**)

rune, alias for int32, represents a Unicode code point

float32 float64

complex64 complex128

Declared variables must be used at least once or the compilation will fail.

Use `_ := unused_variable` to pass compilation 
```go
var foo int // declaration without initialization
var foo int = 42 // declaration with initialization
var foo, bar int = 42, 1302 // declare and init multiple vars at once
var foo = 42 // type omitted, will be inferred
foo := 42 // shorthand, only in func bodies, omit var keyword, type is always implicit
const constant = "This is a constant"

```

## iota

- The iota keyword represents successive **integer constants** 0, 1, 2,…
- It resets to 0 whenever the word const appears in the source code, and increments after each const specification.
- Think of iota as a invisible counter (starting from 0) that is implicitly started with each const block and incremented with each statement

Basic example

```go

const (
    C0 = 2              // 2, iota = 0 even if it does not appear in the first ConstSpec. If the first statement involves the blank variable, the expression cannot be omitted
    C1 int = iota - 5   // 1 - 5 = -4
    C2 = iota + 10      // 2 + 10 = 12
    C3 = iota + 20      // 3 + 20 = 23
    _                   // 4 is skipped, no expression needed
    C4 = iota - 30      // 5 - 30 = -25
    C5                  // repeat of the last ConstSpec, C5=iota - 30 = 6 - 30 = -24
    C6 = iota           // iota = 7
    )   

```
Go has this feature of implicit repetition of the last non-empty expression. So it is possible to declare the following constants without much typing. 

```go
type ByteSize float64
const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

Constants are calculated at compilation time and therefore function calls are not permitted in the expression. 

The idiomatic use case for iota is to create enum types with strings.

- create a new integer type,
- list its values using iota,
- give the type a String function.

See snippet example at the end [String Enum](#string-enum-with-interface-printing)

## Functions
```go
// a simple function
func functionName() {}

// function with parameters (again, types go after identifiers)
func functionName(param1 string, param2 int) {}

// multiple parameters of the same type
func functionName(param1, param2 int) {}

// return type declaration
func functionName() int {
    return 42
}

// Can return multiple values at once
func returnMulti() (int, string) {
    return 42, "foobar"  // can it return list or tuple?
}
var x, str = returnMulti()

// Return multiple named results simply by return
func returnMulti2() (n int, s string) {
    n = 42
    s = "foobar"
    // n and s will be returned
    return
}
var x, str = returnMulti2()

```

### Functions As Values And Closures
```go
func main() {
    // assign a function to a name allows function nesting
    add := func(a, b int) int {     // var add = also works
    // add is of type func(int, int) int
        return a + b
    }
    // use the name to call the function
    fmt.Println(add(3, 4))
}

// Closures, lexically scoped: Functions can access values that were
// in scope when defining the function. Each instance of the returned function
// retains its own version of outer_var
func scope() func() int{    // returns a func with a return type int
    outer_var := 2
    foo := func() int { return outer_var}
    return foo
}

type scopefun func() int

funcs := make([]scopefun, 10)

for i:=0;i<cap(funcs);i++{
    funcs[i] = scope(i)
}

for i:=0;i<cap(funcs);i++{
    fmt.Println(funcs[i]())  // Prints 0, 1, 2... 9
}


// Closures
func outer() (func() int, int) { // returns a function with a return type of int and an int
    outer_var := 2
    inner := func() int {
        outer_var += 99 // outer_var from outer scope is changed.
        return outer_var
    }
    inner()
    return inner, outer_var // return inner func and changed outer_var 101
}
```

### Variadic Functions
```go
func main() {
    fmt.Println(adder())            // 0
	fmt.Println(adder(1, 2, 3)) 	// 6
	fmt.Println(adder(9, 9))	// 18

	nums := []int{10, 20, 30}
	fmt.Println(adder(nums...))	// 60
}

// By using ... before the type name of the last parameter you can indicate that it takes zero or more of those parameters.
// The function is invoked like any other function except we can pass as many arguments as we want.
func adder(args ...int) int {
	total := 0
	for _, v := range args { // Iterates over the arguments whatever the number.
        // the omitted variable is the sequence of the iteration
		total += v
	}
	return total
}
```

## Type Conversions

A wraparound happens when the value is converted to a data type that is too small to hold it. In the preceding example, the 8-bit data type int8 did not have enough space to hold the 64-bit variable big.


```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)

// alternative syntax, types are implicitly inferred
i := 42
f := float64(i)
u := uint(f)
```

## Packages
* Package declaration at top of every source file
* Executables are in package `main`
* Convention: package name == last name of import path (import path `math/rand` => package `rand`)
* Upper case identifier: exported (visible from other packages)
* Lower case identifier: private (not visible from other packages)
  
```go
type Days int

const (
    Monday Days = iota
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
    Sunday
    numOfDays // this identifier will not be exported as its first letter is in lower case 
)

```

## Control structures

### If
```go
func main() {
	// Basic one
	if x > 10 {
		return x
	} else if x == 10 {
		return 10
	} else {
		return -x
	}

	// You can put one statement before the condition
	if a := b + c; a < 42 {
		return a
	} else {
		return a - 42
	}

	// Type assertion inside if
	var val interface{} = "foo"
	if str, ok := val.(string); ok {
        // this is actually one statement before the condition "ok"
        // str, ok := val.(string) will return the value of val and true
        // if the interface val holds value of type string, it will return
        // the string and true.
		fmt.Println(str)
	}
}
```

### Loops
```go
    // There's only `for`, no `while`, no `until`
    for i := 1; i < 10; i++ {   
        // no parentheses around the for statements
    }
    i := 0
    for ; i < 10;  { // while - loop, i still need to be declared first
    }
    for i < 10  { // while loop, you can omit semicolons 
    }
    for { // while (true), infinite loop
    }
    
    // use break/continue on current loop
    // use break/continue with label on outer loop

here:
    for i := 0; i < 2; i++ {
        for j := i + 1; j < 3; j++ {
            if i == 0 {
                continue here
            }
            fmt.Println(j)
            if j == 2 {
                break
            }
        }
    }

there:
    for i := 0; i < 2; i++ {
        for j := i + 1; j < 3; j++ {
            if j == 1 {
                continue
            }
            fmt.Println(j)
            if j == 2 {
                break there
            }
        }
    }
```

### Switch
```go
    // switch statement
    switch operatingSystem {
    case "darwin":
        fmt.Println("Mac OS Hipster")
        // cases break automatically, no fallthrough by default
    case "linux":
        fmt.Println("Linux Geek")
    default:
        // Windows, BSD, ...
        fmt.Println("Other")
    }

    // as with for and if, you can have an assignment statement before the switch value
    switch os := runtime.GOOS; os {
    case "darwin": ...
    }

    // you can also make comparisons in switch cases
    number := 42
    switch {
        case number < 42:
            fmt.Println("Smaller")
        case number == 42:
            fmt.Println("Equal")
        case number > 42:
            fmt.Println("Greater")
    }

    // cases can be presented in comma-separated lists
    var char byte = '?'
    switch char {
        case ' ', '?', '&', '=', '#', '+', '%':
            fmt.Println("Should escape")
    }
```

## Arrays, Slices, Ranges

### Arrays
The size, once declared, is fixed and cannot be extended or reduced once the compilation is completed. This defines its static nature. 

Once an array is declared as an integer type (or any other numeric type, such as float, complex, byte, rune), each element is initialized with a default value of zero. The default value for a string type is “” (empty string) and for a bool, false. For types such as maps, channels, interfaces, and pointers, the default initialization is nil.

Arrays are mutable and changeable by index.

Arrays are passed by value.

```go
var a [10]int // declare an int array with length 10. Array length is part of the type!
a[3] = 42     // set elements
i := a[3]     // read elements

// declare and initialize
var a = [2]int{1, 2}
a := [2]int{1, 2} //shorthand
a := [...]int{1, 2} // elipsis is optional
a := []int{1,2} // if elipsis is omitted, a slice is declared

// Additionally, an array can be assigned like a key/value pair:

var keyValueArray = [5]string{2: "Coffee", 4: "Tea"}
// item 2 and 4 are initialized with the provided strings and the others are initialized to empty strings.
// if a key larger than the maximum key is used, a panic will be triggered 

```

### Slices

- Slices are references to complete or partial arrays.
- A slice is declared by var identifier[] type.
- The length of a slice is the number of elements it contains.
- The capacity of a slice is the number of elements in the underlying array. counting from the first element in the slice. 
- The length and capacity of a slice s can be obtained using the expressions len(s) and cap(s). 

```go
var a []int  // declare a slice - similar to an array, but length is 0 and values are nil
var a = []int {1, 2, 3, 4}   // declare and initialize a slice (backed by the array given implicitly)
a := []string{2: "Coffee", 3: "Tea"} // declare and initialize a slice with a length and a cap of 4

chars := []string{0:"a", 2:"c", 1: "b"}  // ["a", "b", "c"]

var b = a[lo:hi]	// creates a slice (view of the array) from index lo to hi-1
var b = a[1:4]		// slice from index 1 to 3
var b = a[:3]		// missing low index implies 0
var b = a[3:]		// missing high index implies len(a)
a =  append(a,17,3)	// append items to slice a. You can only append to a slice

//The resulting value of append is a slice containing all the elements of the original slice plus the provided values. It will overwrite the items in the backing array provided the length of the slice is not greater than the capacity of the backing array
//If the backing array of s is too small to fit all the given values a bigger array will be allocated. The returned slice will point to the newly allocated array.

a := [5]string{0:"tea", 4:"vermont"}
fmt.Println(a)  // ["tea", "", "", "", "vermont"]
var b[] string = a[1:2]  
fmt.Println(b)  // [""]
fmt.Printf("len of a %d:, cap of a %d\n", len(a), cap(a))  //5, 5
fmt.Printf("len of b %d:, cap of b %d\n", len(b), cap(b))  //1, 4
b=append(b, "boba", "coffee", "wine")   // cap(b) - len(b) <=3, a is modified
fmt.Println(a)  // ["tea", "", "boba", "coffee", "wine"]
fmt.Println(b)  // ["", "boba", "coffee", "wine"], b is still a slice of a
a[4] = "vodka"  // ["tea", "", "boba", "coffee", "vodka"]
fmt.Println(b)  // ["", "boba", "coffee", "vodka"]

var c[] string = a[:]
c = append(c, "cider")
fmt.Println(a)  // [tea "" boba coffee vodka]
fmt.Println(c)  // [tea "" boba coffee vodka cider], c no longer references a

c := append(a,b...)	// concatenate slices a and b

// create a slice with make
a := make([]byte, 5, 5)	// the first arg is length, the second is capacity
a := make([]byte, 5)	// the arg capacity is optional

// create a slice from an array
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // a slice referencing the storage of x
```

### Operations on Arrays and Slices
`len(a)` gives you the length of an array/a slice. It's a built-in function, not a attribute/method on the array.

```go
// loop over an array/a slice
for i, e := range a {
    // i is the index, e the element
}

// if you only need e:
for _, e := range a {
    // e is the element
}

// ...and if you only need the index
for i := range a {
}

// In Go pre-1.4, you'll get a compiler error if you're not using i and e.
// Go 1.4 introduced a variable-free form, so that you can do this
for range time.Tick(time.Second) {
    // do it once a sec
}

```

## Maps

Maps are like dicts in Python and HashMap in Java.

```go
m := make(map[string]int)
m["key"] = 42
fmt.Println(m["key"])

delete(m, "key")    // In Python, dict.pop(key)

elem, ok := m["key"] // test if key "key" is present and retrieve it, if so
// this is more elegant than in Python, if key in dict:


// iterate over map content
for key, value := range m {
}

```

## Structs

There are no classes, only structs. Structs can have methods.
```go
// A struct is a type. It's also a collection of fields

// Declaration
type Vertex struct {
    X, Y float64
}

// Creating
var v = Vertex{1, 2}
var v = Vertex{X: 1, Y: 2} // Creates a struct by defining values with keys
var v = []Vertex{{1,2},{5,2},{5,5}} // Initialize a slice of structs

// map literal of struct
var m = map[string]Vertex{
    "Bell Labs": {40.68433, -74.39967},
    "Google":    {37.42202, -122.08408},
}

// Accessing members
v.X = 4

// You can declare methods on structs. The struct you want to declare the
// method on (the receiving type) comes between the the func keyword and
// the method name. The struct is copied on each method call(!)
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// Call method
v.Abs()

// For methods that modify a Struct, you need to use a pointer (see below) to the Struct
// as the type. With this, the struct value is not copied for the method call.
func (v *Vertex) add(n float64) { // notice no return type is declared after the function name as the function does not return anything
    v.X += n
    v.Y += n
}

```
**Anonymous structs:**

```go
point := struct {
	X, Y int
}{1, 2}
```

## Pointers
```go
p := Vertex{1, 2}  // p is a Vertex
q := &p            // q is a pointer to a Vertex
r := &Vertex{1, 2} // r is also a pointer to a Vertex

// The type of a pointer to a Vertex is *Vertex

var s *Vertex = new(Vertex) // new creates a pointer to a new struct instance
```

## Interfaces

An interface type is defined as a set of method signatures.
A value of interface type can hold any value whose type implements those methods.

Three components are needed to implement interface. 

- an interface declaration
- a custom type declaration 
- a method belonging to the custom type which implements the methods declared under the interface
- a function/method that takes the interface as one of its arguments. The function/method can directly invoke the interface methods inside its body.
  
```go
// interface declaration
type SomeInterface interface {
    SomeMethod() string
}

// types do *not* need to declare to implement interfaces
type T struct {}

// instead, types implicitly satisfy an interface if they implement all required methods
func (v T) SomeMethod() string {
    return "This is an interface!"
}

func SomeFunction(i SomeInterface){
    //...
}
```
Code snippet: implementing the stringer interface on a custom type based on Int.
```go
type MyInt int
func (i MyInt) String() string {
	str := fmt.Sprintf("An integer %d prints through the String interface ", i)
	return str
}

var i MyInt
for i=0;i<5;i++{

    fmt.Println(i) 

}
// Output
/*
An integer 0 prints through the String interface 
An integer 1 prints through the String interface 
An integer 2 prints through the String interface 
An integer 3 prints through the String interface 
An integer 4 prints through the String interface 
*/

```

Another example showing interface checking at runtime
```go
package main

import (

	"fmt"
)


type MyInt int
type MyFloat float64

type SomeInterface interface{
	somemethod() string
}

func (i MyInt) somemethod() string {
	str := fmt.Sprintf("An integer %d prints through the SomeMethod interface ", i)
	return str
}

func (i MyFloat) somemethod() string {
	str := fmt.Sprintf("An floating number %f prints through the SomeMethod interface ", i)
	return str
}


func InterfaceMethod(v any) string{
	i, ok := v.(SomeInterface)      // type assertion to get the underlying type of i
	if ok{
		return i.somemethod()       // now i holds the type SomeInterface and therefore have access to somemethod()
	}else{
		return fmt.Sprintf("Type %T does not implement SomeInterface", v)
	}
}

func main() {

	fmt.Println(InterfaceMethod(MyInt(5)))
	fmt.Println(InterfaceMethod(MyFloat(4.455)))
	fmt.Println(InterfaceMethod(4.5))   // won't compile 
    // It won't compile if the signature of the function InterfaceMethod is (v SomeInterface) as float64 does not implement SomeInterface and therefore you can't pass a floating number to this method. 

}

// An integer 5 prints through the SomeMethod interface 
// An floating number 4.455000 prints through the SomeMethod interface 
// Type float64 does not implement SomeInterface

```

## Embedding

There is no subclassing in Go. Instead, there is interface and struct embedding.

```go
// ReadWriter implementations must satisfy both Reader and Writer
type ReadWriter interface {
    Reader
    Writer
}

// Server exposes all the methods that Logger has
type Server struct {
    Host string
    Port int
    *log.Logger
}

// initialize the embedded type the usual way
server := &Server{"localhost", 80, log.New(...)}

// methods implemented on the embedded struct are passed through
server.Log(...) // calls server.Logger.Log(...)

// the field name of the embedded type is its type name (in this case Logger)
var logger *log.Logger = server.Logger
```

## Errors

There is no exception handling. Instead, functions that might produce an error just declare an additional return value of type [`error`](https://golang.org/pkg/builtin/#error). This is the `error` interface:

```go
// The error built-in interface type is the conventional interface for representing an error condition,
// with the nil value representing no error.
type error interface {
    Error() string
}
```

Here's an example:
```go
func sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, errors.New("negative value")
	}
	return math.Sqrt(x), nil
}

func main() {
	val, err := sqrt(-1)
	if err != nil {
		// handle error
		fmt.Println(err) // negative value
		return
	}
	// All is good, use `val`.
	fmt.Println(val)
}
```

# Concurrency

## Goroutines
Goroutines are lightweight threads (managed by Go, not OS threads). `go f(a, b)` starts a new goroutine which runs `f` (given `f` is a function).

```go
// just a function (which can be later started as a goroutine)
func doStuff(s string) {
}

func main() {
    // using a named function in a goroutine
    go doStuff("foobar")

    // using an anonymous inner function in a goroutine
    go func (x int) {
        // function body goes here
    }(42)

    // either goroutine may not get executed before main exits. 
}
```

The proper way using a channel.

```go
unc main() {
	done := make(chan bool)
	values := []string{"a", "b", "c"}
	for _, v := range values {
		go func(u string) {
			fmt.Println(u)
			done <- true
		}(v)
	}
	for range values {
	    <-done
	}

```
## Channels
```go
ch := make(chan int) // create a channel of type int
ch <- 42             // Send a value to the channel ch.
v := <-ch            // Receive a value from ch

// Non-buffered channels block. Read blocks when no value is available, write blocks until there is a read.

// Create a buffered channel. Writing to a buffered channels does not block if less than <buffer size> unread values have been written.
ch := make(chan int, 100)

close(ch) // closes the channel (only sender should close)

// read from channel and test if it has been closed. read from an open channel if all goroutines are asleep would cause a runtime error (deadlock)
v, ok := <-ch   // won't cause a problem if the channel is closed

// if ok is false, channel has been closed

// Read from channel until it is closed. Unclosed channel causes a runtime panic
for i := range ch {
    fmt.Println(i)
}

// select blocks on multiple channel operations, if one unblocks, the corresponding case is executed. If multiple cases are ready, one would be selected randomly.
func doStuff(channelOut, channelIn chan int) {
    select {
    case channelOut <- 42:
        fmt.Println("We could write to channelOut!")
    case x := <- channelIn:
        fmt.Println("We could read from channelIn")
    case <-time.After(time.Second * 1):
        fmt.Println("timeout")
    }
}
```

### Channel Axioms

[Channel Axioms](https://dave.cheney.net/2014/03/19/channel-axioms)

- A send to a nil channel blocks forever

    ```go
    var c chan string
    c <- "Hello, World!"
    // fatal error: all goroutines are asleep - deadlock!
    ```
- A receive from a nil channel blocks forever

  ```go
  var c chan string
  fmt.Println(<-c)
  // fatal error: all goroutines are asleep - deadlock!
  ```
- A send to a closed channel panics

  ```go
  var c = make(chan string, 1)
  c <- "Hello, World!"
  close(c)
  c <- "Hello, Panic!"
  // panic: send on closed channel
  ```
- A receive from a closed channel returns the zero value immediately. 
  
   `Implication: a closed channel will be selected immediately, and get nil value of the channel type. Thus may cause the other channels in the select never get selected.`

  ```go
  var c = make(chan int, 2)
  c <- 1
  c <- 2
  close(c)
  for i := 0; i < 3; i++ {
      fmt.Printf("%d ", <-c)
  }
  // 1 2 0
  ```

## Printing

```go
fmt.Println("Hello, 你好, नमस्ते, Привет, ᎣᏏᏲ") // basic print, plus newline
p := struct { X, Y int }{ 17, 2 }
fmt.Println( "My point:", p, "x coord=", p.X ) // print structs, ints, etc
s := fmt.Sprintln( "My point:", p, "x coord=", p.X ) // print to string variable

fmt.Printf("%d hex:%x bin:%b fp:%f sci:%e",17,17,17,17.0,17.0) // c-ish format
s2 := fmt.Sprintf( "%d %f", 17, 17.0 ) // formatted print to string variable

hellomsg := `
 "Hello" in Chinese is 你好 ('Ni Hao')
 "Hello" in Hindi is नमस्ते ('Namaste')
` // multi-line string literal, using back-tick at beginning and end
```

## Reflection
### Type Switch
A type switch is like a regular switch statement, but the cases in a type switch specify types (not values) which are compared against the type of the value held by the given interface value.
```go
func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}
```

# Snippets

## Files Embedding

Go programs can embed static files using the `"embed"` package as follows:

```go
package main

import (
	"embed"
	"log"
	"net/http"
)

// content holds the static content (2 files) for the web server.
//go:embed a.txt b.txt
var content embed.FS

func main() {
	http.Handle("/", http.FileServer(http.FS(content)))
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

[Full Playground Example](https://play.golang.org/p/pwWxdrQSrYv)

## HTTP Server
```go
package main

import (
    "fmt"
    "net/http"
)

// define a type for the response
type Hello struct{}

// let that type implement the ServeHTTP method (defined in interface http.Handler)
func (h Hello) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello!")
}

func main() {
    var h Hello
    http.ListenAndServe("localhost:4000", h)
}

// Here's the method signature of http.ServeHTTP:
// type Handler interface {
//     ServeHTTP(w http.ResponseWriter, r *http.Request)
// }
```
## String Enum with Interface Printing

```go
type Direction int

const (
	North Direction = iota
	East
	South
	West
)

func (d Direction) String() string {
	return [...]string{"North", "East", "South", "West"}[d]
}

// In use
var d Direction = North
fmt.Print(d)
switch d {
case North:
	fmt.Println(" goes up.")
case South:
	fmt.Println(" goes down.")
default:
	fmt.Println(" stays put.")
}
// Output: North goes up.
```

Cheaper and safer than using `map[string]interface{}` if the structure of the JSON data is known beforehand. Only use map when dealing with unknown data structures.
`interface{}` is equivalent to the keyword `any`.