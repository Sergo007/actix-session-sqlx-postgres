Use Postgres via Sqlx as session storage backend.

```rust
use actix_web::{web, App, HttpServer, HttpResponse, Error};
use actix_session_sqlx_postgres::SqlxPostgresqlSessionStore;
use actix_session::SessionMiddleware;
use actix_web::cookie::Key;
// The secret key would usually be read from a configuration file/environment variables.
fn get_secret_key() -> Key {
    # todo!()
    // [...]
}
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let secret_key = get_secret_key();
    let psql_connection_string = "postgres://<username>:<password>@127.0.0.1:5432/<yourdatabase>";
    let store = SqlxPostgresqlSessionStore::new(psql_connection_string).await.unwrap();
    HttpServer::new(move ||
            App::new()
            .wrap(SessionMiddleware::new(
                store.clone(),
                secret_key.clone()
            ))
            .default_service(web::to(|| HttpResponse::Ok())))
        .bind(("127.0.0.1", 8080))?
        .run()
        .await
}
```

If you already have a connection pool, you can use something like

```rust
use actix_web::{web, App, HttpServer, HttpResponse, Error};
use actix_session_sqlx_postgres::SqlxPostgresqlSessionStore;
use actix_session::SessionMiddleware;
use actix_web::cookie::Key;
// The secret key would usually be read from a configuration file/environment variables.
fn get_secret_key() -> Key {
    # todo!()
    // [...]
}
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    use sqlx::postgres::PgPoolOptions;
let secret_key = get_secret_key();
    let pool = PgPoolOptions::find_some_way_to_build_your_pool(psql_connection_string);
    let store = SqlxPostgresqlSessionStore::from_pool(pool).await.expect("Could not build session store");
    HttpServer::new(move ||
            App::new()
            .wrap(SessionMiddleware::new(
                store.clone(),
                secret_key.clone()
            ))
            .default_service(web::to(|| HttpResponse::Ok())))
        .bind(("127.0.0.1", 8080))?
        .run()
        .await
}
```

