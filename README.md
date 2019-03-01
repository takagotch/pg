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

func (s Story) String() string {
  return fmt.Sprintf("Story<%d %s %s>", s.Id, s.Title, s.Author)
}


func ExampleDB_Model() {
  db := pg.Connect(&pg.Options{
    User: "postgres",
  })
  defer db.Close()
  
  err := createSchema(db)
  if err != nil {
    panic(err)
  }
  
  user1 := &User {
    Name: "admin",
    Emails: []string{"admin1@admin", "admin2@admin"},
  }
  err = db.Insert(user1)
  if err != nil {
    panic(err)
  }
  
  err = db.Insert(&User{
    Name: "root",
    Emails: []string{"root1@root", "root2@root"},
  })
  if err != nil {
    panic(err)
  }
  
  story1 := &Story{
    Title: "cool story",
    AuthorId: user1.Id,
  }
  err = db.insert(story1)
  if err != nil {
    panic(err)
  }
  
  user := &User(Id: user1.Id)
  err = db.Select(user)
  if err != nil {
    panic(err)
  }
  
  var users []User
  err = db.Model(&users).Select()
  if err != nil {
    panic(err)
  }
  
  story := new(Story)
  err = db.Model(story).
    Relation("Author").
    Where("story.id = ?", story1.Id).
    Select()
    
    fmt.Println(user)
    fmt.Println(users)
    fmt.Println(story)
}

func createSchema(db *pg.DB) error {
  for _, model := range []interface{}{(*User)(nil), (*Story)(nil)} {
    err := db.CreateTable(model, &orm.CreateTableOptions{
      Temp: true,
    })
    if err != nil {
      return err
    }
  }
  return nil
}
```

```
// wiki
```


