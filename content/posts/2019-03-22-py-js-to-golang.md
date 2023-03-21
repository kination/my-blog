---
layout: post
title:  About golang, for python/js developer
date:   2019-03-22
description: Some golang guide for python/js developer
tags:
- python
- golang
- develop
permalink: py-js-to-golang
---

_This is from the draft version of document I've written for my colleagues(python, javascript developers) who is first in golang, by planning to use it in our company products_


## Golang?

![Screenshot](/assets/post_img/py-js-to-golang/golang-5y.jpg)

Go, also known as Golang, is a computer programming language whose development began in 2007 at Google, and it was introduced to the public in 2009.

It has lots of similarity with C language such as:
- Simplified syntax(there are no keywords such as 'class', 'exception'...)
- You can return data by value or reference
- ...

and also has plus:
- Good readability/usability code
- High cost/performance + easy multi-processing

One of great advantage in Golang is cost/performance, and this is why I selected this.

Comparing with Golang, Python's grammar is bit more simple, and Rustlang is using bit less memory(about 5~10%) on runtime. But you can make way more high performance web service than pure Python, with far more simple code than Rustlang.


### GOPATH
‘GOPATH’ environment variable specifies the location of your golang-based workspace. All of golang libraries you installed will be set inside here. so your project couldn’t find installed golang libs/modules if you ignore this.
If you install golang via ‘brew’, it will automatically create and define as ‘$HOME/go’. But I prefer to add your own path.


## About code
### Code Style
Go is ‘almost’ forcing to use fixed format. By ‘gofmt’ program, it reads the updated state of go file, and convert to standard format. This program is included in most of golang IDE as a plugin, so it will automatically activated after you save your code.

### Identifier
#### Blank identifier
The blank identifier is represented by the underscore character _. It serves as an anonymous placeholder instead of a regular (non-blank) identifier and has special meaning in declarations, as an operand, and in assignments.

Golang formatter does not allow unused variable or package. So if you want to keep your unused stuff temporary, put underscore as:

```go
package xxx

import (
    _ "net/http"
}

func main() {
    _ := addValue(1, 2)
}

func addValue(a int, b int) int {
    return a + b
}
```

#### Exported identifiers
An identifier may be exported to permit access to it from another package. An identifier is exported if both:
- First character of the identifier's name should be capital letter
- It is declared in the package block or it is a field name or method name.

All other identifiers are not exported.

### Error handling
Go does not offer keyword for ‘Exception’, so you need to add error checking code often. To allow handling error outside of function, they return error values with proper data.

```go
func doSomething() (string, error) {
    if true {
        return "success", nil
    } else {
        return "", errors.New("cannot do something")
}
}

...
func main() {
    value, err := doSomething()

    if err != nil {
        fmt.Printf("Failed: %v\n", err)
        return
    }
}
```

Also, if you want to stop your code right away, you can shorten
```go
if err != nil {
    fmt.Printf("Failed: %v\n", err)
    return
}
```
as

```go
if err != nil {
    panic(err)
}
```

### Class & Struct
There are no `class` syntax in golang. But you can use class-like definition by using custom type and method.

```go
type User struct {
	FirstName 	string
	LastName 	string
	Id 		int64
}

func (user User) FullName() string {
	return user.LastName + ", " + user.FirstName
} 

func main() {
	user := User {
		FirstName: "Lite",
		LastName: "DeLTA",
		Id: 1,
	}

	fmt.Println(user.FullName())  // => DeLTA, Lite
}
```

This is equivalent in python as:
```python
class User:
    def __init__(self, first_name, last_name, id):
        self.first_name = first_name
        self.last_name = last_name
        self.id = id

    def full_name(self):
        return self.last_name + ', ' + self.first_name


user = User('Lite', 'DeLTA', 1)
print(user.full_name())
```

### Marshal/Unmarshal
Golang’s default ‘encoding/json’ package includes marshalling/unmarshalling, which is for object->json/json->object.

By using ‘net/http’ package, you can get raw request body including byte data. So when you need to parse response body data to JSON value, unmarshal it to access in JSON-way.

Make sure to make struct variables in Capital letter if you want to export it. You can put `json:”original parameter name”` in back of struct variable like below, to define which data will be migrated to the struct object. So if you fetched byte data like:

```
{“name”: “JR-East”, “location”: “tokyo”}
```

unmarshal it as:
```go
type Company struct {
    Name string `json:"name"`
    Location string `json:"location"`
}

...
res, err := http.Get("http://xxx.xxxx")
if err != nil {
    http.Error(w, err.Error(), 500)
    return
}

b, err := ioutil.ReadAll(res.Body)
defer res.Body.Close()

if err != nil {
    http.Error(w, err.Error(), 500)
    return
}

var company Company
err = json.Unmarshal(b, &company)
if err != nil {
    http.Error(w, err.Error(), 500)
    return
}

fmt.Println("Company name is ", company.Name) // Company name is JR-East
```


## Package manager
If you want to use external modules for go, you can simply do:

```
$ go get 'package-repo'
```

This command does:
1. Download and save module in $GOPATH/src/package-repo
2. Execute ‘go install’

But because there are no list file(like package.json or requirements.txt) which defines dependency of your project, it is not good for coworking with your teammates. For example, when you are starting to working on some node project, you can clone the code and just run ‘npm install’ to install the dependencies, instead of installing it all one by one.

For this, most of people were recommending `dep` or `Glide`, but it seems preparing to be in state of support-only, so I prefer to go on with `dep`https://golang.github.io/dep/.


## Others
### Web framework
Like other programming languages, go also has open-source web frameworks, but it is not being recommended like Django in Python, or Rails in Ruby.

One of main reason is, Go language itself is focused on developing modern web. It has built-in modules like `net/http`, and this offers great features for easy development. For example, default `http.Serve` function spawns goroutine per each request so user don’t need to worry about multithreading. You can make great service enough without depending on full frameworks.

Instead of web framework, there are lots of middlewares for handling session, routing, and more. It will help enough for easy web development.
- http://www.gorillatoolkit.org/
- https://github.com/urfave/negroni

### Using database
If you look on DB connection examples, most of them will have code like:

```go
db, err := sql.Open("mysql","user:password@tcp(127.0.0.1:3306)/db_name")
if err != nil {
	log.Fatal(err)
}

defer db.Close()
// ...
```

It seems defining opening/closing database connection, but there are several things need to remind.

`sql.Open(...)` is not the code establishing connection to database. This is just ‘preparing’ process for later use. Actual connection occurs when it is needed for the first time.

`sql.DB` object is designed for long-living. So create one sql.DB object for each distinct datastore and pass it around as needed, or make it available somehow globally. Keep it open. Don’t `Open()` and `Close()` from a short-lived function.

It means...you don’t need to define `db.Close()` for common web project.


## Reference
* <https://golang.org>
* <https://github.com/golang/go/wiki>
