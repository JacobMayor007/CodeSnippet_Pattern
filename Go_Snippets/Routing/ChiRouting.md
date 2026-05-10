# Chi Framework - Routes Explanation

---

### Main

This is the entry point of your server. `chi.NewRouter()` creates a brand new router — think of it as the **main traffic controller** that decides where each request goes. Then `routes.MainRoutes(c)` hands that router over to your routes file so all the routes can be registered on it.

```go
c := chi.NewRouter()

routes.MainRoutes(c)
```

---

### Routes

This is the **central hub** for all your routes. `MainRoutes` receives the router `c` and its only job is to call every route group you have. So when you add a new feature later, like `SanctuaryRoutes` or `BookingRoutes`, you just add them here. Think of it like a table of contents for your API.

```go
func MainRoutes(c *chi.Mux) {
    UserRoutes(c)
}
```

---

### User

This is where the actual **User routes** are defined. The router `c` registers a `GET` request on the path `/hello`. When a client hits `GET /hello`, it will run the `repository.GetUser` function which handles the logic and sends back the response.

```go
func UserRoutes(c *chi.Mux) {
    c.Get("/hello", repository.GetUser)
}
```

---

### Full Flow

```
Client hits GET /hello
        ↓
main.go → chi.NewRouter() creates the router
        ↓
routes.MainRoutes(c) → registers all route groups
        ↓
UserRoutes(c) → registers /hello
        ↓
repository.GetUser → handles the request and responds
```
