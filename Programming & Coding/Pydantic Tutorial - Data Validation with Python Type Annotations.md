# Pydantic Tutorial: Data Validation with Python Type Annotations

## Table of Contents
1. [Introduction to Pydantic](#introduction-to-pydantic)
2. [Installation](#installation)
3. [Basic Models](#basic-models)
4. [Field Types and Validation](#field-types-and-validation)
5. [Field Customization](#field-customization)
6. [Model Configuration](#model-configuration)
7. [Validators and Custom Validation](#validators-and-custom-validation)
8. [Model Methods and Properties](#model-methods-and-properties)
9. [Schema Generation](#schema-generation)
10. [Working with JSON](#working-with-json)
11. [Nested Models and Complex Structures](#nested-models-and-complex-structures)
12. [Model Inheritance](#model-inheritance)
13. [Generic Models](#generic-models)
14. [Pydantic v1 vs v2](#pydantic-v1-vs-v2)
15. [Best Practices](#best-practices)
16. [Common Patterns and Use Cases](#common-patterns-and-use-cases)

## Introduction to Pydantic

Pydantic is a powerful data validation and parsing library for Python that uses type annotations to define data models. It enforces type hints at runtime and provides user-friendly errors when data is invalid.

### Key Features:

- **Type Validation**: Automatically validates data against Python type hints
- **Data Conversion**: Converts input data to the declared types when possible
- **Schema Generation**: Generates JSON Schema from models
- **Integration**: Works with popular frameworks like FastAPI
- **Performance**: Offers high performance with Rust-based validation (in v2)
- **IDE Support**: Excellent IDE completion and type checking

## Installation

### Standard Installation

```bash
pip install pydantic
```

### With Extra Features

```bash
# For email validation
pip install "pydantic[email]"

# For all extras
pip install "pydantic[all]"
```

### Pydantic V2 (latest)

```bash
pip install "pydantic>=2.0.0"
```

## Basic Models

Pydantic's core concept is the model - a class that inherits from `BaseModel` and uses type annotations to validate data.

### Defining a Basic Model

```python
from pydantic import BaseModel
from typing import Optional

class User(BaseModel):
    id: int
    name: str
    email: str
    age: Optional[int] = None
    is_active: bool = True

# Create a user with valid data
user = User(id=1, name="John Doe", email="john@example.com")
print(user)
# User(id=1, name='John Doe', email='john@example.com', age=None, is_active=True)

# Automatic type conversion
user_from_dict = User(id="2", name="Jane Smith", email="jane@example.com", age="30")
print(user_from_dict)
# User(id=2, name='Jane Smith', email='jane@example.com', age=30, is_active=True)
```

### Validation Errors

```python
try:
    # This will raise a validation error
    invalid_user = User(id="abc", name=123, email="not-an-email")
except Exception as e:
    print(str(e))
    # 3 validation errors for User
    # id
    #   Input should be a valid integer [type=int_parsing, input_value='abc', input_type=str]
    # name
    #   Input should be a valid string [type=string_type, input_value=123, input_type=int]
    # email
    #   Input should be a valid string [type=string_type, input_value=NotAnEmail(), input_type=NotAnEmail]
```

## Field Types and Validation

Pydantic supports a wide range of field types, from Python's built-in types to more complex validation types.

### Basic Types

```python
from pydantic import BaseModel
from typing import List, Dict, Set, Tuple, Union
from datetime import datetime, date

class Example(BaseModel):
    # Basic types
    string_field: str
    int_field: int
    float_field: float
    bool_field: bool
    
    # Container types
    list_field: List[int]
    dict_field: Dict[str, float]
    set_field: Set[str]
    tuple_field: Tuple[int, str, float]
    
    # Union types (either int or str)
    union_field: Union[int, str]
    
    # Date and time types
    datetime_field: datetime
    date_field: date

# Example usage
example = Example(
    string_field="hello",
    int_field=123,
    float_field=3.14,
    bool_field=True,
    list_field=[1, 2, 3],
    dict_field={"a": 1.0, "b": 2.0},
    set_field={"a", "b", "c"},
    tuple_field=(1, "test", 2.5),
    union_field="either string or number",
    datetime_field="2023-01-01T12:00:00",  # ISO format string is automatically parsed
    date_field="2023-01-01"
)
```

### Special Types

Pydantic provides specialized types with built-in validation:

```python
from pydantic import BaseModel, EmailStr, HttpUrl, FilePath, DirectoryPath, constr, conint

class AdvancedTypes(BaseModel):
    # Email validation
    email: EmailStr
    
    # URL validation
    website: HttpUrl
    
    # Path validation
    config_file: FilePath
    data_directory: DirectoryPath
    
    # Constrained types
    username: constr(min_length=3, max_length=50, pattern=r'^[a-zA-Z0-9_]+$')
    age: conint(ge=0, lt=120)  # Greater than or equal to 0, less than 120

# Note: Requires extras
# pip install "pydantic[email]"
```

## Field Customization

### Using Field for Advanced Customization

```python
from pydantic import BaseModel, Field
from typing import List

class Product(BaseModel):
    id: int
    name: str = Field(..., min_length=1, max_length=100)
    description: str = Field(default="", title="Product description", max_length=1000)
    price: float = Field(..., gt=0, description="Price must be greater than zero")
    tags: List[str] = Field(default_factory=list)
    in_stock: bool = Field(default=True)
    
    class Config:
        schema_extra = {
            "example": {
                "id": 123,
                "name": "Laptop",
                "description": "A high-performance laptop",
                "price": 999.99,
                "tags": ["electronics", "computers"],
                "in_stock": True
            }
        }

# The ellipsis (...) means the field is required
laptop = Product(id=1, name="Laptop", price=999.99)
```

### Field Options

- `default`: Default value if not provided
- `default_factory`: Function to generate default value
- `alias`: Alternative name for the field in source data
- `title`, `description`: Used for schema generation
- `gt`, `ge`, `lt`, `le`: Greater than, greater than or equal, less than, less than or equal
- `min_length`, `max_length`: String and list length validation
- `regex`: Pattern validation for strings

## Model Configuration

Pydantic models can be configured with various options using the inner `Config` class:

```python
from pydantic import BaseModel
from datetime import datetime

class UserProfile(BaseModel):
    user_id: int
    full_name: str
    created_at: datetime
    
    class Config:
        # Allow extra fields in input data without validation errors
        extra = "ignore"  # Options: "ignore", "allow", "forbid"
        
        # Validate field assignment after model creation
        validate_assignment = True
        
        # Convert snake_case field names to camelCase
        alias_generator = lambda string: ''.join(
            word.capitalize() if i > 0 else word
            for i, word in enumerate(string.split('_'))
        )
        
        # Enable this to respect field aliases in JSON output
        populate_by_name = True
        
        # Set case-sensitive field names
        case_sensitive = True
        
        # Frozen model (immutable)
        frozen = False

# With alias_generator, user_id becomes userId in JSON schema
```

## Validators and Custom Validation

### Method Validators

```python
from pydantic import BaseModel, validator, root_validator
from typing import List

class Order(BaseModel):
    id: str
    items: List[str]
    quantities: List[int]
    total_price: float
    
    # Validate a single field
    @validator('id')
    def id_must_be_valid_format(cls, v):
        if not v.startswith('ORD'):
            raise ValueError('Order ID must start with ORD')
        return v.upper()
    
    # Validate multiple fields
    @validator('quantities')
    def quantities_must_be_positive(cls, quantities):
        if any(q <= 0 for q in quantities):
            raise ValueError('All quantities must be positive')
        return quantities
    
    # Validate dependencies between fields
    @validator('items', 'quantities')
    def lists_must_have_same_length(cls, v, values):
        if 'items' in values and 'quantities' in values:
            if len(values['items']) != len(v):
                raise ValueError('Items and quantities must have the same length')
        return v
    
    # Validate the entire model (root_validator)
    @root_validator
    def check_total_price(cls, values):
        # In a real app, you'd calculate this from items and their prices
        if values.get('total_price', 0) <= 0:
            raise ValueError('Total price must be positive')
        return values

# Create a valid order
order = Order(
    id="ord123", 
    items=["laptop", "mouse"], 
    quantities=[1, 2], 
    total_price=1200.00
)
```

### Field Validators (Pydantic v2)

```python
from pydantic import BaseModel, field_validator, model_validator
from typing import List

class Order(BaseModel):
    id: str
    items: List[str]
    quantities: List[int]
    total_price: float
    
    # Field validator (v2 style)
    @field_validator('id')
    @classmethod
    def id_must_be_valid_format(cls, v):
        if not v.startswith('ORD'):
            raise ValueError('Order ID must start with ORD')
        return v.upper()
    
    # Model validator (v2 style)
    @model_validator(mode='after')
    def check_total_price(self):
        if len(self.items) != len(self.quantities):
            raise ValueError('Items and quantities must have the same length')
        if self.total_price <= 0:
            raise ValueError('Total price must be positive')
        return self
```

## Model Methods and Properties

Pydantic models come with useful built-in methods and you can add your own:

```python
from pydantic import BaseModel
from typing import List, Optional

class ShoppingCart(BaseModel):
    user_id: int
    items: List[dict]
    coupon_code: Optional[str] = None
    
    # Custom property
    @property
    def item_count(self) -> int:
        return len(self.items)
    
    # Custom method
    def total_price(self) -> float:
        return sum(item.get('price', 0) * item.get('quantity', 1) for item in self.items)
    
    # Apply discount based on coupon
    def apply_discount(self, discount_percentage: float) -> float:
        total = self.total_price()
        return total * (1 - discount_percentage / 100)

# Create a shopping cart
cart = ShoppingCart(
    user_id=123,
    items=[
        {"id": 1, "name": "Laptop", "price": 1000, "quantity": 1},
        {"id": 2, "name": "Mouse", "price": 25, "quantity": 2}
    ],
    coupon_code="SAVE10"
)

print(f"Item count: {cart.item_count}")  # 2
print(f"Total price: ${cart.total_price()}")  # $1050
print(f"Price after 10% discount: ${cart.apply_discount(10)}")  # $945
```

## Schema Generation

Pydantic can generate JSON Schema documentation for your models:

```python
from pydantic import BaseModel, Field
from typing import List, Optional
import json

class Address(BaseModel):
    street: str
    city: str
    country: str
    postal_code: str

class User(BaseModel):
    id: int
    name: str = Field(..., description="The user's full name")
    email: str = Field(..., description="The user's email address")
    addresses: List[Address] = Field(default_factory=list)
    age: Optional[int] = Field(None, description="The user's age in years")

# Get JSON Schema
schema = User.model_json_schema()
print(json.dumps(schema, indent=2))

# Output can be used for:
# - API documentation
# - Form generation
# - Data validation in frontend apps
```

## Working with JSON

Pydantic makes it easy to work with JSON data:

```python
from pydantic import BaseModel
import json
from typing import List, Dict, Any

class BlogPost(BaseModel):
    id: int
    title: str
    content: str
    tags: List[str] = []
    metadata: Dict[str, Any] = {}
    
# JSON string
json_data = '''
{
    "id": 1,
    "title": "Pydantic Tutorial",
    "content": "This is a tutorial about Pydantic",
    "tags": ["python", "pydantic", "tutorial"],
    "metadata": {
        "views": 1000,
        "published_date": "2023-01-01"
    }
}
'''

# Parse JSON string to model
post = BlogPost.model_validate_json(json_data)
print(post.title)  # Pydantic Tutorial

# Convert model to dict
post_dict = post.model_dump()
print(type(post_dict))  # <class 'dict'>

# Convert model to JSON string
post_json = post.model_dump_json()
print(type(post_json))  # <class 'str'>

# Include/exclude fields
print(post.model_dump(include={"title", "content"}))
print(post.model_dump(exclude={"metadata"}))

# Export with camelCase field names (Pydantic v2)
print(post.model_dump(by_alias=True))
```

## Nested Models and Complex Structures

Pydantic handles nested models naturally:

```python
from pydantic import BaseModel
from typing import List, Optional
from datetime import datetime

class Comment(BaseModel):
    id: int
    text: str
    created_at: datetime
    author_id: int

class Post(BaseModel):
    id: int
    title: str
    content: str
    published: bool = False
    comments: List[Comment] = []
    
class User(BaseModel):
    id: int
    username: str
    email: str
    bio: Optional[str] = None
    posts: List[Post] = []

# Create nested structure
user = User(
    id=1,
    username="john_doe",
    email="john@example.com",
    bio="Python developer",
    posts=[
        Post(
            id=1,
            title="Hello World",
            content="My first post",
            published=True,
            comments=[
                Comment(
                    id=1,
                    text="Great post!",
                    created_at="2023-01-02T14:30:00",
                    author_id=2
                )
            ]
        )
    ]
)

# Access nested attributes
print(user.posts[0].comments[0].text)  # Great post!
```

## Model Inheritance

Pydantic supports model inheritance for code reuse:

```python
from pydantic import BaseModel, EmailStr
from datetime import datetime
from typing import Optional, List

class BaseUser(BaseModel):
    id: int
    email: EmailStr
    created_at: datetime = datetime.now()
    
class BasicProfile(BaseUser):
    username: str
    bio: Optional[str] = None
    
class AdminUser(BaseUser):
    username: str
    permissions: List[str]
    admin_since: datetime
    
class CompleteProfile(BasicProfile):
    first_name: Optional[str] = None
    last_name: Optional[str] = None
    phone_number: Optional[str] = None
    profile_picture: Optional[str] = None

# Create objects using inheritance
basic_user = BasicProfile(
    id=1, 
    email="user@example.com",
    username="user123"
)

admin = AdminUser(
    id=2,
    email="admin@example.com",
    username="admin",
    permissions=["create", "read", "update", "delete"],
    admin_since="2023-01-01T00:00:00"
)
```

## Generic Models

Pydantic supports generics for reusable model patterns:

```python
from pydantic import BaseModel, Field
from typing import Generic, TypeVar, List, Optional

T = TypeVar('T')

class Pagination(BaseModel):
    total: int
    page: int
    page_size: int
    
class BaseResponse(BaseModel):
    success: bool
    message: Optional[str] = None

class Response(Generic[T], BaseResponse):
    data: Optional[T] = None
    pagination: Optional[Pagination] = None

# Define a specific model
class User(BaseModel):
    id: int
    name: str
    
# Use the generic response with User
user_response = Response[User](
    success=True,
    message="User fetched successfully",
    data=User(id=1, name="John Doe")
)

# Use the generic response with a list of users
users_response = Response[List[User]](
    success=True,
    message="Users fetched successfully",
    data=[
        User(id=1, name="John Doe"),
        User(id=2, name="Jane Smith")
    ],
    pagination=Pagination(total=100, page=1, page_size=10)
)

# Another use with a different model
class Product(BaseModel):
    id: int
    name: str
    price: float

products_response = Response[List[Product]](
    success=True,
    data=[
        Product(id=1, name="Laptop", price=999.99),
        Product(id=2, name="Mouse", price=24.99)
    ]
)
```

## Pydantic v1 vs v2

Pydantic v2 was a major rewrite in Rust with significant changes:

### Key Differences

| Feature | v1 | v2 |
|---------|----|----|
| Performance | Pure Python | Rust-powered core (much faster) |
| API | `dict()`, `json()` | `model_dump()`, `model_dump_json()` |
| Validators | `@validator`, `@root_validator` | `@field_validator`, `@model_validator` |
| Type Validation | Python-based | Rust-based with RustEnum types |
| Dataclasses | `pydantic.dataclasses` | Native Python with `pydantic.dataclasses.dataclass` |
| Default Schema | Draft 7 | 2020-12 |

### Migration Example

```python
# Pydantic v1
from pydantic import BaseModel, validator

class UserV1(BaseModel):
    name: str
    age: int
    
    @validator('age')
    def check_age(cls, v):
        if v < 0:
            raise ValueError('Age must be positive')
        return v
    
    # V1 methods
    def dict(self):
        return super().dict()
    
    def json(self):
        return super().json()

# Pydantic v2
from pydantic import BaseModel, field_validator

class UserV2(BaseModel):
    name: str
    age: int
    
    @field_validator('age')
    @classmethod
    def check_age(cls, v):
        if v < 0:
            raise ValueError('Age must be positive')
        return v
    
    # V2 methods
    def model_dump(self):
        return super().model_dump()
    
    def model_dump_json(self):
        return super().model_dump_json()
```

## Best Practices

### 1. Use Type Hints Properly

```python
# Good
from typing import Optional, List, Dict, Union

class User(BaseModel):
    id: int
    name: str
    tags: List[str] = []
    metadata: Dict[str, str] = {}
    age: Optional[int] = None
    value: Union[int, float, None] = None

# Bad
class User(BaseModel):
    id: int  # Good
    name = ""  # Missing type annotation
    tags = []  # Missing type annotation
```

### 2. Default Values

```python
from typing import List
from pydantic import BaseModel, Field

# Good
class Product(BaseModel):
    name: str
    description: str = ""  # Empty string default
    tags: List[str] = Field(default_factory=list)  # Use default_factory for mutable types
    
# Bad
class Product(BaseModel):
    name: str
    description: str = ""  # This is fine
    tags: List[str] = []  # Don't use mutable defaults without default_factory
```

### 3. Use Config for Common Settings

```python
from pydantic import BaseModel

# Create a base model with common configuration
class AppBaseModel(BaseModel):
    class Config:
        extra = "forbid"
        validate_assignment = True
        arbitrary_types_allowed = True
        use_enum_values = True

# Inherit from the base model
class MyModel(AppBaseModel):
    name: str
    value: int
```

### 4. Validation Error Handling

```python
from pydantic import BaseModel, ValidationError

class User(BaseModel):
    username: str
    email: str
    age: int

try:
    user = User(username="john", email="not-an-email", age="thirty")
except ValidationError as e:
    print("Validation errors:")
    for error in e.errors():
        print(f"Field: {error['loc'][0]}, Error: {error['msg']}")
        
    # You can also get them as JSON
    print(e.json())
```

## Common Patterns and Use Cases

### 1. API Request/Response Validation with FastAPI

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserCreate(BaseModel):
    username: str
    email: EmailStr
    password: str

class UserResponse(BaseModel):
    id: int
    username: str
    email: EmailStr

@app.post("/users/", response_model=UserResponse)
async def create_user(user: UserCreate):
    # UserCreate validation happens automatically
    # Your business logic here...
    return UserResponse(id=1, username=user.username, email=user.email)
```

### 2. Environment Settings Management

```python
from pydantic import BaseSettings, Field
from typing import Optional

class DatabaseSettings(BaseSettings):
    host: str = "localhost"
    port: int = 5432
    username: str
    password: str
    database: str
    
    class Config:
        env_prefix = "DB_"  # Will look for DB_HOST, DB_PORT, etc.
        case_sensitive = False

class AppSettings(BaseSettings):
    app_name: str = "My App"
    debug: bool = False
    database: DatabaseSettings
    api_key: Optional[str] = None
    
    class Config:
        env_nested_delimiter = "__"

# Usage
db_settings = DatabaseSettings()
app_settings = AppSettings(database=db_settings)
```

### 3. Data Transfer Objects (DTOs)

```python
from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

# Input DTO
class CreateArticleDTO(BaseModel):
    title: str
    content: str
    tags: List[str] = []
    published: bool = False

# Output DTO
class ArticleDTO(BaseModel):
    id: int
    title: str
    content: str
    tags: List[str]
    published: bool
    created_at: datetime
    updated_at: Optional[datetime] = None
    author_id: int

# Function using DTOs
def create_article(article_data: CreateArticleDTO, author_id: int) -> ArticleDTO:
    # Your database logic here
    article = {
        **article_data.model_dump(),
        "id": 1,
        "created_at": datetime.now(),
        "author_id": author_id
    }
    return ArticleDTO(**article)
```

### 4. Configuration from Files

```python
import yaml
from pydantic import BaseModel
from typing import Dict, List

class ServerConfig(BaseModel):
    host: str
    port: int
    debug: bool = False

class DatabaseConfig(BaseModel):
    url: str
    timeout: int = 30
    pool_size: int = 5

class LoggingConfig(BaseModel):
    level: str = "INFO"
    format: str = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
    handlers: List[str] = ["console"]

class AppConfig(BaseModel):
    app_name: str
    version: str
    server: ServerConfig
    database: DatabaseConfig
    logging: LoggingConfig
    features: Dict[str, bool] = {}

# Load config from YAML file
def load_config(config_file: str) -> AppConfig:
    with open(config_file, "r") as f:
        config_data = yaml.safe_load(f)
    return AppConfig(**config_data)

# Usage
try:
    config = load_config("config.yaml")
    print(f"Starting {config.app_name} v{config.version}")
    print(f"Server: {config.server.host}:{config.server.port}")
except Exception as e:
    print(f"Error loading configuration: {e}")
```