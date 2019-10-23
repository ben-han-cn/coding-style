# Formatting
1. use gofmt
```shell
gofmt -w youapp.go
```

# Names
1. Package name
 - Short
 - Lower case, single word
 - Singular
1. Getter for attribute shouldn't include Get 
1. Interface name is noun
1. Use MixedCaps rather underscores to write multiword
1. Variable names should be short rather than long. The
   further from its declaration that a name is used, the
   more descriptive the name must be.

# Zero value & nil
- bool  false
- numbers 0
- string ""
- pointer, slice, map, channel, function, interface -> nil
- nil is not a keyword, but predefined identifier
- nil interface is (nil, nil)

# Control structures
1. If
```go
if err := file.Chmod(0644); err != nil {
    log.Print(err)
    return err
}
```

1. For
```go
//only need key or index
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
//only need value
sum := 0
for _, value := range array {
    sum += value
}
//while in c
for condition {
}
//dead loop
for {}
```

1. Type switch
```go
var t interface{}
t = someFunction()
switch t := t.(type) {
case bool:
    fmt.Printf("boolean %t\n", t)
case int:
    fmt.Printf("integer %d\n", t) 
case *int:
    fmt.Printf("pointer to integer %d\n", *t)
default:
    fmt.Printf("unexpected type %T\n", t)
}
```

1. Defer
Schedules a function call to be run immediately before the function 
executing the defer returns. Normally defer is used to clean up resources
```go
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close() 
    //....
}
```

# init 
init is called after all the variable declarations in the package have
evaluated their initializers, and those are evaluated only after all the
imported package have been initialized

# Methods   
1. Pointers vs Values  
  - Value methods can be invokedd on pointers and values, but pointer methods
  only be invoked on pointers
  - Invoke a value method on pointer will cause the method to receive the copy
  of the value     
1. Grouping and ordering
  - Functions should be sorted in rough call order
  - Functions in a file should be grouped by receiver
```go
type something struct{ ... }
func newSomething() *something {
    return &something{}
}
func (s *something) Cost() {
  return calcCost(s.weights)
}
func (s *something) Stop() {...}
func calcCost(n []int) int {...}
```

# Error handling
1. use error as return type
```go
type error interface {
    Error() string
}
```
1. Error cases tend to end in return, instead of using else
```go
f, err := os.Open(name)
if err != nil {
    return err
}
```
1. Use error to return failure instead of bool and nil pointer
1. Add context to errors   
  *Don't*:
```go
file, err := os.Open("foo.txt")
if err != nil {
    return err
}
```
  *Do*:
```go
file, err := os.Open("foo.txt")
if err != nil {
    return fmt.Errorf("open foo.txt failed: %w", err)
}
```    
1. If client needs to detect the error, create error variable
```go
var ErrCouldNotOpen = errors.New("could not open")  
func Open() error {
  return ErrCouldNotOpen
}
// package bar
if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```
1. return nil instead of empty self defined error type
```go
func do() error {
    var err *doError
    //....
    return err
}
func main() {
    // err is (*doError, nil) which isn't nil
    if err := do(); err == nil {
    }
}
```

# Enum
```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)
```

# Dependency management
1. Use dep or go mod
1. Divide imports into four groups sorted from internal to 
external for readability:
  - Standard library
  - Everything else

# Logging
Use structred logging
```go
import "github.com/sirupsen/logrus"

logger.WithField("port", port).Info("Server is listening")
```
# Unit test
1. use table driven test  
  *Don't*:
```go
func TestAdd(t *testing.T) {
	assert.Equal(t, 1+1, 2)
	assert.Equal(t, 1+-1, 0)
	assert.Equal(t, 1, 0, 1)
	assert.Equal(t, 0, 0, 0)
}
```
  *Do*:
```go
func TestAdd(t *testing.T) {
	cases := []struct {
		A, B, Expected int
	}{
		{1, 1, 2},
		{1, -1, 0},
		{1, 0, 1},
		{0, 0, 0},
	}

	for _, tc := range cases {
		t.Run(fmt.Sprintf("%d + %d", tc.A, tc.B), func(t *testing.T) {
			t.Parallel()
			assert.Equal(t, t.Expected, tc.A+tc.B)
		})
	}
}
```  

# Performance  
1. Reserve capacity if size is know in advance  
```go
m := make(map[string]int, 10)
s := make([]int, 10)
```
1. Perfer strconv over fmt 
```go
for i := 0; i < b.N; i++ {
    s := strconv.Itoa(rand.Int())
}
```
1. Avoid string-to-byte conversion
```go
import (
    "bytes"
)
name := "benjamin"
data := []byte(name) //data is a copy
rs := bytes.Runes(data) //[]rune utf8 code point
```
1. use Buffer to read or build data
```go
import (
	"bytes"
	"encoding/base64"
	"io"
	"os"
)
func main() {
	// A Buffer can turn a string or a []byte into an io.Reader.
	buf := bytes.NewBufferString("R29waGVycyBydWxlIQ==")
	dec := base64.NewDecoder(base64.StdEncoding, buf)
	io.Copy(os.Stdout, dec)

    var b bytes.Buffer
	b.Grow(64)
	b.WriteString([]byte("64 bytes or fewer"))
}
```
1. use Scanner to handle stream
```go
import (
    "bufio"
    "fmt"
    "strings"
)
func main() {
    input := "foo   bar      baz"
    scanner := bufio.NewScanner(strings.NewReader(input))
    scanner.Split(bufio.ScanWords)
    for scanner.Scan() {
        //scanner.Bytes() won't allocate new data
        fmt.Println(scanner.Text())
    }
}
```
