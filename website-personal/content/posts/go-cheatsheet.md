+++
date = '2026-06-05T21:52:33+05:30'
draft = false
title = 'Go Cheatsheet'
+++
## Basics

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go!")
}
```

## Variables & Constants

```go
var name string = "Alice"
var age = 30                     // type inference
name := "Bob"                    // short declaration (inside func only)
var x, y int = 1, 2              // multiple init

const Pi = 3.14
const (
    StatusOK = 200
    StatusNotFound = 404
)
```

## Types

```
bool                      // true, false
string
int, int8, int16, int32, int64
uint, uint8, uint16, uint32, uint64
float32, float64
complex64, complex128
byte                      // alias for uint8
rune                      // alias for int32 (Unicode code point)
```

**Zero values:** `0` for numbers, `false` for bool, `""` for string, `nil` for pointers/slices/maps/channels/funcs/interfaces.

## Collections

### Array (fixed size)
```go
var arr [3]int = [3]int{1, 2, 3}
arr2 := [...]int{4, 5, 6}
```

### Slice (dynamic)
```go
s := []int{1, 2, 3}
s = append(s, 4)
sub := s[1:3]             // [2, 3] (exclusive end)
make([]int, 5)            // len=5, cap=5
make([]int, 3, 10)        // len=3, cap=10
```

### Map
```go
m := map[string]int{"a": 1, "b": 2}
m["c"] = 3
v, ok := m["a"]           // ok == false if key missing
delete(m, "a")
```

### Struct
```go
type Person struct {
    Name string
    Age  int
}

p := Person{"Alice", 30}
p := Person{Name: "Bob", Age: 25}
p.Name = "Charlie"
```

## Control Flow

### If
```go
if x > 0 {
    fmt.Println("positive")
} else if x < 0 {
    fmt.Println("negative")
} else {
    fmt.Println("zero")
}

// with init statement
if err := doStuff(); err != nil {
    fmt.Println(err)
}
```

### For (only loop construct)
```go
for i := 0; i < 10; i++ { }           // C-style
for i < 10 { }                         // while-style
for { }                                // infinite
for i, v := range slice { }           // index, value
for k, v := range map { }             // key, value
for _, v := range slice { }           // skip index
```

### Switch
```go
switch n {
case 1:
    fmt.Println("one")
case 2, 3:
    fmt.Println("two or three")
default:
    fmt.Println("other")
}

// without expression (if/else chain)
switch {
case n < 0:
    fmt.Println("negative")
case n == 0:
    fmt.Println("zero")
}
```

## Functions

```go
func add(a, b int) int {
    return a + b
}

// multiple return values
func div(a, b int) (int, error) {
    if b == 0 { return 0, errors.New("div by zero") }
    return a / b, nil
}

// named return values
func sum(a, b int) (result int) {
    result = a + b
    return                          // naked return (bare)
}

// variadic
func sum(nums ...int) int { }
```

## Methods

```go
type Rectangle struct { W, H float64 }

// value receiver
func (r Rectangle) Area() float64 { return r.W * r.H }

// pointer receiver (modifies struct)
func (r *Rectangle) Scale(f float64) { r.W *= f; r.H *= f }
```

**Rule:** If any method uses a pointer receiver, all should — be consistent.

## Interfaces

```go
type Shape interface {
    Area() float64
}

func PrintArea(s Shape) {
    fmt.Println(s.Area())
}

// implicit implementation (no "implements" keyword)
type Circle struct { R float64 }
func (c Circle) Area() float64 { return math.Pi * c.R * c.R }
```

## Pointers

```go
var x int = 10
var p *int = &x
fmt.Println(*p)            // 10 (dereference)
*p = 20                    // modifies x
```

## Error Handling

```go
f, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}
defer f.Close()

// custom errors
errors.New("something went wrong")
fmt.Errorf("user %d not found", id)
```

## Defer

```go
f, _ := os.Create("file.txt")
defer f.Close()            // runs when enclosing func returns

// LIFO order: prints 3, 2, 1
defer fmt.Println(1)
defer fmt.Println(2)
defer fmt.Println(3)
```

## Goroutines & Channels

```go
go myFunc()                // starts a goroutine

// channel
ch := make(chan int)
ch <- 42                   // send
v := <-ch                  // receive

// buffered channel
ch := make(chan int, 10)

// close channel
close(ch)
for v := range ch { }      // read until closed

// select (wait on multiple channels)
select {
case v := <-ch1:
    fmt.Println(v)
case <-ch2:
    fmt.Println("received on ch2")
case <-time.After(1 * time.Second):
    fmt.Println("timeout")
}
```

## Packages

```go
package mypackage            // directory name matches package name

// exported (public) = capital letter
func ExportedFunc() {}

// unexported (private) = lowercase
func unexportedFunc() {}
```

## Pointers vs Values (quick rule)

| Context | Should use |
|---|---|
| Modify receiver | Pointer `*T` |
| Large struct | Pointer `*T` (avoids copy) |
| Nil receiver | Pointer `*T` |
| Immutable small data | Value `T` |
| Interface satisfaction | Pointer `*T` if methods defined on `*T` |

## Common Packages

```go
"fmt"              // Printf, Sprintf, Scanf
"os"               // File, Open, Read, Write
"io"               // Reader, Writer interface
"io/ioutil"        // ReadFile, WriteFile (deprecated, use os)
"strings"          // Contains, Split, Join, Trim
"strconv"          // Atoi, Itoa, ParseInt
"time"             // Now, Sleep, Duration
"encoding/json"    // Marshal, Unmarshal
"net/http"         // Get, Handle, ListenAndServe
"sync"             // Mutex, WaitGroup
"errors"           // New, Unwrap, Is, As
```

## Testing

```go
// file: math_test.go
package main

import "testing"

func TestAdd(t *testing.T) {
    result := add(2, 3)
    if result != 5 {
        t.Errorf("expected 5, got %d", result)
    }
}
```

```
go test                    # run tests
go test -v                 # verbose
go test ./...              # all packages
go test -bench .           # benchmarks
```

## Project Layout (minimal)

```
mymod/
├── go.mod                 # module definition
├── main.go                # package main
├── mypackage/
│   └── stuff.go           # package mypackage
└── mypackage_test.go
```

## Commands

```bash
go mod init <module>       # init module
go build                   # compile
go run .                   # run
go test ./...              # test
go fmt ./...               # format
go vet ./...               # static analysis
go mod tidy                # clean up deps
go get <pkg>               # add dependency
```
