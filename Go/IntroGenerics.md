## [GopherCon 2021: Robert Griesemer & Ian Lance Taylor - Generics!](https://www.youtube.com/watch?v=Pa_e9EeCdy8)

### Guidance

- DON'T start writing code by defining type paramater constraints on a function 
- First write a function with a specific type and then evaluate if to be made generic - 
- Use generics only where lot of common code exsits



### When to use type-parameter based Generics:

 Situations 	

- Boilerplate code which looks the same, just input/output types are different 	
- Any general logic which is largely independent of the typ - 

Type-parameters in Function definitions



```go
type List[T any] struct {
  next 	*List[T]
  val 	T
}

// Index returns the index of x in s, or -1 if not found
func Index[T comparable](s []T, x T) int {
  for i, v := range s {
    // v and x are type T, which has the comparable
    // constraint, so we can use == here
    if v == x {
      return i
    }
  }
  return -1
}
```

