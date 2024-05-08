# go语言解析map[string]interface{}的注意事项


go语言解析 map[string]interface{} 数据格式的注意事项：

- map记得分配内存
- 解析出来的**int类型会变成float64类型**
- 注意判断不为nil后再转换类型

<!-- more -->

示例代码：

```go
package main

import (
    "fmt"
    "encoding/json"
)

func main() {
    var m map[string]interface{}    //声明变量，不分配内存
    m = make(map[string]interface{}) //必可不少，分配内存
    m["name"] = "saul"
    var age int = 20
    m["age"] = age
    m["nation"] = "China"
    print_map(m)
    fmt.Println("==============")

    data, err:=json.Marshal(m)
    if err != nil {
        
    }
    fmt.Println("err:", err)
    fmt.Println(data)
    fmt.Println()

    m1 := make(map[string]interface{})
    err = json.Unmarshal(data, &m1)
    fmt.Println("err:", err)
    fmt.Println(m1)
    print_map(m1)
    fmt.Println()

    if m1["name"]!= nil {
        fmt.Println(m1["name"].(string))
    }
    if m1["type"]!= nil {
        fmt.Println(m1["type"].(string))
    } else {
        fmt.Println("there is not the key of type")
    }

}

//解析 map[string]interface{} 数据格式
func print_map(m map[string]interface{}) {
    for k, v := range m {
        switch value := v.(type) {
        case nil:
            fmt.Println(k, "is nil", "null")
        case string:
            fmt.Println(k, "is string", value)
        case int:
            fmt.Println(k, "is int", value)
        case float64:
            fmt.Println(k, "is float64", value)
        case []interface{}:
            fmt.Println(k, "is an array:")
            for i, u := range value {
                fmt.Println(i, u)
            }
        case map[string]interface{}:
            fmt.Println(k, "is an map:")
            print_map(value)
        default:
            fmt.Println(k, "is unknown type", fmt.Sprintf("%T", v))
        }
    }
}
```

结果：

```
name is string saul
age is int 20
nation is string China
==============
[123 34 97 103 101 34 58 50 48 44 34 110 97 109 101 34 58 34 115 97 117 108 34 44 34 110 97 116 105 111 110 34 58 34 67 104 105 110 97 34 125]
==============
age is float64 20
name is string saul
nation is string China
==============
saul
there is not the key of type
```










