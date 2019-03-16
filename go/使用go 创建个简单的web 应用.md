使用go 创建个简单的web 应用

```go
package main

import (
	"fmt"
	"net/http"
)

func handleFunc(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w,"this is my first web!")
}

func main (){
	http.HandleFunc("/", handleFunc)
	http.ListenAndServe(":3000", nil)
}
```

http://http://127.0.0.1:3000/

得到：this is my first web!

