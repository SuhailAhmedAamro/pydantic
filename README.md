## What is Pydantic?

Pydantic is a Python library for **data validation and settings management** using type hints. It plays a key role in **FastAPI** by validating request and response data, automatically converting types, and ensuring data integrity. This is especially important for building robust agent-based systems like DACA, where correct data structure is crucial.

---

## Key Features of Pydantic

* ✅ **Type Validation**: Uses Python type hints to validate input/output.
* 🔁 **Automatic Type Conversion**: Converts data types intelligently (e.g., string to integer).
* ❌ **Detailed Error Messages**: Provides clear feedback when data doesn’t match the schema.
* 🧩 **Nested Models**: Allows structured and reusable data models.
* 🔄 **Serialization Support**: Easily converts to/from JSON or other formats.
* 🆗 **Defaults & Optional Fields**: Simplifies handling of missing or optional fields.
* 🛡️ **Custom Validators**: Add your own validation logic when needed.

---

## Step 1: Introduction to Pydantic with Examples

Start a new FastAPI project:

```bash
uv init fastdca_p1
cd fastdca_p1
uv venv
source .venv/bin/activate
uv add "fastapi[standard]"
```

### Example 1: Basic Pydantic Model

File: `pydantic_example_1.py`

```python
from pydantic import BaseModel, ValidationError

class User(BaseModel):
    id: int
    name: str
    email: str
    age: int | None = None

user_data = {"id": 1, "name": "Alice", "email": "alice@example.com", "age": 25}
user = User(**user_data)
print(user)
print(user.model_dump())

try:
    invalid_user = User(id="abc", name="Bob", email="bob@example.com")
except ValidationError as e:
    print(e)
```

Command:

```bash
uv run python pydantic_example_1.py
```

Expected error output:

```
1 validation error for User
id
  value is not a valid integer (type=type_error.integer)
```

---

### Example 2: Nested Pydantic Models

File: `pydantic_example_2.py`

```python
from pydantic import BaseModel, EmailStr

class Address(BaseModel):
    street: str
    city: str
    zip_code: str

class UserWithAddress(BaseModel):
    id: int
    name: str
    email: EmailStr
    addresses: list[Address]

user_data = {
    "id": 2,
    "name": "Bob",
    "email": "bob@example.com",
    "addresses": [
        {"street": "123 Main St", "city": "NYC", "zip_code": "10001"},
        {"street": "456 Oak Ave", "city": "LA", "zip_code": "90001"},
    ],
}
user = UserWithAddress.model_validate(user_data)
print(user.model_dump())
```

---

### Example 3: Custom Validator

File: `pydantic_example_3.py`

```python
from pydantic import BaseModel, EmailStr, validator, ValidationError
from typing import List

class Address(BaseModel):
    street: str
    city: str
    zip_code: str

class UserWithAddress(BaseModel):
    id: int
    name: str
    email: EmailStr
    addresses: List[Address]

    @validator("name")
    def check_name_length(cls, value):
        if len(value) < 2:
            raise ValueError("Name must be at least 2 characters long")
        return value

try:
    user = UserWithAddress(
        id=3,
        name="A",
        email="charlie@example.com",
        addresses=[{"street": "789 Pine Rd", "city": "Chicago", "zip_code": "60601"}],
    )
except ValidationError as e:
    print(e)
```

---

## Why Use Pydantic in DACA?

Pydantic fits perfectly in agentic AI systems like DACA because:

* It **validates incoming data** from users and agents.
* It supports **complex structures** (e.g., nested metadata).
* It **serializes responses** into JSON effortlessly.
* It improves **error clarity** and debugging.

---

## Step 2: Build a FastAPI App with Pydantic Models

File: `main.py`

```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel, Field
from datetime import datetime, UTC
from uuid import uuid4

app = FastAPI(
    title="DACA Chatbot API",
    description="A chatbot API using FastAPI and Pydantic",
    version="0.1.0",
)

# Define Metadata model
class Metadata(BaseModel):
    timestamp: datetime = Field(default_factory=lambda: datetime.now(tz=UTC))
    session_id: str = Field(default_factory=lambda: str(uuid4()))

# Define user message model
class Message(BaseModel):
    user_id: str
    text: str
    metadata: Metadata
    tags: list[str] | None = None

# Define chatbot response model
class Response(BaseModel):
    user_id: str
    reply: str
    metadata: Metadata

@app.get("/")
async def home():
    return {"message": "Welcome to the DACA Chatbot API. Visit /docs to explore."}

@app.get("/users/{user_id}")
async def get_user(user_id: str, role: str | None = None):
    return {"user_id": user_id, "role": role or "guest"}

@app.post("/chat/", response_model=Response)
async def chat(message: Message):
    if not message.text.strip():
        raise HTTPException(status_code=400, detail="Message text cannot be empty.")
    
    reply = f"Hello, {message.user_id}! You said: '{message.text}'."
    return Response(user_id=message.user_id, reply=reply, metadata=Metadata())
```

---

## Run the Application

```bash
fastapi dev main.py

