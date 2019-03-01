### pg
---
https://github.com/go-pg/pg

https://github.com/go-pg/pg/wiki/Writing-Queries

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

```go
package pg_test

import (
  "fmt"
  
  "github.com/go-pg/pg"
)

type Params struct {
  X int
  Y int
}

func (p *Params) Sum() int {
  return p.X + p.Y
}

func Example_placeholders() {
  var num int
  
  _, err := db.Query(pg.Scan(&num), "SELECT ?", 42)
  if err != nil {
    panic(err)
  }
  fmt.Println("simple:", num)
  
  _, err = db.Query(pg.Scan(&num), "SELECT ?0 + ?0", 1)
  if err  != nil {
    panic(err)
  }
  fmt.Println("simple:", num)
  
  _, err = db.Query(pg.Scan(&num), "SELECT ?0 + ?0", 1)
  if err != nil {
    panic(err)
  }
  fmt.Println("indexed:", num)
  
  _, err = db.Query(pg.Scan(&num), "SELECT ?x + ?y + ?Sum")
  if err != nil {
    panic(err)
  }
  fmt.Println("named:", num)
  
  _, err = db.Query(pg.Scan(&num), "SELECT ?x + ?y + ?Sum", params)
  if err != nil {
    panic(err)
  }
  fmt.Println("named:", num)
  
  _, err = db.WithParam("z", 1).Query(pg.Scan(&num), "SELECT ?x + ?y + ?z", params)
  if err != nil {
    panic(err)
  }
  fmt.Println("global:", num)
  
  var tableName, tableAlias, columns string
  _, err = db.Model(&Params{}).Query(
    pg.Scan(&tableName, &tableAlias, &columns),
    "SELECT '?TableName', '?TableAlias', '?Columns'"
  )
  if err != nil {
    panic(err)
  }
  fmt.Println("table name:", tableName)
  fmt.Pringln("table alias:", tableAlias)
  fmt.Println("columns:", columns)
}

book := &Book{Id: 1}
err := db.Select(book)


book := new(Book)
err := db.Model(book).Where("id = ?", 1).Select()

err := db.Model(book).Column("title", "text").Where("id = ?", 1).Select()


var title, text string
err := db.Model((*Book)(nil)).
  Column("title", "text").
  Where("id = ?", 1).
  Select(&title, &text)
// SELECT "title", "text"
// FROM "books" WHERE id = 1


err := db.Model(book).
  Where("id > ?", 100).
  Where("title LIKE ?", "my%")
  Limit(1).
  Select()
// SELECT "book"."id", "book"."title", "book"."text"
// FROM "books"
// WHERE (id > 100) AND (title LIKE 'my%')
// LIMIT 1

err := db.Model(book).
  Where("id > ?", 100).
  WhereOr("title LIKE ?", "my%").
  Limit(1).
  Select()
// SELECT "book"."id", "book"."title", "book"."text"
// FROM "books"
// WHERE (id > 100) OR (title LIKE 'my%')
// LIMIT 1


err := db.Model(book).
  Where("title LIKE ?", "my%").
  WhereGroup(func(q *orm.Query) (*orm.Query, error) {
    q = q.WhereOr("id = 1").
      WhereOr("id = 2")
    return q, nil
  }).
  Limit(1).
  Select()
  
  
var books []Book
err := db.Model(&books).Order("id ASC").Limit(20).Select()
// SELECT "book"."id", "book"."title", "book"."text"
// FROM "books"
// ORDER BY id ASC LIMIT 20


count, err := db.Model((*Book)(nil)).Count()

count, err := db.Model(&books).Limit(20).SelectAndCount()

count, err := db.Model(&books).Limit(20).SelectAndCountEstimate(100000)

var res []struct {
  AuthorId int
  BookCount int
}
err := db.Model((*Book)(nil)).
  Column("author_id").
  ColumnExpr("count(*) AS book_count").
  Group("author_id").
  Order("book_count DESC").
  Select(&res)


var ids []int
err := db.Model((*Book)(nil)).ColumnExpr("array_agg(id)").Select(pg.Array(&ids))

ids := []int{1, 2, 3}
err := db.Model((*Book)(nil)).
  Where("id in (?)", pg.In(ids)).
  Select()


book := &Book{}
ids := []int{1, 2, 3}
err := db.Model(book).
  Where("id = ?", 1).
  For("UPDATE").
  Select()
// SELECT * FROM books WHERE id = 1 FOR UPDATE


book := new(Book)
err := db.Model(book).
  ColumnExpr("book.*").
  ColumnExpr("a.id AS author__id, a.name AS author__name").
  Join("JOIN authors AS a ON a.id = book.author_id").
  First()
// SELECT book.*, a.id AS author__id, a.name AS author__name
// FROM books
// JOIN authors AS a ON a.id = book.author_id
// ORDER BY id LIMIT 1


q.Join("LEFT JOIN authors AS a")
  JoinOn("a.id = book.author_id").
  JoinOn("a.active = ?", true)

authorBooks := db.Model((*Book)(nil)).Where("author_id = ?", 1)

err := db.Model().
  With("author_books", authorBooks).
  Table("author_books").
  Select(&books)


err := db.Model(&books).
  Where("author_id = ?", 1).
  WrapWith("author_books").
  Table("author_books").
  Select(&books)


authorBooks := db.Model((*Book)(nil)).Where("author_id = ?", 1)
err := db.Model(nil).TableExpr("(?)", author_books).Select(&book)

err := db.Model(book).Column("Author").Select()

err := db.Model(book).Column("book.id", "Author.id").Select()

err := db.Model(book).Column("Author._").Select()

err := db.Model(book).Column("_", "Author").Select()

var books []Book
err := db.Model(&books).
  Apply(orm.Pagination(req.URL.Query())).
  Select()
  
type BookFilter struct {
  orm.Pager
  AuthorID int64
}

func (f *BookFilter) Filter(q *orm.Query) (*orm.Query, error) {
  if f.AuthorID > 0 {
    q = q.Where("b.author_id = ?", f.AuthorID)
  }
  
  f.Pager.MaxLimit = 100
  f.Pager.MaxOffset = 100000
  
  q = q.Apply(f.Pager.Paginate)
  
  return q, nil
}

var filter BookFilter
filter.Pager.SetURLValues(req.URL.Query())
filter.AuthorID = 10

var books []Book
err := db.Model(&books).Apply(filter.Filter).Select()


err := db.Insert(book)

err := db.Model(book).Returning("*").Insert()

err := db.Model(book1, book2).Insert()

_, err := db.Model(book).
  Where("title = ?title").
  OnConflict("DO NOTHING").
  SelectOrInsert()
  
  
_, err := db.Model(book).
  OnConflict("(id) DO UPDATE").
  Set("title = EXCLUDED.title")
  Insert()

err := db.Update(book)

res, err := db.Model(book).Set("title = ?title").Where("id = ?id").Update()
```


