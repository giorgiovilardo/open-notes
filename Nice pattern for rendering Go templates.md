We can shove the templates inside a separate package, that will have this shape (consider moving gohtml + css into different dir for embed purposes):

```
❯ tree frontend
frontend
├── about.gohtml
├── base.gohtml
├── frontend.go
├── home.gohtml
└── output.css
```

`frontend.go` will basically be the gateway for all renders; I attach a copy of a file setup for [Echo:](https://echo.labstack.com/)

```go
package frontend

import (
	"embed"
	"github.com/labstack/echo/v4"
	"html/template"
	"io"
)

//go:embed *
var files embed.FS

const (
	Home  = "home"
	About = "about"
)

type Frontend struct {
	templates map[string]*template.Template
}

func NewFrontend() *Frontend {
	parse := func(file string) *template.Template {
		return template.Must(
			template.New(file).ParseFS(files, "base.gohtml", file))
	}
	f := &Frontend{}
	f.templates = make(map[string]*template.Template)
	f.templates[Home] = parse("home.gohtml")
	f.templates[About] = parse("about.gohtml")
	return f
}

func (f *Frontend) Render(w io.Writer, name string, data interface{}, c echo.Context) error {
	return f.templates[name].ExecuteTemplate(w, "base.gohtml", data)
}
```

`Render()` is just a specific Echo impl that is then hooked up to Echo renderer:

```go
e := echo.New()
tmpl := frontend.NewFrontend()
e.Renderer = tmpl
```

The basic idea is to control template generation in a single place, in this case creating a `PageName: Template` k:v inheriting from the base template. We can then render like this in the handlers, using the consts like an enum:

```go
e.GET("/", func(c echo.Context) error {
	return c.Render(http.StatusOK, frontend.Home, nil)
})

e.GET("/about", func(c echo.Context) error {
    return c.Render(http.StatusOK, frontend.About, nil)
})
```

This has the drawback of leaving `data` without type safety, but we can add it by indirecting a little bit;

```go
type IndexParams struct {
	Name string
}

// copy Echo's Render signature as retvals
func Index(context IndexParams) (int, string, interface{}) {
	return http.StatusOK, "home", context
}
```

Now we have IDE help, it's not possible to invoke frontend.Index without the correct context.

```go
e.GET("/", func(c echo.Context) error {
    return c.Render(frontend.Index(frontend.IndexParams{
        Name: "Home",
    }))
})
```

We can use the same technique when we use stdlib:

```go
home = parse("home.gohtml")

type IndexParams struct {
    Name string
}

func Index(w io.Writer, context IndexParams) {
    return home.Execute(w, context)
}
```

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    context := frontend.IndexParams{Name: "Foo"}
    frontend.Index(w, context)
})
```

We wrap and abstract away the templates, leading to type safety.