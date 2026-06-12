+++ date = '2026-06-12T07:00:30+05:30' draft = false title = 'Generics and Itertors in GO' +++
# Mastering Generics and Iterators in Go

Go is famous for its simplicity and fast performance. 
For years, it lacked two features common in other languages: Generics and Iterators. 
Recent updates completely changed this. 
Go 1.18 introduced generics. 
Go 1.23 added formal support for iterators. 
Together, these features allow you to write highly reusable, clean, and type-safe code.

Here is your complete guide to mastering generics and iterators in modern Go.

---

## 1. Understanding Generics in Go

Generics allow you to write functions and data structures without pinning them to a specific type. 
Instead of writing separate functions for `int`, `float64`, and `string`, you write one function using a **Type Parameter**.

### Type Constraints and the `any` Keyword
Type constraints define what types a generic function can accept. 
The most common constraint is `any`, which is an alias for the empty interface `interface{}`.

```go
package main

import "fmt"

// PrintSlice accepts a slice of any data type
func PrintSlice[T any](s []T) {
	for _, v := range s {
		fmt.Println(v)
	}
}

func main() {
	PrintSlice([]int{1, 2, 3})
	PrintSlice([]string{"Go", "Generics", "Iterators"})
}
```

### Creating Custom Constraints
Sometimes `any` is too broad. 
If you need to compare values using operators like `<`, `>`, or `==`, you must restrict your type parameter. 
The standard library provides the `cmp` package for ordered types.

```go
package main

import (
	"cmp"
	"fmt"
)

// FindMax works for any type that supports ordered comparison (numbers, strings)
func FindMax[T cmp.Ordered](a, b T) T {
	if a > b {
		return a
	}
	return b
}

func main() {
	fmt.Println(FindMax(10, 20))             // int
	fmt.Println(FindMax("apple", "banana"))   // string
}
```

---

## 2. Navigating Iterators in Go

Before Go 1.23, iterating over custom collections required custom loops, channels, or heavy callback functions. 
Go 1.23 standardized iterators using the `iter` package. 

Go supports two main iterator types:
1. `iter.Seq[V]`: Iterates over a sequence of single values (like values in a list).
2. `iter.Seq2[K, V]`: Iterates over pairs of values (like key-value pairs in a map or index-value pairs in a slice).

### The Magic of `range`
Any function that matches the signature of `iter.Seq` or `iter.Seq2` can be plugged directly into a standard `for ... range` loop.

Here is how to create a custom loop that generates a sequence of numbers:

```go
package main

import (
	"fmt"
	"iter"
)

// Count returns an iterator that counts from start to end
func Count(start, end int) iter.Seq[int] {
	return func(yield func(int) bool) {
		for i := start; i <= end; i++ {
			// yield returns false if the loop breaks early
			if !yield(i) {
				return
			}
		}
	}
}

func main() {
	// The range keyword natively supports the Count iterator function
	for num := range Count(1, 5) {
		fmt.Println(num)
	}
}
```

---

## 3. Combining Generics and Iterators

The real power unlocks when you combine generics and iterators. 
You can build functional tools like `Map`, `Filter`, and `Reduce` that work seamlessly across any data type.

### Example: A Generic Filter Iterator
Let's build a functional `Filter` tool. 
It takes an existing iterator and returns a new iterator containing only elements that pass a specific condition.

```go
package main

import (
	"fmt"
	"iter"
)

// Filter accepts a generic iterator and returns a filtered generic iterator
func Filter[T any](seq iter.Seq[T], predicate func(T) bool) iter.Seq[T] {
	return func(yield func(T) bool) {
		for v := range seq {
			if predicate(v) {
				if !yield(v) {
					return
				}
			}
		}
	}
}

// Helper to convert a slice into an iter.Seq
func FromSlice[T any](slice []T) iter.Seq[T] {
	return func(yield func(T) bool) {
		for _, v := range slice {
			if !yield(v) {
				return
			}
		}
	}
}

func main() {
	numbers := FromSlice([]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10})
	
	// Filter out odd numbers
	isEven := func(n int) bool { return n%2 == 0 }
	evenNumbers := Filter(numbers, isEven)

	for num := range evenNumbers {
		fmt.Println(num) // Outputs: 2, 4, 6, 8, 10
	}
}
```

---

## Best Practices

* **Do not overcomplicate**: Use generics only when you need to duplicate identical logic across multiple types.
* **Respect loop closure**: Always check the boolean return value of `yield()`. If it returns `false`, stop processing immediately.
* **Leverage `slices` and `maps` packages**: Modern Go standard libraries already include highly optimized generic functions. Check them out before coding your own.
