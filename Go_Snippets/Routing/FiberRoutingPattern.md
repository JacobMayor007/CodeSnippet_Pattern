# Fiber Framework - Routes Explanation

---

### Main

This is the entry point of your server. `fiber.App()` creates a new Fiber application — it's the **core of your server** that handles all incoming requests. Then `routes.MainRoutes(app)` passes that app to your routes file so all routes get registered on it.

```go
package main

func main() {
    app := fiber.App()

    routes.MainRoutes(app)
}
```

---

### Routes

This is the **central hub** for all your routes. `SetupRoutes` receives the Fiber app and your repositories (`UserRepository` and `ChatRepository`) as dependencies — this is the **dependency injection** part. Its only job is to call every route group and pass the right repository to each one. When you add a new feature later, like `BookingRoutes`, you just add it here.

```go
package routes

import (
    "websocket_server/repository"

    "github.com/gofiber/fiber/v2"
)

func SetupRoutes(app *fiber.App, user repository.UserRepository, chat repository.ChatRepository) {
    UserRoutes(app, user)
    ChatRoutes(app, chat)
}
```

---

### User

This is where the actual **User routes** are defined. It receives the app and the `UserRepository` as a dependency. Then it creates a `UserRepo` struct from the `api` package and assigns the repository to it — this is how the handler gets access to the database logic. Finally, `app.Post("/user", userApi.CreateUser)` registers a `POST` request on `/user`, so when a client hits `POST /user`, it runs `CreateUser` to handle the request.

```go
import (
    "websocket_server/api"
    "websocket_server/repository"

    "github.com/gofiber/fiber/v2"
)

func UserRoutes(app *fiber.App, repo repository.UserRepository) {
    userApi := api.UserRepo{
        UserRepository: repo,
    }

    app.Post("/user", userApi.CreateUser)
}
```

---

### Full Flow

```
Client hits POST /user
        ↓
main.go → fiber.App() creates the server
        ↓
routes.SetupRoutes(app, user, chat) → registers all route groups
        ↓
UserRoutes(app, user) → registers POST /user
        ↓
userApi.CreateUser → handles the request and responds
```
