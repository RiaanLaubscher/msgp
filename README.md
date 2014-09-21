msgp
=======

This is a tool for serializing Go `struct`s using the [MesssagePack](http://msgpack.org) standard. It is targeted 
at the `go generate` tool.

### Use

In a source file, include the following directive:

```go
//go:generate msgp
```

The `msgp` command will generate `Unmarshal`, `Marshal`, `EncodeMsg`, and `DecodeMsg` methods for all exported struct
definitions in the file. You will need to include that directive in every file that contains structs that 
need code generation. The generated files will be named {filename}_gen.go.

Field names can be overridden in much the same way as the `encoding/json` package. For example:

```go
type Person struct {
	Name       string `msg:"name"`
	Address    string `msg:"address"`
	Age        int    `msg:"age"`
	Hidden     string `msg:"-"` // this field is ignored
	unexported bool             // this field is also ignored
}
```

The `msgp/enc` package has a function called `CopyToJSON` which can take MessagePack-encoded binary
and translate it directly into JSON. It has reasonably high performance, and works much the same way that `io.Copy` does.

### Status

Very alpha. The code generation can be flaky for complicated structs. However, if the generated code compiles, you 
can be reasonably certain that it will work.

### Performance

As you might imagine, the generated code is quite a lot more performant than reflection-based serialization. Here 
are the two built-in benchmarking cases provided. Each benchmark writes the struct to a buffer and then extracts 
it again.

##### Benchmark 1

Type specification:
```go
type TestType struct {
	F   *float64          `msg:"float"`
	Els map[string]string `msg:"elements"`
	Obj struct {
		ValueA string `msg:"value_a"`
		ValueB []byte `msg:"value_b"`
	} `msg:"object"`
	Child *TestType `msg:"child"`
}
```

|  Method | Time | Heap Use | Heap Allocs |
|:-------:|:----:|:--------:|:-----------:|
| msgp codegen | 2553ns | 129 B | 5 allocs |
| [ugorji/go](http://github.com/ugorji/go) | 8467ns | 2015 B | 43 allocs |
| encoding/json | 11976ns | 2004 B | 33 allocs |



##### Benchmark 2

Type specification:
```go
type TestFast struct {
	Lat  float64
	Long float64
	Alt  float64
	Data []byte
}
```
|  Method | Time | Heap Use | Heap Allocs |
|:-------:|:----:|:--------:|:-----------:|
| msgp codegen | 1151ns | 57 B | 1 allocs |
| [ugorji/go](http://github.com/ugorji/go) | 5199ns | 1227 B | 27 allocs |
| encoding/json | 8120ns | 1457 B | 11 allocs |
