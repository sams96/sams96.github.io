---
title: Applying Hexagonal Architecture to a Mid-Size Go Backend
url: /go-project-layout
date: 2025-02-26
categories:
  - programming
tags:
  - golang
tootLink: "https://toot.io/@mondoman712/114071633084030867"
---

I've been working on web backends in Go for the past 5 years, and on various
projects of covering a range of ages from brand new greenfield projects to one
built on top of a core that predates the Go language. The needs of these
backends tend not to be overly complex, handling user management, payments, and
a relativly simple product. They're maintained by teams of 2-20 backend
engineers and work with a SPA web frontend communicating with JSON over HTTP.

With my most recent work being mostly greenfield, I've been thinking more about
how I can apply what I've learned from my colleagues working on older projects,
and avoid decisions made in the name of moving fast which cause a lot of pain
further down the line.

<!--more--></br>

What needs to be considered when it comes to layout out a project is finding the
right balance of flexibility without over-abstracting. You want to be able to
move quickly and adapt to ever-changing requirements without being slowed down
by your own code. This heavily depends on what you're working on, but I'm going
present what I find works for me for the type of projects I have experience
working on.

I arrived at this layout not purely by attempting to apply hexagonal
architecture to the projects I have worked on. It is something that I have read
about and is often brought up when it comes to Go project layouts, but I have
arrived here mostly through my experience and by thinking about how to find the
right level of abstraction that works well for these projects.

Hexagonal architecture splits packages into one of two types: the application's
core and its adapters. The core being the part that defines your application's
business logic and ties the adapters together, and the adapters being wrappers
around external services or other external communication. I would further add a
third type to this, which is utility packages that fill neither of the above
roles.

## Core

Core packages should contain all of your main type definitions which the
adapters will also be built around, as well as functions defining the actions
that can take place. In the simplest case this could just be a wrapper around a
database function, but it could also involve much more, including validation and
coordination of different adapters.

You can split your core where you have clear seperation between domains. A
simple example of this is splitting the user management away from the actual
functionality of your application (assuming that the functionailty isn't heavily
relieant on user management). If your application is small, you can just leave
it all in one package. If you aren't sure, just leave it in one package until
clear lines emerge[^1].
[^1]: [Don't factor early](https://grugbrain.dev/#grug-on-factring-your-code)

Below I have used a simple user management package as an example framework for a
core package. I have omitted contexts (which you really should be passing) and
imports for brevity. _All code samples in this article are for illustrative
purposes only, don't expect them to work as is_.

```go
package users

type User struct {
	ID       uuid.UUID
	Email    string
}

type UserStorer interface {
	NewUser(email, password string) (*User, error)
}

type Manager struct {
	store UserStorer
}

func NewManager(store UserStorer) *Manager {
	return &Manager{
		store: store,
	}
}

func (m *Manager) New(email, password string) (*User, error) {
	...
}
```

The `UserStorer` here will likely only have one definition. Even though as
developers we like the idea of having flexibility to swap out the database
easily, in my experience in a commercial setting it's very unlikely to happen.
I also don't think it's really needed for testing, I would just write
integration tests here and run against the DB instead of mocking it. That said,
I like to have the interface defined here anyway because it fits nicely with the
philosphy of core package and having it define what the adapters do.

Ideally, your core packages won't be dependant on each other and you can keep
everything entirely separate, however this often isn't the case. In some simple
cases, you can just rely on the layer above (the driving adapter, see below) to
call separate functions in different core packages. For more complex
dependencies, I think it's fine for the core packages to sometimes be dependant
on one another (i.e my user manager might include a manager of another domain).
The Go compiler's blocking of circular dependencies should prevent things from
getting too complex here.

## Adapters

Adapters are what let your core code communicate with the outside world. They
generally exist either as a wrapper around some third party service in order for
it to conform to an interface that has been defined in the core, the obvious
example being your database layer, or as an API calling the functions in your
core package. These are sometimes referred to as _driven_ and _driving_
adapters.

### Driven Adapters

Driven adapters are those called by your core package, and should satisy the
interface defined there. The most common example, and the one I've used as an
example below is a database but they could also easily be a key-value store or
object storage. Beyond that, the same pattern can be used to send analytics
events or transactional emails.

```go
package postgres

type DB struct {
	db *sqlx.DB
}

func NewDB(/* connection info */) *DB { ... }

func (db *DB) NewUser(email, password string) (*User, error) { ... }
```

I would keep all of the code relating to a single external service in the same
package, so even if you have multiple core packages interacting with one. If
it's a single db, all of the database code can go in one place[^4].
[^4]: At least until you start splitting your tables into different schemas.

### Driving Adapters

A driving adapter is one that calls the functions in your core, and makes them
available externally. For this example I'm going to use a simple _REST_[^2] API
for our user service, designed for some SPA frontend, as this is what I have
most commonly encountered.
[^2]: This isn't actually REST, but is often referred to as REST, see
	[_How Did REST Come To Mean The Opposite of REST?_](https://htmx.org/essays/how-did-rest-come-to-mean-the-opposite-of-rest/)

```go
package webapi

type Handlers struct {
	users *users.Manager
}

func NewHandlers(users *users.Manager) *Handlers {
	return &Handlers{users: users}
}

func (h *Handlers) Route(mux *http.ServeMux) {
	mux.HandleFunc("POST /register", h.handlePostRegister)
}

func (h *Handlers) handlePostRegister(w http.ResponseWriter, r *http.Request) {
	var req postRegisterRequest
	err := json.NewDecoder(r.Body).Decode(&req)
	if err != nil {
		/* handle error */
	}

	user, err := h.users.New(req.Email, req.Password)
	if err != nil {
		/* handle error */
	}

	resp := postRegisterResponse{
		UserID: user.ID,
	}
	err = json.NewEncoder(w).Encode(&resp)
	if err != nil {
		/* handle error */
	}

	w.WriteHeader(http.StatusCreated)
}
```

When it comes to factoring this package, I like to divide it by intended
consumer, so routes for your SPA frontend go in one place, and webhook handlers
go in another etc. That way the request and reply formats and expectations of
all routes in one package is the same, as is the authentication.

## Utilities

Utility packages are those that don't fit into either previous category, by both
not implementing business logic and not interacting direcltly with an external
service. Aside from any convenience abstractions that you want to reuse in
multiple packages, the main thing I expect to fit this category in our web app
is authorization. Auth _kind of_ fits in with the driving adapter, but also
splits of nicely, and if you're exposing multiple APIs you're probably going to
reuse some of the code.

Also, don't take this category as a suggestion to create a `utils` package.
Generically named packages like this are recommended against in go, and _A
little copying is better than a little dependancy_[^3].
[^3]: From [Go Proverbs with Rob Pike](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=568s)
	at Gopherfest 2015

## Conclusion and other thoughts

When it comes to the folder structure, I would advise keeping everything as flat
as possible, Subfolders are nice where you have a bunch of non-go files related
to a package, such as your database migrations. You probably don't need a `src`,
`pkg`, or `internal` folder to contain other packages. You can just leave
everything in the top level of your repo. These make sense in very large
projects like [Moby](https://github.com/moby/moby) or
[Kubernetes](https://github.com/kubernetes/kubernetes), but for our web service
you're just making things harder to find. Leave it at the top level until that
starts to cause issues.

</br>

As I said at the top, this is the layout that I've arrived at from my experience
working on commercial web backends in Go. I think it's a practical level of
abstraction to stay flexible without spending too much time or adding too much
complexity.

You might think that there isn't actually much to what I have
suggested here, but that's part of the point. You don't need much abstraction
and not having it will help reduce the mental load of working on your codebase,
allowing you to focus on productive work and for new developers to be onboarded
more easily.

### Further Reading

 * [Hexagonal architecture](https://alistair.cockburn.us/hexagonal-architecture/) - Alistair Cockburn
 * [Package names](https://go.dev/blog/package-names) -  Sameer Ajmani
 * [Less is exponentially more](https://commandcenter.blogspot.com/2012/06/less-is-exponentially-more.html) - Rob Pike
