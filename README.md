# ToDo Application API

A REST API for managing personal to-do items. The application is built with FastAPI and SQLAlchemy, stores users and todos in PostgreSQL, and protects user-specific operations with JWT bearer tokens. Users with the `admin` role can also view and delete todos across the application.

## Features

- User registration and password hashing
- Login with OAuth2 password form credentials and JWT access tokens
- Create, list, retrieve, update, and delete a user's todos
- Admin-only endpoints for viewing and deleting todos
- Interactive API documentation provided by FastAPI

## Requirements

- Python
- PostgreSQL

Python packages are listed in `requirements.txt`.

## Run locally

1. Clone the repository and enter the project directory.
2. Create and activate a virtual environment:

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   On macOS or Linux, activate it with:

   ```sh
   source .venv/bin/activate
   ```

3. Install the dependencies:

   ```sh
   pip install -r requirements.txt
   ```

4. Configure the PostgreSQL connection in `database.py` before starting the application. The current code defines the connection URL in that file. Use your own database credentials and never commit credentials to source control.
5. Start the development server:

   ```sh
   uvicorn main:app --reload
   ```

6. Open the interactive API documentation at <http://127.0.0.1:8000/docs>. The alternative ReDoc page is at <http://127.0.0.1:8000/redoc>.

On startup, the application creates the database tables declared in `models.py` if they do not already exist.

## Live site:

1. https://todo-application-fastapi.onrender.com
2. https://todo-application-fastapi.onrender.com/docs

## Authentication

Register a user with `POST /createuser`, then submit the user's username and password as form data to `POST /login`. The login endpoint returns an access token. For protected endpoints, send it in the request header:

```http
Authorization: Bearer <access_token>
```

Tokens expire after 30 minutes. The role supplied when registering determines whether the user can access admin routes; use the `admin` role only for trusted administrators.

## API endpoints

| Method   | Path                      | Authentication     | Description                            |
| -------- | ------------------------- | ------------------ | -------------------------------------- |
| `POST`   | `/createuser`             | None               | Register a user                        |
| `POST`   | `/login`                  | None               | Log in and receive an access token     |
| `GET`    | `/`                       | Bearer token       | List the current user's todos          |
| `GET`    | `/todo/{todo_id}`         | Bearer token       | Get one of the current user's todos    |
| `POST`   | `/create/`                | Bearer token       | Create a todo for the current user     |
| `PUT`    | `/edit/{todo_id}`         | Bearer token       | Update one of the current user's todos |
| `DELETE` | `/delete/{todo_id}`       | Bearer token       | Delete one of the current user's todos |
| `GET`    | `/user`                   | Bearer token       | Get the current user's record          |
| `PUT`    | `/edituser`               | Bearer token       | Update the current user's profile      |
| `PUT`    | `/passwordchange`         | Bearer token       | Change the current user's password     |
| `GET`    | `/admin/todo`             | Admin bearer token | List all todos                         |
| `DELETE` | `/admin/delete/{todo_id}` | Admin bearer token | Delete any todo                        |

### Todo request body

`POST /create/` requires all fields below. The caller supplies the `id`.

```json
{
  "id": 1,
  "title": "Buy groceries",
  "description": "Milk and bread",
  "priority": 3,
  "completed": false
}
```

`description` is limited to 100 characters and `priority` must be between 1 and 5. For `PUT /edit/{todo_id}`, send only the fields to change; it accepts `title`, `description`, `priority`, and `completed`.

## Run tests

```sh
pytest
```

The tests use the configured database, so ensure it is reachable and has the expected test data/setup before running them.

## Deployment and secrets

For a Render Web Service, use `pip install -r requirements.txt` as the build command and `uvicorn main:app --host 0.0.0.0 --port $PORT` as the start command. Configure the application to connect to the service's PostgreSQL database before deploying.

Before deploying, move database credentials and the JWT signing key out of source code into environment-based configuration, and rotate any credentials or signing keys that have been exposed. Do not publish secrets in the README, repository, or deployment logs.
