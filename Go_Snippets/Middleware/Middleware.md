# Middleware & Routes Explanation

---

### Middleware

```go
// middleware/auth.go
package middleware

import (
    "net/http"
)

func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")

        if token == "" {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }

        // TODO: validate your JWT token here

        next.ServeHTTP(w, r) // ✅ pass to next handler if authenticated
    })
}
```

**Line by line:**

- `func AuthMiddleware(next http.Handler) http.Handler` — This is a function that wraps around your route handlers. The `next` parameter is basically saying _"whatever comes after me, I'll decide if it gets to run or not."_

- `return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request)` — It returns a new handler. `w` is where you **write responses** back to the client, `r` is the **incoming request** from the client.

- `token := r.Header.Get("Authorization")` — It grabs the `Authorization` header from the request. This is where your frontend sends the token, usually like `Bearer eyJhbGci...`

- `if token == ""` — Checks if the token is missing or empty.

- `http.Error(w, "Unauthorized", http.StatusUnauthorized)` — If there's **no token**, send back a `401 Unauthorized` response to the client.

- `return` — Stops everything here. Nothing else runs after this if the token is missing.

- `next.ServeHTTP(w, r)` — If the token exists, **let the request through** to the actual route handler. Think of it like a bouncer saying _"okay you're good, go in."_

---

### Routes

```go
// routes/user.go
package routes

import (
    "github.com/go-chi/chi/v5"
    "sanctuary_server/middleware"
    "sanctuary_server/handlers"
)

func UserRoutes(r *chi.Mux) {
    // Public routes — no auth needed
    r.Post("/login", handlers.Login)
    r.Post("/register", handlers.Register)

    // Protected routes — auth required
    r.Group(func(r chi.Router) {
        r.Use(middleware.AuthMiddleware)
        r.Get("/users", handlers.GetUsers)
        r.Delete("/users/{id}", handlers.DeleteUser)
    })
}
```

**Line by line:**

- `r.Post("/login", handlers.Login)` — Public route. Anyone can hit this, no token needed. Makes sense because the user isn't logged in yet.

- `r.Post("/register", handlers.Register)` — Same as login, public. No auth required to create an account.

- `r.Group(func(r chi.Router)` — Creates a **sub-group** of routes. Anything inside this group shares the same middleware. Like a VIP section in a club.

- `r.Use(middleware.AuthMiddleware)` — Applies the auth check to **every route inside this group only**. Routes outside the group are not affected.

- `r.Get("/users", handlers.GetUsers)` — Protected route. Before reaching `GetUsers`, the request must pass through `AuthMiddleware` first.

- `r.Delete("/users/{id}", handlers.DeleteUser)` — Also protected. The `{id}` is a URL parameter, like `/users/123`.

---

### Full Request Flow

```
Client Request
     ↓
Is it /login or /register?  →  Go straight to handler ✅
     ↓
Is it /users or /users/{id}?
     ↓
AuthMiddleware → No token? → 401 Unauthorized ❌
     ↓
Has token? → Go to handler ✅
```
