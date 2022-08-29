---
title: "Format verbs in golang's printf"
tags: ["English","入门","语法","已完结","go","printf"]
date: 2021-09-04T23:35:29+08:00
---

<b>Golang has formatted I/O functions about `printf` where in package `fmt`.They support the format 'verbs' which are derived from C' but are simpler.</b>  
------

# Example
```go
package main

import (
    "fmt"
    "os"
)

type point struct {
    x, y int
}

func main() {

    p := point{1, 2}
    fmt.Printf("%v\n", p)                      // {1 2}

    fmt.Printf("%+v\n", p)                     // {x:1 y:2}

    fmt.Printf("%#v\n", p)                     // main.point{x:1, y:2}

    fmt.Printf("%T\n", p)                      // main.point

    fmt.Printf("%t\n", true)                   // true

    fmt.Printf("%d\n", 123)                    // 123

    fmt.Printf("%b\n", 14)                     // 1110

    fmt.Printf("%c\n", 33)                     // !

    fmt.Printf("%x\n", 456)                    // 1c8

    fmt.Printf("%f\n", 78.9)                   // 78.900000

    fmt.Printf("%e\n", 123400000.0)            // 1.234000e+08
    fmt.Printf("%E\n", 123400000.0)            // 1.234000E+08

    fmt.Printf("%s\n", "\"string\"")           // "string"

    fmt.Printf("%q\n", "\"string\"")           // "\"string\""

    fmt.Printf("%x\n", "hex this")             // 6865782074686973

    fmt.Printf("%p\n", &p)                     // 0x42135100

    fmt.Printf("|%6d|%6d|\n", 12, 345)         // |    12|   345|

    fmt.Printf("|%6.2f|%6.2f|\n", 1.2, 3.45)   // |  1.20|  3.45|

    fmt.Printf("|%-6.2f|%-6.2f|\n", 1.2, 3.45) // |1.20  |3.45  |

    fmt.Printf("|%6s|%6s|\n", "foo", "b")      // |   foo|     b|

    fmt.Printf("|%-6s|%-6s|\n", "foo", "b")    // |foo   |b     |

    s := fmt.Sprintf("a %s", "string")
    fmt.Println(s)                             // a string

    fmt.Fprintf(os.Stderr, "an %s\n", "error") // an error
}
```

# References
[go dev](https://pkg.go.dev/fmt#pkg-overview)
[go by example](https://gobyexample.com/string-formatting)
[Using '%q' inside fmt.Printf in Go (instead of '%s')](https://dev.to/mcaci/using-q-inside-fmt-printf-in-go-2i23)
