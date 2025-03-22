## Parsing the templates

```go
tmpl := template.Must(template.ParseGlob("templates/*.html"))
```

Warn: templates will be executed at runtime, the project can compile without a template existing and you will not see the error until the panic.

Big Warn: you need to parse a single template, so if you want to use a base template, you need to parse files sequentially

## Naming a template

Default name is the file name, but a custom name can also be defined:

```
{{define "yourName"}}
...content...
{{end}}
```

## Using a template

```go
tmpl.ExecuteTemplate(writer, name, data)
```

`name` can be filename or the name used with `define.`

## Including templates

```
{{define "footer"}}
<footer>
  Foo Ter
</footer>
{{end}}
```

In another template:

```
<body>
  {{template "footer"}}
</body>
```

## Passing data around
```go
http.HandleFunc("/fofo", func(w http.ResponseWriter, r *http.Request) {
    mapp := make(map[string]string)
    mapp["name"] = "Fofo"
    mapp["age"] = "20"
    data := struct {
        Date  time.Time
        Map   map[string]string
        Array []int
    }{
        Date:  time.Now(),
        Array: []int{1, 2, 3, 4},
        Map:   mapp,
    }
    _ = tmpl.ExecuteTemplate(w, "withData", data)
})
```

Any kind of data can be passed.

```
{{define "withData"}}
<p>
    Top level data is accessed with a single dot: {{.}}
</p>

<p>
    Date is now: <pre> {{.Date}} </pre>
</p>

<p>
    Iterating:
    {{range .Array}}
    <pre>{{.}}</pre>
    {{end}}
</p>

<p>
    Map:
    {{range $key, $value := .Map}}
    <pre>{{printf "%s: %s" $key $value}}</pre>
    {{end}}
</p>

<p>
    Choosing a map/array item:
    <pre>Array item 1: {{index .Array 1}}</pre>
    <pre>Map item "name": {{index .Map "name"}}</pre>
</p>
{{end}}
```

In case you include other templates, you can do something like `{{template "yourName" .}}` to pass the entire data, otherwise choose what to send.

## Conditionals

```
{{if gt .Multiplier 3}}
  <p>Big Bonus</b>
{{else}}
  <p>Normal Bonus</b>
{{end}}
```

## Adding template functions

```go
// somewhere in your code
func toUpper(str string) string {
	return strings.ToUpper(str)
}

func dateIsoFormat(t time.Time) string {
	return t.Format(time.RFC3339)
}
```

```go
// where you instance the templates
funcMap := template.FuncMap{
    "toUpper":       toUpper,
    "dateIsoFormat": dateIsoFormat,
}
tmpl := template.Must(template.New("").Funcs(funcMap).ParseGlob("templates/*.html"))
```

Careful as Funcs must be passed before parsing the template files.

## Further references

* https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax