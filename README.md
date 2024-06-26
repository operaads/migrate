[![GitHub Workflow Status (branch)](https://img.shields.io/github/actions/workflow/status/operaads/migrate/ci.yaml?branch=master)](https://github.com/operaads/migrate/actions/workflows/ci.yaml?query=branch%3Amaster)
[![GoDoc](https://pkg.go.dev/badge/github.com/operaads/migrate)](https://pkg.go.dev/github.com/operaads/migrate/v4)
[![Coverage Status](https://img.shields.io/coveralls/github/operaads/migrate/master.svg)](https://coveralls.io/github/operaads/migrate?branch=master)
[![packagecloud.io](https://img.shields.io/badge/deb-packagecloud.io-844fec.svg)](https://packagecloud.io/operaads/migrate?filter=debs)
[![Docker Pulls](https://img.shields.io/docker/pulls/migrate/migrate.svg)](https://hub.docker.com/r/migrate/migrate/)
![Supported Go Versions](https://img.shields.io/badge/Go-1.19%2C%201.20-lightgrey.svg)
[![GitHub Release](https://img.shields.io/github/release/operaads/migrate.svg)](https://github.com/operaads/migrate/releases)
[![Go Report Card](https://goreportcard.com/badge/github.com/operaads/migrate/v4)](https://goreportcard.com/report/github.com/operaads/migrate/v4)

# migrate

__Database migrations written in Go. Use as [CLI](#cli-usage) or import as [library](#use-in-your-go-project).__

* Migrate reads migrations from [sources](#migration-sources)
   and applies them in correct order to a [database](#databases).
* Drivers are "dumb", migrate glues everything together and makes sure the logic is bulletproof.
   (Keeps the drivers lightweight, too.)
* Database drivers don't assume things or try to correct user input. When in doubt, fail.

Forked from [mattes/migrate](https://github.com/mattes/migrate)

## Databases

Database drivers run migrations. [Add a new database?](database/driver.go)

* [SQLite](database/sqlite)
* [SQLite3](database/sqlite3) ([todo #165](https://github.com/mattes/migrate/issues/165))
* [MySQL/ MariaDB](database/mysql)

### Database URLs

Database connection strings are specified via URLs. The URL format is driver dependent but generally has the form: `dbdriver://username:password@host:port/dbname?param1=true&param2=false`

Any [reserved URL characters](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters) need to be escaped. Note, the `%` character also [needs to be escaped](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_the_percent_character)

Explicitly, the following characters need to be escaped:
`!`, `#`, `$`, `%`, `&`, `'`, `(`, `)`, `*`, `+`, `,`, `/`, `:`, `;`, `=`, `?`, `@`, `[`, `]`

It's easiest to always run the URL parts of your DB connection URL (e.g. username, password, etc) through an URL encoder. See the example Python snippets below:

```bash
$ python3 -c 'import urllib.parse; print(urllib.parse.quote(input("String to encode: "), ""))'
String to encode: FAKEpassword!#$%&'()*+,/:;=?@[]
FAKEpassword%21%23%24%25%26%27%28%29%2A%2B%2C%2F%3A%3B%3D%3F%40%5B%5D
$ python2 -c 'import urllib; print urllib.quote(raw_input("String to encode: "), "")'
String to encode: FAKEpassword!#$%&'()*+,/:;=?@[]
FAKEpassword%21%23%24%25%26%27%28%29%2A%2B%2C%2F%3A%3B%3D%3F%40%5B%5D
$
```

## Migration Sources

Source drivers read migrations from local or remote sources. [Add a new source?](source/driver.go)

* [Filesystem](source/file) - read from filesystem
* [io/fs](source/iofs) - read from a Go [io/fs](https://pkg.go.dev/io/fs#FS)

## CLI usage

* Simple wrapper around this library.
* Handles ctrl+c (SIGINT) gracefully.
* No config search paths, no config files, no magic ENV var injections.

__[CLI Documentation](cmd/migrate)__

### Basic usage

```bash
$ migrate -source file://path/to/migrations -database postgres://localhost:5432/database up 2
```

## Use in your Go project

* API is stable and frozen for this release (v3 & v4).
* Uses [Go modules](https://golang.org/cmd/go/#hdr-Modules__module_versions__and_more) to manage dependencies.
* To help prevent database corruptions, it supports graceful stops via `GracefulStop chan bool`.
* Bring your own logger.
* Uses `io.Reader` streams internally for low memory overhead.
* Thread-safe and no goroutine leaks.

__[Go Documentation](https://pkg.go.dev/github.com/operaads/migrate/v4)__

```go
import (
    "github.com/operaads/migrate/v4"
    _ "github.com/operaads/migrate/v4/database/mysql"
    _ "github.com/operaads/migrate/v4/source/github"
)

func main() {
    m, err := migrate.New(
        "github://mattes:personal-access-token@mattes/migrate_test",
        "mysql://localhost:5432/database?sslmode=enable")
    m.Steps(2)
}
```

Want to use an existing database client?

```go
import (
    "database/sql"
    _ "github.com/lib/pq"
    "github.com/operaads/migrate/v4"
    "github.com/operaads/migrate/v4/database/mysql"
    _ "github.com/operaads/migrate/v4/source/file"
)

func main() {
    db, err := sql.Open("mysql", "postgres://localhost:5432/database?sslmode=enable")
    driver, err := postgres.WithInstance(db, &postgres.Config{})
    m, err := migrate.NewWithDatabaseInstance(
        "file:///migrations",
        "postgres", driver)
    m.Up() // or m.Step(2) if you want to explicitly set the number of migrations to run
}
```

## Getting started

Go to [getting started](GETTING_STARTED.md)

(more tutorials to come)

## Migration files

Each migration has an up and down migration. [Why?](FAQ.md#why-two-separate-files-up-and-down-for-a-migration)

```bash
1481574547_create_users_table.up.sql
1481574547_create_users_table.down.sql
```

[Best practices: How to write migrations.](MIGRATIONS.md)

## Coming from another db migration tool?

Check out [migradaptor](https://github.com/musinit/migradaptor/).
*Note: migradaptor is not affliated or supported by this project*

## Versions

Version | Supported? | Import | Notes
--------|------------|--------|------
**master** | :white_check_mark: | `import "github.com/operaads/migrate/v4"` | New features and bug fixes arrive here first |
**v4** | :white_check_mark: | `import "github.com/operaads/migrate/v4"` | Used for stable releases |
**v3** | :x: | `import "github.com/operaads/migrate"` (with package manager) or `import "gopkg.in/operaads/migrate.v3"` (not recommended) | **DO NOT USE** - No longer supported |

## Development and Contributing

Yes, please! [`Makefile`](Makefile) is your friend,
read the [development guide](CONTRIBUTING.md).

Also have a look at the [FAQ](FAQ.md).

---

Looking for alternatives? [https://awesome-go.com/#database](https://awesome-go.com/#database).
