---
title: "Handling Reference to Loop Iterator Variable in Go"
date: 2022-12-07T12:05:03-03:00
---

## Introduction

Go is a statically typed compiled programmin language that was created by Google back in 2009.
Since [Go](https://go.dev/) is a very generic world if you search for `Go` on Google you are going to get a lot
of results that have nothing to do with the awesome Go programming language. To avoid that
issue people, when searching things related to programming use `golang` and some people think
the name of the language is that one because it is so common on Google, but that is not the
case, the real name of the programming language is just `Go`. Ok, this has nothing to do with
I am going to present you below but I wanted to write a little about that for myself. Thank you
for your attention so far. Now, let's do what we have come to do here, talk about handling
reference to loop iterator variable in Go.

I am kinda new to Go and I say _new_ that way because it wasn't recently, like a month or two ago
that I have started coding in Go. The first I used Go was back in 2018. For some reason, I don't
know exactly what I was looking for but I ended up finding about Go on the interned and decided
to experiment with and liked the language and used the language for about six months back then.
After that period I moved away from Go not because I did not the language. I like this language a lot,
but I was working with other stuff like JavaScript, React and Node, but now at the end of 2022
I have decided to invest some time on Go again because I wanted to focus more on back-end development
and Go has some nice characteristics that I really like. Those are:

- It is a compiled language. That means you write your code, compile id down to a native binary and
  ship that binary to production.
- It is statically typed.
- It is simple, in my opnion. It is a language you can learn enough to ship something in less than
  a week
- It comes with a nice standard library
- It is opnionated on things like formatting, unused vars, modules and so on
  - It has a tool that formats the code for you
  - If you declare a variable and do not use it your code will not compile

## How I ended up here

I am building a REST API in Go using [Chi](https://github.com/go-chi/chi) for routing and
[pgx](https://github.com/jackc/pgx) + [sqlx](https://github.com/jmoiron/sqlx) for database
access. PostgreSQL, my go-to database, is being used to store the data.

This is my first time building a REST API with Go. How to structure that king of project in Go?
Well, I really didn't know before starting this project. What I did was looking up some examples
of repositories on Github and then copy that one into my project. Imagine you're new to a programming
language but you already have experience building the same kind of stuff on other languages, would you
build everying from scratch? I chose not to do that but instead use a boilerplate that cover most the
things I need when building an API. I have achieved that with the following repositories as an starting
point:

- [https://github.com/qiangxue/go-rest-api](https://github.com/qiangxue/go-rest-api)
- [https://github.com/qreasio/go-starter-kit](https://github.com/qreasio/go-starter-kit)
- [https://github.com/sentrionic/Valkyrie](https://github.com/sentrionic/Valkyrie)

The API, so far, is a CRUD on the users table. Bear with me, this is not a simple CRUD. It is an 
awesome REST API that allows you to create, read, update and delete users. That API comes with
pagination so that you won't be fetching the users table but limiting the result to just 10 or 20
rows when you hit the `GET /api/users` endpoint. What happens when you hit this endpoint is the
following: You have the option to set pagination params which are `page` (the page number) and
`per_page` (size of the list of users returned). By default `page` is 1 and `per_page` is 20.
With pagination being passed, the enpoint becomes `GET /api/users?page=1&per_page20`.

With Chi, you write the handler and associate that handler with a route by doing the following:

```go
// ... resource definition
func (r resource) listUsers(w http.ResponseWriter, req *http.Request) {
  users, err := r.service.Query(0, 20)
  ...
}

router := chi.NewRouter()
r.Get("/api/users", listUsers)
// ...
```

That `listUsers` handler function, call the users service, which
in turn calls the users repository. The user repository has a
`GetUsers` function which sends a query to the database to retrieve
the users.

```go
// ... other stuff like `userQueries` definition and db connection above
func (q *userQueries) GetUsers(offset, limit int) ([]entity.User, error) {
	users := []entity.User{}

	query := `SELECT * FROM users ORDER BY id LIMIT $1 OFFSET $2`

	err := q.db.Select(&users, query, limit, offset)

	return users, err
}
// ...
```

So far, everything works as expected. Now comes the part where I got in trouble
with reference to Loop Iterator Variable.

## Reference to Loop Iterator Variable in Go

When returning data to the client it is a good practice to have a data
representation with the properties you want the client to access. Returning
the user password in the response is not cool. My API has a struct
name `UserResponse` which is responsible for that. The result from invoking
the repository function `GetUsers` is a slice of `entity.User`. An `entity.User`
object has the password in it. The handler needs to turn each `entity.User` into
a `UserResponse`. Here is the `UserResponse` definition

```go
type UserResponse struct {
  entity.User
  Password    bool `json:"-"`
}
```

If you are wondering why I am creating the `Password` property again within `UserResponse` and
setting it as a bool, you can learn more about it on 
[JSON and struct composition in Go](https://attilaolah.eu/2014/09/10/json-and-struct-composition-in-go).

Here is the code on the handler that creates a slice of `UserResponse`.

```go
// ... service definition, user response and so on
func (s service) Query(offset int, limit int) ([]User, error) {
	users, err := s.repo.GetUsers(offset, limit)
	if err != nil {
		return nil, err
	}

	var result []UserResponse

	for _, u := range users {
		result = append(result, UserResponse{User: &u})
	}

	return result, nil
}
```

That looks great but it does not work properly because I am assuming here that
`u` from `_, u :: ranage users { }` is a variable containing a different user
on each iteration. That assumption is correct. Indeed for each iteration I get
a different user. The problem is how I use that variable `u` when appending it
to the `result` slice. After I finished writing this code and restarted the server
I hit that endpoint and got a list back with three users, which were exactly the
number of users I had in the users table. To my surprise, unfortunately, the list
had the same user three times, which was the last user. I had no idea what was going
on and tried different approaches to fix this but none helped until I figured out [the 
real issue](https://github.com/golang/go/wiki/CommonMistakes).

When iterating slices, you assign each element (a user object in my case) to the loop
iterator variable. That loop variable `u` is the same for the whole loop. It will be the
same when the first user is assigned to it as well as when the last user is assigned to it.
Be being the same variable I mean it has the same memory address. Each iteration of
the loop you assign a user to that address. On the first iteration you have the first user
assigned to `u`. So `u` is how you access the first user. The `u` address is `0x40e020`. 
However, on the next loop iteration `u` gets assigned the second user. Now you no longer
can access the first user because `u` points to the second one. On the first iteration
that user was appended to the `result` slice doing this `result = append(result, User{User: &u})`.
That does not what it is expected to do because `&u` is keeping reference to the current user.
What gets appended to the slice is the `u` address. The `u` refers to the first user in the
first iteration. On the second iteration the same happens, this time for the second user. `u`
is appended again to the slice. By the end of the second iteration the slice contains to references,
right? That is partially right. What I mean by partially is though the slice has two elements,
those two elements points to the same location, which the `u` location. So you get the same
address for the first and second element in the slice. When showing up that data to the client
you end up displaying the same user no matter how many users you have to return. That user
that is repeated on the response is always the last user in the `users` slice because on the
last iteration over `users` `u` got assigned the last user and since all elements in the
`result` slice have the same address.

Here is the solution I was able to figure out after reading that `the real issue` lin above.

```go
for _, u := range users {
  newU := u
  result = append(result, UserResponse{User: newU})
}
```

To keep track of the current user a new variable `newU` is created and the current user
is assigned to it. Since `:=` is being used a new address is generated and that new one
is appended to the slice. Now each user has its own reference.
