# 55 - Templates

> Text and HTML templating in Go.

---

## 📌 What You'll Learn

- text/template basics
- html/template for safe HTML
- Template syntax and actions
- Functions and pipelines
- Template inheritance

---

## 📝 text/template Basics

```go
// template_basics.go
package main

import (
    "os"
    "text/template"
)

func main() {
    // Simple template
    tmpl := template.Must(template.New("hello").Parse("Hello, {{.}}!\n"))
    tmpl.Execute(os.Stdout, "World")
    
    // Struct data
    type User struct {
        Name  string
        Email string
        Age   int
    }
    
    userTmpl := template.Must(template.New("user").Parse(`
User: {{.Name}}
Email: {{.Email}}
Age: {{.Age}}
`))
    
    user := User{Name: "Alice", Email: "alice@example.com", Age: 30}
    userTmpl.Execute(os.Stdout, user)
    
    // Map data
    mapTmpl := template.Must(template.New("map").Parse(`
Name: {{.name}}
City: {{.city}}
`))
    
    data := map[string]string{
        "name": "Bob",
        "city": "NYC",
    }
    mapTmpl.Execute(os.Stdout, data)
}
```

**Output:**
```
Hello, World!

User: Alice
Email: alice@example.com
Age: 30

Name: Bob
City: NYC
```

---

## 🔄 Template Actions

```go
// template_actions.go
package main

import (
    "os"
    "text/template"
)

type Data struct {
    Name    string
    Items   []string
    Count   int
    Active  bool
    Details map[string]string
}

func main() {
    tmpl := template.Must(template.New("actions").Parse(`
=== Conditionals ===
{{if .Active}}Active{{else}}Inactive{{end}}

{{if gt .Count 5}}Count > 5{{else if gt .Count 0}}Count between 1-5{{else}}Count is 0{{end}}

=== Loops ===
{{range .Items}}
  - {{.}}
{{end}}

{{range $index, $item := .Items}}
  {{$index}}: {{$item}}
{{end}}

=== With (change context) ===
{{with .Details}}
  City: {{.city}}
  Country: {{.country}}
{{end}}

=== Variables ===
{{$name := .Name}}
Hello, {{$name}}!

=== Comments ===
{{/* This is a comment */}}
`))

    data := Data{
        Name:   "Alice",
        Items:  []string{"apple", "banana", "cherry"},
        Count:  3,
        Active: true,
        Details: map[string]string{
            "city":    "NYC",
            "country": "USA",
        },
    }
    
    tmpl.Execute(os.Stdout, data)
}
```

**Output:**
```
=== Conditionals ===
Active

Count between 1-5

=== Loops ===
  - apple
  - banana
  - cherry

  0: apple
  1: banana
  2: cherry

=== With (change context) ===
  City: NYC
  Country: USA

=== Variables ===
Hello, Alice!

```

---

## 🔗 Pipelines and Functions

```go
// template_functions.go
package main

import (
    "os"
    "strings"
    "text/template"
)

func main() {
    // Built-in functions
    builtinTmpl := template.Must(template.New("builtin").Parse(`
=== Built-in Functions ===
Length: {{len .Items}}
Index: {{index .Items 0}}
Print: {{print "Hello" " " "World"}}
Printf: {{printf "%s is %d" .Name .Age}}
`))

    data := struct {
        Name  string
        Age   int
        Items []string
    }{
        Name:  "Alice",
        Age:   30,
        Items: []string{"a", "b", "c"},
    }
    builtinTmpl.Execute(os.Stdout, data)
    
    // Custom functions
    funcMap := template.FuncMap{
        "upper": strings.ToUpper,
        "lower": strings.ToLower,
        "add":   func(a, b int) int { return a + b },
        "repeat": func(s string, n int) string {
            return strings.Repeat(s, n)
        },
    }
    
    customTmpl := template.Must(template.New("custom").
        Funcs(funcMap).
        Parse(`
=== Custom Functions ===
Upper: {{upper .Name}}
Lower: {{lower .Name}}
Add: {{add 5 3}}
Repeat: {{repeat "Go" 3}}

=== Pipelines ===
Pipeline: {{.Name | upper}}
Chained: {{.Name | upper | printf "Hello, %s!"}}
`))

    customTmpl.Execute(os.Stdout, data)
}
```

**Output:**
```
=== Built-in Functions ===
Length: 3
Index: a
Print: Hello World
Printf: Alice is 30

=== Custom Functions ===
Upper: ALICE
Lower: alice
Add: 8
Repeat: GoGoGo

=== Pipelines ===
Pipeline: ALICE
Chained: Hello, ALICE!
```

---

## 🌐 html/template (Safe HTML)

```go
// template_html.go
package main

import (
    "html/template"
    "os"
)

func main() {
    // html/template auto-escapes dangerous content!
    
    htmlTmpl := template.Must(template.New("html").Parse(`
<!DOCTYPE html>
<html>
<head><title>{{.Title}}</title></head>
<body>
    <h1>{{.Title}}</h1>
    <p>User input: {{.UserInput}}</p>
    <a href="{{.URL}}">Link</a>
</body>
</html>
`))

    data := struct {
        Title     string
        UserInput string
        URL       string
    }{
        Title:     "My Page",
        UserInput: "<script>alert('XSS')</script>",  // Dangerous!
        URL:       "javascript:alert('XSS')",        // Dangerous!
    }
    
    // html/template escapes dangerous content automatically
    htmlTmpl.Execute(os.Stdout, data)
    
    /*
    Output (escaped, safe):
    <p>User input: &lt;script&gt;alert(&#39;XSS&#39;)&lt;/script&gt;</p>
    <a href="#ZgotmplZ">Link</a>
    */
}
```

**Output:**
```
<!DOCTYPE html>
<html>
<head><title>My Page</title></head>
<body>
    <h1>My Page</h1>
    <p>User input: &lt;script&gt;alert(&#39;XSS&#39;)&lt;/script&gt;</p>
    <a href="#ZgotmplZ">Link</a>
</body>
</html>
```

---

## 📁 Template Files

```go
// template_files.go
package main

import (
    "html/template"
    "os"
)

/*
Templates in files:

templates/
├── base.html
├── header.html
└── page.html

base.html:
{{define "base"}}
<!DOCTYPE html>
<html>
<head><title>{{template "title" .}}</title></head>
<body>
    {{template "header" .}}
    {{template "content" .}}
</body>
</html>
{{end}}

header.html:
{{define "header"}}<header>{{.SiteName}}</header>{{end}}

page.html:
{{define "title"}}{{.PageTitle}}{{end}}
{{define "content"}}<main>{{.Content}}</main>{{end}}
*/

func main() {
    // Parse multiple template files
    // tmpl := template.Must(template.ParseFiles(
    //     "templates/base.html",
    //     "templates/header.html",
    //     "templates/page.html",
    // ))
    
    // Or use glob pattern
    // tmpl := template.Must(template.ParseGlob("templates/*.html"))
    
    // Execute specific template
    // tmpl.ExecuteTemplate(os.Stdout, "base", data)
    
    // Inline example
    tmpl := template.Must(template.New("").Parse(`
{{define "base"}}
<!DOCTYPE html>
<html>
<body>
    {{template "header" .}}
    {{template "content" .}}
</body>
</html>
{{end}}

{{define "header"}}<header>Welcome to {{.Site}}</header>{{end}}

{{define "content"}}<main>{{.Body}}</main>{{end}}
`))

    data := map[string]string{
        "Site": "My Website",
        "Body": "Hello, World!",
    }
    
    tmpl.ExecuteTemplate(os.Stdout, "base", data)
}
```

**Output:**
```
<!DOCTYPE html>
<html>
<body>
    <header>Welcome to My Website</header>
    <main>Hello, World!</main>
</body>
</html>
```

---

## 🎯 Template Syntax Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│  TEMPLATE SYNTAX                                                │
├─────────────────────────────────────────────────────────────────┤
│  ACCESS                                                         │
│    {{.}}           Current value                                │
│    {{.Name}}       Field access                                 │
│    {{.Method}}     Method call                                  │
│                                                                 │
│  CONDITIONALS                                                   │
│    {{if .X}}...{{end}}                                          │
│    {{if .X}}...{{else}}...{{end}}                               │
│    {{if .X}}...{{else if .Y}}...{{end}}                         │
│                                                                 │
│  LOOPS                                                          │
│    {{range .Items}}...{{end}}                                   │
│    {{range $i, $v := .Items}}...{{end}}                         │
│                                                                 │
│  VARIABLES                                                      │
│    {{$x := .Value}}                                             │
│                                                                 │
│  FUNCTIONS                                                      │
│    {{len .Items}}                                               │
│    {{printf "%s" .Name}}                                        │
│                                                                 │
│  PIPELINES                                                      │
│    {{.Name | upper}}                                            │
│    {{.Name | upper | printf "Hello, %s"}}                       │
│                                                                 │
│  COMPARISON (must use functions)                                │
│    {{if eq .A .B}}...{{end}}                                    │
│    {{if ne .A .B}}...{{end}}                                    │
│    {{if lt .A .B}}...{{end}}                                    │
│    {{if le .A .B}}...{{end}}                                    │
│    {{if gt .A .B}}...{{end}}                                    │
│    {{if ge .A .B}}...{{end}}                                    │
│                                                                 │
│  TEMPLATES                                                      │
│    {{define "name"}}...{{end}}                                  │
│    {{template "name" .}}                                        │
│    {{block "name" .}}default{{end}}                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## ➡️ Next Steps

**Next Topic:** [56 - Signal Handling](./56-signals.md)

