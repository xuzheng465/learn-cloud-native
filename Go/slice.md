```go
// push
a = append(a, x)

// pop
x, a = a[len(a)-1], a[:len(a)-1]

// Push Front
a = append([]T{x}, a...)

// pop front
x, a = a[0], a[1:]

// filtering without allocating
// This trick uses the fact that a slice shares the same backing array and capacity as the original, so the storage is reused for the filtered slice. Of course, the original contents are modified.
b:=a[:0]
for _, x := range a {
  if f(x) {
    b = append(b, x)
  }
}

// for elements which must be garbage collected, 
for i:=len(b); i < len(a); i++ {
  a[i] = nil
}

// revsersing
for i:=len(a)/2-1; i>=0; i-- {
  opp := len(a) - 1 - i
  a[i], a[opp] = a[opp], a[i]
}

// revsersing II
for left, right := 0, len(a)-1; left<right; left,right = left+1, right-1 {
  a[left], a[right] = a[right], a[left]
}

// shuffle

```

