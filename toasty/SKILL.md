---
name: toasty-guidance
description: Guidance for using the Toasty async ORM crate — schema definition, CRUD, relations, queries, and transactions
crates: toasty
activation: always
---

Toasty is an async ORM for Rust supporting SQL (SQLite, PostgreSQL, MySQL) and NoSQL (DynamoDB). It prioritizes type safety and leans into each database's capabilities rather than hiding them.

# Schema definition

Define models with `#[derive(toasty::Model)]`. Mark the primary key with `#[key]` and auto-generated fields with `#[auto]` (auto-increment for integers, UUID v7 for `uuid::Uuid`).

```rust
#[derive(Debug, toasty::Model)]
struct User {
    #[key]
    #[auto]
    id: i64,

    #[unique]
    email: String,

    name: String,

    #[has_many]
    todos: toasty::HasMany<Todo>,
}

#[derive(Debug, toasty::Model)]
struct Todo {
    #[key]
    #[auto]
    id: i64,

    #[index]
    user_id: i64,

    #[belongs_to(key = user_id, references = id)]
    user: toasty::BelongsTo<User>,

    title: String,
}
```

Use `#[derive(toasty::Embed)]` for value types that flatten into the parent table:

```rust
#[derive(toasty::Embed)]
struct Address {
    street: String,
    city: String,
}
```

# Database setup

Register all models with the builder and connect:

```rust
let db = toasty::Db::builder()
    .register::<User>()
    .register::<Todo>()
    .connect("sqlite://memory")
    .await?;
```

Connection strings: `sqlite://memory`, `sqlite:///path/to/db`, `postgresql://user:pass@host/db`, `mysql://user:pass@host/db`, `dynamodb://region`.

# CRUD operations

## Create

```rust
let user = User::create()
    .name("Alice")
    .email("alice@example.com")
    .todo(Todo::create().title("Task 1"))  // nested create
    .exec(&mut db)
    .await?;
```

## Read

```rust
// By primary key (generated finder)
let user = User::get_by_id(&mut db, &id).await?;

// All records
let users = User::all().exec(&mut db).await?;

// Filter
let users = User::all()
    .and(User::fields().name().eq("Alice"))
    .exec(&mut db)
    .await?;

// Single result
let user = User::all()
    .and(User::fields().email().eq("a@b.com"))
    .one()
    .exec(&mut db)
    .await?;

// Optional result
let user = User::all()
    .and(User::fields().email().eq("a@b.com"))
    .first()
    .exec(&mut db)
    .await?;

// Count
let n = User::all().count().exec(&mut db).await?;
```

## Update

```rust
user.update()
    .name("Bob")
    .exec(&mut db)
    .await?;
```

## Delete

```rust
User::all()
    .and(User::fields().active().eq(false))
    .delete()
    .exec(&mut db)
    .await?;
```

# Query expressions

Build type-safe filters using generated field paths:

```rust
User::fields().age().gt(18)
User::fields().age().ge(21)
User::fields().name().ne("admin")
User::fields().id().is_in([1, 2, 3])
User::fields().phone().is_some()
User::fields().phone().is_none()

// Combine with boolean logic
User::fields().age().gt(18).and(User::fields().active().eq(true))
```

The `query!` macro offers a shorthand:

```rust
toasty::query!(User filter .name == "Alice" && .age > 18)
```

# Relations

Three relation types: `HasMany<T>`, `HasOne<T>`, `BelongsTo<T>`.

**Lazy loading** (default) -- access a relation to load it on demand:

```rust
let todos = user.todos().exec(&mut db).await?;
let owner = todo.user().exec(&mut db).await?;
```

**Eager loading** -- preload with `.include()`:

```rust
let users = User::all()
    .include(User::fields().todos())
    .exec(&mut db)
    .await?;
```

# Pagination

Cursor-based pagination:

```rust
let page = User::all().paginate(10).exec(&mut db).await?;
for u in &page.items { /* ... */ }
if page.has_next() {
    let next = page.next(&mut db).await?;
}
```

# Transactions

```rust
let mut tx = db.transaction().await?;
User::create().name("Alice").exec(&mut tx as &mut dyn toasty::Executor).await?;
tx.commit().await?;
// Drop without commit = automatic rollback
```

Nested transactions (savepoints) are supported via `tx.transaction().await?`.

# Batching

Execute independent statements together:

```rust
use toasty::batch;

let (users, posts) = batch((
    User::all(),
    Post::all(),
)).exec(&mut db).await?;
```

# Key design points

- The `Executor` trait is implemented by both `Db` and `Transaction` -- pass `&mut db` or `&mut tx` interchangeably.
- `Db` owns a connection pool (deadpool) and is cheap to clone.
- For a dedicated connection, use `db.connection().await?`.
- Relations are **unloaded by default**; calling `.get()` before loading panics. Use `.include()` for eager loading or call the relation method to load lazily.
- Statement type parameter `T` is the **returning type**, not the model (e.g., `Query<List<User>>` returns `Vec<User>`).

See the `LLM.txt` resource within this skill for the full API reference.
