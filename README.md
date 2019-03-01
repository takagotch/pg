### pg
---
https://github.com/go-pg/pg

```
go get -u github.com/go-pg/pg
```

```go
package pg_test

import (
  "fmt"
  
  "github.com/go-pg/pg"
  "github.com/go-pg/pg/orm"
)

type User struct {
  Id int64
  Name string
  Emails []string
}

func (u User) String() string {
  return fmt.Sprintf("User<%d %s %v>", u.Id, u.Name, u.Emails)
}

type Story struct {
  Id int64
  Title string
  AuthorId int64
  Author *User
}

func () String() string {

}


func ExampleDB_Model() {


}

func createSchema(db *pg.DB) error {

}
```

```
// wiki
```


