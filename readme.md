# gin-gonic 官方文档笔记

## 1. 安装

To install Gin package, you need to install Go and set your Go workspace first.

1. The first need [Go](https://golang.org/) installed (**version 1.13+ is required**), then you can use the below Go command to install Gin.

   > ```go
   >  go get -u github.com/gin-gonic/gin
   > ```

2. Import it in your code:

   > ```go
   > import "github.com/gin-gonic/gin"
   > ```

 3. (Optional) Import `net/http`. This is required for example if using constants such as `http.StatusOK`.

    > ```go
    > import "net/http"
    > ```

## 2. 使用Http动词

```go
func main() {
	// Creates a gin router with default middleware:
	// logger and recovery (crash-free) middleware
	router := gin.Default()

	router.GET("/someGet", getting)
	router.POST("/somePost", posting)
	router.PUT("/somePut", putting)
	router.DELETE("/someDelete", deleting)
	router.PATCH("/somePatch", patching)
	router.HEAD("/someHead", head)
	router.OPTIONS("/someOptions", options)

	// By default it serves on :8080 unless a
	// PORT environment variable was defined.
	router.Run()
	// router.Run(":3000") for a hard coded port
}
```

## 3. 路径中的参数

```go
func main() {
	router := gin.Default()

	// 使用:[param] 只会处理参数匹配的 /user/john 但是并不会处理 /user/ 或者 /user
	// This handler will match /user/john but will not match /user/ or /user
	router.GET("/user/:name", func(c *gin.Context) {
		name := c.Param("name")
		c.String(http.StatusOK, "Hello %s", name)
	})

	// 使用*[param] 可以同时处理有这个参数的 /user/john/ 和没有这个参数的 /user/john/send
	// However, this one will match /user/john/ and also /user/john/send
	// If no other routers match /user/john, it will redirect to /user/john/
	router.GET("/user/:name/*action", func(c *gin.Context) {
		name := c.Param("name")
		action := c.Param("action")
		message := name + " is " + action
		c.String(http.StatusOK, message)
	})
    
	// gin.Context将会储存匹配的路径
	// For each matched request Context will hold the route definition
	router.POST("/user/:name/*action", func(c *gin.Context) {
		b := c.FullPath() == "/user/:name/*action" // true
		c.String(http.StatusOK, "%t", b)
	})

	// 绝对路径的优先级高于含有参数的路径 不管路由他们的方法定义顺序
    // This handler will add a new router for /user/groups.
	// Exact routes are resolved before param routes, regardless of the order they were defined.
	// Routes starting with /user/groups are never interpreted as /user/:name/... routes
	router.GET("/user/groups", func(c *gin.Context) {
		c.String(http.StatusOK, "The available groups are [...]")
	})

	router.Run(":8080")
}
```

## 4. Querystring

```go
func main() {
	router := gin.Default()

	// Query string parameters are parsed using the existing underlying request object.
	// The request responds to a url matching:  /welcome?firstname=Jane&lastname=Doe
	router.GET("/welcome", func(c *gin.Context) {
		firstname := c.DefaultQuery("firstname", "Guest")
		lastname := c.Query("lastname") // shortcut for c.Request.URL.Query().Get("lastname")

		c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
	})
	router.Run(":8080")
}
```

## 5. Multipart/Urlencoded Form

```go
func main() {
	router := gin.Default()

	router.POST("/form_post", func(c *gin.Context) {
		message := c.PostForm("message")
		nick := c.DefaultPostForm("nick", "anonymous")

		c.JSON(200, gin.H{
			"status":  "posted",
			"message": message,
			"nick":    nick,
		})
	})
	router.Run(":8080")
}
```

### 另外一个例子 Another example: query + post form

> ```
> POST /post?id=1234&page=1 HTTP/1.1
> Content-Type: application/x-www-form-urlencoded
> 
> name=manu&message=this_is_great
> ```

```go
func main() {
	router := gin.Default()

	router.POST("/post", func(c *gin.Context) {

		id := c.Query("id")
		page := c.DefaultQuery("page", "0")
		name := c.PostForm("name")
		message := c.PostForm("message")

		fmt.Printf("id: %s; page: %s; name: %s; message: %s", id, page, name, message)
	})
	router.Run(":8080")
}
```

> ```
> id: 1234; page: 1; name: manu; message: this_is_great
> ```

## 6. querystring 或者 postform 参数传map类型

> POST /post?ids[a]=1234&ids[b]=hello HTTP/1.1
> Content-Type: application/x-www-form-urlencoded
>
> names[first]=thinkerou&names[second]=tianou

```go
func main() {
	router := gin.Default()

	router.POST("/post", func(c *gin.Context) {

		ids := c.QueryMap("ids")
		names := c.PostFormMap("names")

		fmt.Printf("ids: %v; names: %v", ids, names)
	})
	router.Run(":8080")
}
```

