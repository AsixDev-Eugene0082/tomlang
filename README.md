# TOML parser for Go with reflection

TOML stands for Tom's Obvious, Minimal Language.

Spec: https://github.com/mojombo/toml

Documentation: http://godoc.org/github.com/BurntSushi/toml

Installation:

```bash
go get github.com/BurntSushi/toml
```

Try the toml checker:

```bash
go get github.com/BurntSushi/toml/tomlcheck
tomlcheck some-toml-file.toml
```

## Examples

This package works similarly to how the Go standard library handles `XML`
and `JSON`. Namely, data is loaded into Go values via reflection.

For the simplest example, consider some TOML file as just a list of keys
and values:

```toml
Age = 25
Cats = [ "Cauchy", "Plato" ]
Pi = 3.14
Perfection = [ 6, 28, 496, 8128 ]
DOB = 1987-07-05T05:45:00Z
```

Which could defined in Go as:

```go
type Config struct {
  Age int
  Cats []string
  Pi float64
  Perfection []int
  DOB time.Time // requires `import time`
}
```

And then decoded with:

```go
var conf Config
if err := toml.Decode(tomlData, &conf); err != nil {
  // handle error
}
```

You can also use struct tags if your struct field name doesn't map to a TOML
key value directly:

```toml
some_key_NAME = "wat"
```

```go
type TOML struct {
  ObscureKey string `toml:"some_key_NAME"`
}
```

## More complex usage

Here's an example of how to load the example from the official spec page:

```toml
# This is a TOML document. Boom.

title = "TOML Example"

[owner]
name = "Tom Preston-Werner"
organization = "GitHub"
bio = "GitHub Cofounder & CEO\nLikes tater tots and beer."
dob = 1979-05-27T07:32:00Z # First class dates? Why not?

[database]
server = "192.168.1.1"
ports = [ 8001, 8001, 8002 ]
connection_max = 5000
enabled = true

[servers]

  # You can indent as you please. Tabs or spaces. TOML don't care.
  [servers.alpha]
  ip = "10.0.0.1"
  dc = "eqdc10"

  [servers.beta]
  ip = "10.0.0.2"
  dc = "eqdc10"

[clients]
data = [ ["gamma", "delta"], [1, 2] ] # just an update to make sure parsers support it

# Line breaks are OK when inside arrays
hosts = [
  "alpha",
  "omega"
]
```

And the corresponding Go types are:

```go
type tomlConfig struct {
	Title string `toml:"title"`
	Owner ownerInfo `toml:"owner"`
	DB database `toml:"database"`
	Servers map[string]server `toml:"servers"`
	Clients clients `toml:"clients"`
}

type ownerInfo struct {
	Name string `toml:"name"`
	Org string `toml:"organization"`
	Bio string `toml:"bio"`
	DOB time.Time `toml:"dob"`
}

type database struct {
	Server string `toml:"server"`
	Ports []int `toml:"ports"`
	ConnMax int `toml:"connection_max"`
	Enabled bool `toml:"enabled"`
}

type server struct {
	IP string `toml:"ip"`
	DC string `toml:"dc"`
}

type clients struct {
	Data [][]interface{} `toml:"data"`
	Hosts []string `toml:"hosts"`
}
```

A working example of the above can be found in `_examples/example.{go,toml}`.

