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
tmpl := template.Must(template.New("hello").Parse("Hello, {{.}}!\n"))
tmpl.Execute(os.Stdout, "World")
// Output: Hello, World!

type User struct { Name, Email string; Age int }
userTmpl := template.Must(template.New("user").Parse(`
User: {{.Name}}
Email: {{.Email}}
Age: {{.Age}}
`))
userTmpl.Execute(os.Stdout, User{Name: "Alice", Email: "alice@example.com", Age: 30})
// Output: User: Alice, Email: alice@example.com, Age: 30

mapTmpl := template.Must(template.New("map").Parse("Name: {{.name}}\nCity: {{.city}}\n"))
mapTmpl.Execute(os.Stdout, map[string]string{"name": "Bob", "city": "NYC"})
// Output: Name: Bob, City: NYC
```

---

## 🔄 Template Actions

```go
// Conditionals: {{if .Active}}Active{{else}}Inactive{{end}}
// Loops: {{range .Items}} - {{.}}{{end}}
// With: {{with .Details}}City: {{.city}}{{end}}
// Variables: {{$name := .Name}}Hello, {{$name}}!
// Comments: {{/* comment */}}
```

---

## 🔗 Pipelines and Functions

```go
// Built-in: len, index, print, printf
// Custom functions
funcMap := template.FuncMap{
    "upper": strings.ToUpper,
    "add":   func(a, b int) int { return a + b },
}
tmpl := template.Must(template.New("").Funcs(funcMap).Parse(`
Upper: {{upper .Name}}
Add: {{add 5 3}}
Pipeline: {{.Name | upper}}
`))
// Output: Upper: ALICE, Add: 8, Pipeline: ALICE
```

---

## 🌐 html/template (Safe HTML)

```go
// html/template auto-escapes dangerous content!
data := struct {
    UserInput string
}{
    UserInput: "<script>alert('XSS')</script>",
}
htmlTmpl.Execute(os.Stdout, data)
// Output: &lt;script&gt;alert(&#39;XSS&#39;)&lt;/script&gt; (escaped, safe)
```

---

## 📁 Template Files

```go
tmpl := template.Must(template.ParseFiles("base.html", "header.html", "page.html"))
tmpl.ExecuteTemplate(os.Stdout, "base", data)

// Or: template.ParseGlob("templates/*.html")
```

---

## 📋 Template Syntax Quick Reference

| Syntax | Purpose |
|--------|---------|
| `{{.}}` | Current value |
| `{{.Name}}` | Field access |
| `{{if .X}}...{{end}}` | Conditionals |
| `{{range .Items}}...{{end}}` | Loops |
| `{{$x := .Value}}` | Variables |
| `{{.Name \| upper}}` | Pipelines |
| `{{if eq .A .B}}` | Comparison (eq, ne, lt, le, gt, ge) |
| `{{define "name"}}...{{end}}` | Named templates |
| `{{template "name" .}}` | Include template |

---

## ➡️ Next Steps

**Next Topic:** [56 - Signal Handling](./56-signals.md)
