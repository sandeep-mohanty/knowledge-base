# Railway Oriented Programming in Python: A Complete Tutorial

## Introduction

Railway Oriented Programming (ROP) is a functional programming pattern for handling errors in a composable way. It models operations as railway tracks: success cases continue on the main track, while failures divert to a parallel error track. This tutorial shows you how to implement ROP in Python using a custom Result type.

## Table of Contents

1. [Understanding the Concept](#understanding-the-concept)
2. [Creating the Result Type](#creating-the-result-type)
3. [Basic Operations](#basic-operations)
4. [Chaining Operations](#chaining-operations)
5. [Practical Examples](#practical-examples)
6. [Advanced Patterns](#advanced-patterns)
7. [Best Practices](#best-practices)

## Understanding the Concept

In traditional Python error handling, we often see nested try-except blocks, multiple if-else statements, or exceptions. ROP provides a cleaner alternative by encapsulating success and failure in a single type that can be composed.

## Creating the Result Type

Let's start by creating our core Result type using Python's type hints and classes:

### Basic Result Type

```python
from typing import TypeVar, Generic, Callable, Optional, Any, Union

T = TypeVar('T')  # Success value type
E = TypeVar('E')  # Error type
U = TypeVar('U')  # Another type for transformations

class Result(Generic[T, E]):
    """Base class for Railway Oriented Programming pattern."""
    
    @staticmethod
    def success(value: T) -> 'Success[T, E]':
        """Create a Success result."""
        return Success(value)
    
    @staticmethod
    def failure(error: E) -> 'Failure[T, E]':
        """Create a Failure result."""
        return Failure(error)
    
    def is_success(self) -> bool:
        """Check if result is Success."""
        return isinstance(self, Success)
    
    def is_failure(self) -> bool:
        """Check if result is Failure."""
        return isinstance(self, Failure)

class Success(Result[T, E]):
    """Represents a successful operation."""
    
    def __init__(self, value: T):
        self.value = value
    
    def __repr__(self) -> str:
        return f"Success({self.value})"

class Failure(Result[T, E]):
    """Represents a failed operation."""
    
    def __init__(self, error: E):
        self.error = error
    
    def __repr__(self) -> str:
        return f"Failure({self.error})"
```

### Type-Safe Access Functions

```python
class Result(Generic[T, E]):
    # ... existing methods ...
    
    def get_value_or_default(self, default_value: T) -> T:
        """Get the value or return default if it's a failure."""
        raise NotImplementedError
    
    def get_value_or_else(self, default_fn: Callable[[E], T]) -> T:
        """Get the value or compute default using the error."""
        raise NotImplementedError
    
    def get_value_or_throw(self, error_fn: Optional[Callable[[E], Exception]] = None) -> T:
        """Get the value or throw an exception."""
        raise NotImplementedError

class Success(Result[T, E]):
    # ... existing methods ...
    
    def get_value_or_default(self, default_value: T) -> T:
        return self.value
    
    def get_value_or_else(self, default_fn: Callable[[E], T]) -> T:
        return self.value
    
    def get_value_or_throw(self, error_fn: Optional[Callable[[E], Exception]] = None) -> T:
        return self.value

class Failure(Result[T, E]):
    # ... existing methods ...
    
    def get_value_or_default(self, default_value: T) -> T:
        return default_value
    
    def get_value_or_else(self, default_fn: Callable[[E], T]) -> T:
        return default_fn(self.error)
    
    def get_value_or_throw(self, error_fn: Optional[Callable[[E], Exception]] = None) -> T:
        if error_fn:
            raise error_fn(self.error)
        if isinstance(self.error, Exception):
            raise self.error
        raise Exception(str(self.error))
```

## Basic Operations

Now let's implement the core operations for our Result type:

### Map Operation

The map operation transforms the success value while preserving failures:

```python
class Result(Generic[T, E]):
    # ... existing methods ...
    
    def map(self, fn: Callable[[T], U]) -> 'Result[U, E]':
        """Transform success value using the given function."""
        raise NotImplementedError

class Success(Result[T, E]):
    # ... existing methods ...
    
    def map(self, fn: Callable[[T], U]) -> Result[U, E]:
        """Transform the success value."""
        return Result.success(fn(self.value))

class Failure(Result[T, E]):
    # ... existing methods ...
    
    def map(self, fn: Callable[[T], U]) -> Result[U, E]:
        """Identity operation for Failure case."""
        return Failure(self.error)
```

### FlatMap (Bind) Operation

The flatMap operation allows chaining operations that return Results:

```python
class Result(Generic[T, E]):
    # ... existing methods ...
    
    def flat_map(self, fn: Callable[[T], 'Result[U, E]']) -> 'Result[U, E]':
        """Chain operations that return Results."""
        raise NotImplementedError

class Success(Result[T, E]):
    # ... existing methods ...
    
    def flat_map(self, fn: Callable[[T], Result[U, E]]) -> Result[U, E]:
        """Chain with another operation returning Result."""
        return fn(self.value)

class Failure(Result[T, E]):
    # ... existing methods ...
    
    def flat_map(self, fn: Callable[[T], Result[U, E]]) -> Result[U, E]:
        """Identity operation for Failure case."""
        return Failure(self.error)
```

### MapError Operation

Transform error values:

```python
class Result(Generic[T, E]):
    # ... existing methods ...
    
    def map_error(self, fn: Callable[[E], F]) -> 'Result[T, F]':
        """Transform error value using the given function."""
        raise NotImplementedError

class Success(Result[T, E]):
    # ... existing methods ...
    
    def map_error(self, fn: Callable[[E], F]) -> Result[T, F]:
        """Identity operation for Success case."""
        return self

class Failure(Result[T, E]):
    # ... existing methods ...
    
    def map_error(self, fn: Callable[[E], F]) -> Result[T, F]:
        """Transform the error value."""
        return Failure(fn(self.error))
```

### Match Operation

Pattern matching for handling both success and failure cases:

```python
class Result(Generic[T, E]):
    # ... existing methods ...
    
    def match(self, on_success: Callable[[T], R], on_failure: Callable[[E], R]) -> R:
        """Pattern matching for handling both success and failure."""
        raise NotImplementedError

class Success(Result[T, E]):
    # ... existing methods ...
    
    def match(self, on_success: Callable[[T], R], on_failure: Callable[[E], R]) -> R:
        """Apply the success function."""
        return on_success(self.value)

class Failure(Result[T, E]):
    # ... existing methods ...
    
    def match(self, on_success: Callable[[T], R], on_failure: Callable[[E], R]) -> R:
        """Apply the failure function."""
        return on_failure(self.error)
```

## Chaining Operations

Let's create a more complete version of our Result type with additional utility functions:

### Complete Result Implementation

```python
# result.py
from typing import TypeVar, Generic, Callable, Optional, List, Any, Union
import asyncio

T = TypeVar('T')  # Success value type
E = TypeVar('E')  # Error type
U = TypeVar('U')  # Another type for transformations
F = TypeVar('F')  # Another error type
R = TypeVar('R')  # Return type for match

class Result(Generic[T, E]):
    """Base class for Railway Oriented Programming pattern."""
    
    @staticmethod
    def success(value: T) -> 'Success[T, E]':
        """Create a Success result."""
        return Success(value)
    
    @staticmethod
    def failure(error: E) -> 'Failure[T, E]':
        """Create a Failure result."""
        return Failure(error)
    
    def is_success(self) -> bool:
        """Check if result is Success."""
        return isinstance(self, Success)
    
    def is_failure(self) -> bool:
        """Check if result is Failure."""
        return isinstance(self, Failure)
    
    def map(self, fn: Callable[[T], U]) -> 'Result[U, E]':
        """Transform success value using the given function."""
        raise NotImplementedError
    
    def flat_map(self, fn: Callable[[T], 'Result[U, E]']) -> 'Result[U, E]':
        """Chain operations that return Results."""
        raise NotImplementedError
    
    def map_error(self, fn: Callable[[E], F]) -> 'Result[T, F]':
        """Transform error value using the given function."""
        raise NotImplementedError
    
    def tap(self, fn: Callable[[T], None]) -> 'Result[T, E]':
        """Execute side effect on success value."""
        raise NotImplementedError
    
    def tap_error(self, fn: Callable[[E], None]) -> 'Result[T, E]':
        """Execute side effect on error value."""
        raise NotImplementedError
    
    def match(self, on_success: Callable[[T], R], on_failure: Callable[[E], R]) -> R:
        """Pattern matching for handling both success and failure."""
        raise NotImplementedError
    
    def get_value_or_default(self, default_value: T) -> T:
        """Get the value or return default if it's a failure."""
        raise NotImplementedError
    
    def get_value_or_else(self, default_fn: Callable[[E], T]) -> T:
        """Get the value or compute default using the error."""
        raise NotImplementedError
    
    def get_value_or_throw(self, error_fn: Optional[Callable[[E], Exception]] = None) -> T:
        """Get the value or throw an exception."""
        raise NotImplementedError
    
    def recover(self, fn: Callable[[E], T]) -> 'Result[T, E]':
        """Recover from failure by producing a success value."""
        raise NotImplementedError
    
    def recover_with(self, fn: Callable[[E], 'Result[T, E]']) -> 'Result[T, E]':
        """Recover from failure by producing a new Result."""
        raise NotImplementedError
    
    @staticmethod
    def from_exception(fn: Callable[[], T], error_mapper: Callable[[Exception], E] = lambda e: e) -> 'Result[T, E]':
        """Create a Result from a function that might throw an exception."""
        try:
            return Result.success(fn())
        except Exception as e:
            return Result.failure(error_mapper(e))
    
    @staticmethod
    async def from_awaitable(awaitable, error_mapper: Callable[[Exception], E] = lambda e: e) -> 'Result[T, E]':
        """Create a Result from an awaitable."""
        try:
            value = await awaitable
            return Result.success(value)
        except Exception as e:
            return Result.failure(error_mapper(e))


class Success(Result[T, E]):
    """Represents a successful operation."""
    
    def __init__(self, value: T):
        self.value = value
    
    def __repr__(self) -> str:
        return f"Success({self.value})"
    
    def map(self, fn: Callable[[T], U]) -> Result[U, E]:
        """Transform the success value."""
        return Result.success(fn(self.value))
    
    def flat_map(self, fn: Callable[[T], Result[U, E]]) -> Result[U, E]:
        """Chain with another operation returning Result."""
        return fn(self.value)
    
    def map_error(self, fn: Callable[[E], F]) -> Result[T, F]:
        """Identity operation for Success case."""
        return self
    
    def tap(self, fn: Callable[[T], None]) -> Result[T, E]:
        """Execute side effect on success value."""
        fn(self.value)
        return self
    
    def tap_error(self, fn: Callable[[E], None]) -> Result[T, E]:
        """Identity operation for Success case."""
        return self
    
    def match(self, on_success: Callable[[T], R], on_failure: Callable[[E], R]) -> R:
        """Apply the success function."""
        return on_success(self.value)
    
    def get_value_or_default(self, default_value: T) -> T:
        return self.value
    
    def get_value_or_else(self, default_fn: Callable[[E], T]) -> T:
        return self.value
    
    def get_value_or_throw(self, error_fn: Optional[Callable[[E], Exception]] = None) -> T:
        return self.value
    
    def recover(self, fn: Callable[[E], T]) -> Result[T, E]:
        return self
    
    def recover_with(self, fn: Callable[[E], Result[T, E]]) -> Result[T, E]:
        return self
    
    async def map_async(self, fn) -> Result[U, E]:
        try:
            new_value = await fn(self.value)
            return Result.success(new_value)
        except Exception as e:
            return Result.failure(e)
    
    async def flat_map_async(self, fn) -> Result[U, E]:
        try:
            return await fn(self.value)
        except Exception as e:
            return Result.failure(e)


class Failure(Result[T, E]):
    """Represents a failed operation."""
    
    def __init__(self, error: E):
        self.error = error
    
    def __repr__(self) -> str:
        return f"Failure({self.error})"
    
    def map(self, fn: Callable[[T], U]) -> Result[U, E]:
        """Identity operation for Failure case."""
        return Failure(self.error)
    
    def flat_map(self, fn: Callable[[T], Result[U, E]]) -> Result[U, E]:
        """Identity operation for Failure case."""
        return Failure(self.error)
    
    def map_error(self, fn: Callable[[E], F]) -> Result[T, F]:
        """Transform the error value."""
        return Failure(fn(self.error))
    
    def tap(self, fn: Callable[[T], None]) -> Result[T, E]:
        """Identity operation for Failure case."""
        return self
    
    def tap_error(self, fn: Callable[[E], None]) -> Result[T, E]:
        """Execute side effect on error value."""
        fn(self.error)
        return self
    
    def match(self, on_success: Callable[[T], R], on_failure: Callable[[E], R]) -> R:
        """Apply the failure function."""
        return on_failure(self.error)
    
    def get_value_or_default(self, default_value: T) -> T:
        return default_value
    
    def get_value_or_else(self, default_fn: Callable[[E], T]) -> T:
        return default_fn(self.error)
    
    def get_value_or_throw(self, error_fn: Optional[Callable[[E], Exception]] = None) -> T:
        if error_fn:
            raise error_fn(self.error)
        if isinstance(self.error, Exception):
            raise self.error
        raise Exception(str(self.error))
    
    def recover(self, fn: Callable[[E], T]) -> Result[T, E]:
        return Success(fn(self.error))
    
    def recover_with(self, fn: Callable[[E], Result[T, E]]) -> Result[T, E]:
        return fn(self.error)
    
    async def map_async(self, fn) -> Result[U, E]:
        return Failure(self.error)
    
    async def flat_map_async(self, fn) -> Result[U, E]:
        return Failure(self.error)
```

## Practical Examples

### Example 1: User Registration Flow

```python
# types.py
from dataclasses import dataclass
from datetime import datetime
from typing import Optional

@dataclass
class User:
    id: str
    email: str
    password_hash: str
    created_at: datetime

@dataclass
class ValidationError:
    field: str
    message: str
    
    def __str__(self):
        return f"{self.field}: {self.message}"

# user_service.py
import uuid
import hashlib
from datetime import datetime
from typing import Union
from result import Result
from types import User, ValidationError

class UserService:
    def __init__(self):
        self.users = {}  # Simple in-memory storage
    
    def register_user(self, email: str, password: str) -> Result[User, Union[ValidationError, str]]:
        return (self.validate_email(email)
                .flat_map(lambda _: self.validate_password(password))
                .flat_map(lambda _: self.check_email_not_exists(email))
                .flat_map(lambda _: self.create_user(email, password))
                .flat_map(lambda user: self.send_welcome_email(user))
                .tap(lambda user: print(f"User registered: {user.email}")))
    
    def validate_email(self, email: str) -> Result[str, ValidationError]:
        if not email or '@' not in email:
            return Result.failure(ValidationError(
                field="email",
                message="Invalid email format"
            ))
        return Result.success(email)
    
    def validate_password(self, password: str) -> Result[str, ValidationError]:
        if not password or len(password) < 8:
            return Result.failure(ValidationError(
                field="password",
                message="Password must be at least 8 characters"
            ))
        return Result.success(password)
    
    def check_email_not_exists(self, email: str) -> Result[str, ValidationError]:
        # Check if email exists in our "database"
        exists = any(user.email == email for user in self.users.values())
        
        if exists:
            return Result.failure(ValidationError(
                field="email",
                message="Email already registered"
            ))
        
        return Result.success(email)
    
    def create_user(self, email: str, password: str) -> Result[User, str]:
        try:
            user = User(
                id=str(uuid.uuid4()),
                email=email,
                password_hash=self.hash_password(password),
                created_at=datetime.now()
            )
            
            # Save to our "database"
            self.users[user.id] = user
            
            return Result.success(user)
        except Exception as e:
            return Result.failure(f"Failed to create user: {str(e)}")
    
    def send_welcome_email(self, user: User) -> Result[User, str]:
        try:
            # Simulate sending an email
            print(f"Sending welcome email to {user.email}")
            
            return Result.success(user)
        except Exception as e:
            return Result.failure(f"Failed to send welcome email: {str(e)}")
    
    def hash_password(self, password: str) -> str:
        # Simple password hashing (not for production)
        return hashlib.sha256(password.encode()).hexdigest()

# usage.py
from user_service import UserService

user_service = UserService()

# Success case
success_result = user_service.register_user("test@example.com", "password123")
success_result.match(
    lambda user: print(f"Registration successful: {user.email}"),
    lambda error: print(f"Registration failed: {error}")
)

# Failure case - invalid email
failure_result = user_service.register_user("invalid-email", "password123")
failure_result.match(
    lambda user: print(f"Registration successful: {user.email}"),
    lambda error: print(f"Registration failed: {error}")
)
```

### Example 2: Order Processing Pipeline

```python
# order_types.py
from dataclasses import dataclass
from datetime import datetime
from typing import List, Optional

@dataclass
class OrderItem:
    product_id: int
    quantity: int
    unit_price: Optional[float] = None

@dataclass
class OrderRequest:
    customer_id: int
    items: List[OrderItem]

@dataclass
class PricedOrder:
    request: OrderRequest
    total_amount: float
    discount: float
    final_amount: float

@dataclass
class PaymentResult:
    order: PricedOrder
    transaction_id: str
    status: str  # 'APPROVED' or 'DECLINED'

@dataclass
class Order:
    order_id: str
    customer_id: int
    items: List[OrderItem]
    total_amount: float
    discount: float
    final_amount: float
    payment_transaction_id: str
    order_status: str  # 'CONFIRMED' or 'CANCELLED'
    created_at: datetime

@dataclass
class OrderError:
    code: str
    message: str
    timestamp: datetime
    trace_id: Optional[str] = None
    
    def __str__(self):
        return f"[{self.code}] {self.message}"

# order_service.py
import uuid
from datetime import datetime
from result import Result
from order_types import *

class OrderService:
    def process_order(self, request: OrderRequest) -> Result[Order, OrderError]:
        return (self.validate_order_request(request)
                .flat_map(lambda req: self.check_inventory(req))
                .flat_map(lambda req: self.calculate_pricing(req))
                .flat_map(lambda priced_order: self.apply_discounts(priced_order))
                .flat_map(lambda priced_order: self.process_payment(priced_order))
                .flat_map(lambda payment_result: self.create_order(payment_result))
                .flat_map(lambda order: self.update_inventory(order))
                .map_error(lambda error: self.enrich_error(error))
                .tap(lambda order: self.log_success(order))
                .tap_error(lambda error: self.log_error(error)))
    
    def validate_order_request(self, request: OrderRequest) -> Result[OrderRequest, OrderError]:
        if not request.items or len(request.items) == 0:
            return Result.failure(OrderError(
                code="EMPTY_ORDER",
                message="Order has no items",
                timestamp=datetime.now()
            ))
        return Result.success(request)
    
    def check_inventory(self, request: OrderRequest) -> Result[OrderRequest, OrderError]:
        for item in request.items:
            if not self.has_stock(item):
                return Result.failure(OrderError(
                    code="OUT_OF_STOCK",
                    message=f"Item {item.product_id} is out of stock",
                    timestamp=datetime.now()
                ))
        return Result.success(request)
    
    def calculate_pricing(self, request: OrderRequest) -> Result[PricedOrder, OrderError]:
        try:
            # Add prices to items first
            items_with_prices = []
            for item in request.items:
                price = self.get_price(item.product_id)
                items_with_prices.append(OrderItem(
                    product_id=item.product_id,
                    quantity=item.quantity,
                    unit_price=price
                ))
            
            total_amount = sum(item.quantity * (item.unit_price or 0) for item in items_with_prices)
            
            priced_order = PricedOrder(
                request=OrderRequest(
                    customer_id=request.customer_id,
                    items=items_with_prices
                ),
                total_amount=total_amount,
                discount=0,  # Will be calculated in apply_discounts
                final_amount=total_amount  # Will be updated in apply_discounts
            )
            
            return Result.success(priced_order)
        except Exception as e:
            return Result.failure(OrderError(
                code="PRICING_ERROR",
                message=f"Failed to calculate price: {str(e)}",
                timestamp=datetime.now()
            ))
    
    def apply_discounts(self, priced_order: PricedOrder) -> Result[PricedOrder, OrderError]:
        try:
            discount = 0
            
            # $10 off for orders over $100
            if priced_order.total_amount > 100:
                discount += 10
            
            # Additional $5 off for loyal customers
            if self.is_loyal_customer(priced_order.request.customer_id):
                discount += 5
            
            updated_order = PricedOrder(
                request=priced_order.request,
                total_amount=priced_order.total_amount,
                discount=discount,
                final_amount=priced_order.total_amount - discount
            )
            
            return Result.success(updated_order)
        except Exception as e:
            return Result.failure(OrderError(
                code="DISCOUNT_ERROR",
                message=f"Failed to apply discounts: {str(e)}",
                timestamp=datetime.now()
            ))
    
    def process_payment(self, priced_order: PricedOrder) -> Result[PaymentResult, OrderError]:
        try:
            # Simulate payment processing
            payment_result = PaymentResult(
                order=priced_order,
                transaction_id=str(uuid.uuid4()),
                status="APPROVED"
            )
            
            return Result.success(payment_result)
        except Exception as e:
            return Result.failure(OrderError(
                code="PAYMENT_ERROR",
                message=f"Payment processing failed: {str(e)}",
                timestamp=datetime.now()
            ))
    
    def create_order(self, payment_result: PaymentResult) -> Result[Order, OrderError]:
        try:
            order = Order(
                order_id=str(uuid.uuid4()),
                customer_id=payment_result.order.request.customer_id,
                items=payment_result.order.request.items,
                total_amount=payment_result.order.total_amount,
                discount=payment_result.order.discount,
                final_amount=payment_result.order.final_amount,
                payment_transaction_id=payment_result.transaction_id,
                order_status="CONFIRMED",
                created_at=datetime.now()
            )
            
            # Save to database would happen here
            
            return Result.success(order)
        except Exception as e:
            return Result.failure(OrderError(
                code="ORDER_CREATION_ERROR",
                message=f"Failed to create order: {str(e)}",
                timestamp=datetime.now()
            ))
    
    def update_inventory(self, order: Order) -> Result[Order, OrderError]:
        try:
            # Update inventory for each item
            for item in order.items:
                self.update_product_stock(item.product_id, item.quantity)
            
            return Result.success(order)
        except Exception as e:
            # Log error but still return success (might want different behavior in real system)
            self.log_inventory_update_error(e, order)
            return Result.success(order)
    
    def enrich_error(self, error: OrderError) -> OrderError:
        return OrderError(
            code=error.code,
            message=error.message,
            timestamp=error.timestamp,
            trace_id=str(uuid.uuid4())
        )
    
    # Helper methods
    def has_stock(self, item: OrderItem) -> bool:
        # Inventory check simulation
        return True
    
    def get_price(self, product_id: int) -> float:
        # Price lookup simulation
        return 10.0 * product_id
    
    def is_loyal_customer(self, customer_id: int) -> bool:
        # Customer status check simulation
        return customer_id % 2 == 0
    
    def update_product_stock(self, product_id: int, quantity: int) -> None:
        # Update inventory simulation
        print(f"Updating stock for product {product_id}, quantity: -{quantity}")
    
    def log_success(self, order: Order) -> None:
        print(f"Order {order.order_id} processed successfully")
    
    def log_error(self, error: OrderError) -> None:
        print(f"Error: [{error.code}] {error.message}")
    
    def log_inventory_update_error(self, error: Exception, order: Order) -> None:
        print(f"Warning: Inventory update failed for order {order.order_id}: {str(error)}")

# usage.py
from order_service import OrderService
from order_types import OrderRequest, OrderItem

order_service = OrderService()

# Success case
result = order_service.process_order(OrderRequest(
    customer_id=1,
    items=[
        OrderItem(product_id=101, quantity=2),
        OrderItem(product_id=202, quantity=1)
    ]
))

result.match(
    lambda order: print(f"Order processed: {order.order_id}, Total: ${order.final_amount}"),
    lambda error: print(f"Order processing failed: {error}")
)
```

### Example 3: File Processing with Error Recovery

```python
# file_types.py
from dataclasses import dataclass
from datetime import datetime
from typing import List

@dataclass
class FileData:
    id: str
    name: str
    values: List[int]

@dataclass
class TransformedData:
    id: str
    name: str
    processed_date: datetime
    values: List[int]

@dataclass
class ProcessedFile:
    original_id: str
    output_path: str
    processed_date: datetime

@dataclass
class FileError:
    code: str
    message: str
    
    def __str__(self):
        return f"{self.code}: {self.message}"

# file_processor.py
import os
import json
import uuid
from datetime import datetime
from result import Result
from file_types import *

class FileProcessor:
    def process_file(self, file_path: str) -> Result[ProcessedFile, FileError]:
        return (self.read_file(file_path)
                .flat_map(lambda content: self.validate_format(content))
                .flat_map(lambda content: self.parse_content(content))
                .flat_map(lambda data: self.transform(data))
                .flat_map(lambda transformed: self.save(transformed))
                .recover_with(lambda error: 
                    self.try_alternative_parser(file_path) if error.code == 'PARSE_ERROR' 
                    else Result.failure(error)
                ))
    
    def read_file(self, file_path: str) -> Result[str, FileError]:
        try:
            if not os.path.exists(file_path):
                return Result.failure(FileError(
                    code="FILE_NOT_FOUND",
                    message=f"File not found: {file_path}"
                ))
            
            with open(file_path, 'r', encoding='utf-8') as file:
                content = file.read()
            return Result.success(content)
        except Exception as e:
            return Result.failure(FileError(
                code="READ_ERROR",
                message=f"Cannot read file: {str(e)}"
            ))
    
    def validate_format(self, content: str) -> Result[str, FileError]:
        if (content.startswith('<?xml') or 
                content.startswith('{') or 
                content.startswith('[')):
            return Result.success(content)
        
        return Result.failure(FileError(
            code="FORMAT_ERROR",
            message="Invalid file format"
        ))
    
    def parse_content(self, content: str) -> Result[FileData, FileError]:
        try:
            # Parsing logic depends on file type
            if content.startswith('<?xml'):
                return self.parse_xml(content)
            else:
                return self.parse_json(content)
        except Exception as e:
            return Result.failure(FileError(
                code="PARSE_ERROR",
                message=f"Failed to parse content: {str(e)}"
            ))
    
    def transform(self, data: FileData) -> Result[TransformedData, FileError]:
        try:
            # Apply business transformations
            transformed = TransformedData(
                id=data.id,
                name=data.name.upper(),  # Simple transformation
                processed_date=datetime.now(),
                values=[v * 2 for v in data.values]  # Double all values
            )
            
            return Result.success(transformed)
        except Exception as e:
            return Result.failure(FileError(
                code="TRANSFORM_ERROR",
                message=f"Failed to transform data: {str(e)}"
            ))
    
    def save(self, data: TransformedData) -> Result[ProcessedFile, FileError]:
        try:
            # Create output filename
            output_path = f"processed_{data.id}_{int(datetime.now().timestamp())}.json"
            
            # Simulate saving
            json_data = json.dumps({
                "id": data.id,
                "name": data.name,
                "processedDate": data.processed_date.isoformat(),
                "values": data.values
            }, indent=2)
            # Uncomment in real implementation
            # with open(output_path, 'w', encoding='utf-8') as file:
            #     file.write(json_data)
            
            processed_file = ProcessedFile(
                original_id=data.id,
                output_path=output_path,
                processed_date=data.processed_date
            )
            
            return Result.success(processed_file)
        except Exception as e:
            return Result.failure(FileError(
                code="SAVE_ERROR",
                message=f"Failed to save processed data: {str(e)}"
            ))
    
    def try_alternative_parser(self, file_path: str) -> Result[ProcessedFile, FileError]:
        print(f"Trying alternative parser for {file_path}")
        
        return (self.read_file(file_path)
                .flat_map(lambda content: self.parse_with_fallback_strategy(content))
                .flat_map(lambda data: self.transform(data))
                .flat_map(lambda transformed: self.save(transformed)))
    
    def parse_with_fallback_strategy(self, content: str) -> Result[FileData, FileError]:
        try:
            # Implement a more forgiving parser
            # For this example, create fallback data
            fallback_data = FileData(
                id=str(uuid.uuid4()),
                name="Recovered Data",
                values=[1, 2, 3]
            )
            
            return Result.success(fallback_data)
        except Exception as e:
            return Result.failure(FileError(
                code="FALLBACK_PARSE_ERROR",
                message=f"Even fallback parsing failed: {str(e)}"
            ))
    
    def parse_xml(self, content: str) -> Result[FileData, FileError]:
        # XML parsing logic (simplified)
        return Result.success(FileData(
            id=str(uuid.uuid4()),
            name="XML Data",
            values=[10, 20, 30]
        ))
    
    def parse_json(self, content: str) -> Result[FileData, FileError]:
        # JSON parsing logic (simplified)
        try:
            parsed = json.loads(content)
            return Result.success(FileData(
                id=parsed.get('id', str(uuid.uuid4())),
                name=parsed.get('name', 'JSON Data'),
                values=parsed.get('values', [100, 200, 300]) if isinstance(parsed.get('values'), list) else [100, 200, 300]
            ))
        except:
            # If strict parsing fails, return mock data
            return Result.success(FileData(
                id=str(uuid.uuid4()),
                name="JSON Data",
                values=[100, 200, 300]
            ))

# usage.py
from file_processor import FileProcessor

processor = FileProcessor()

# Success case (if the file exists)
result = processor.process_file('./data.json')

result.match(
    lambda file: print(f"File processed: {file.output_path}"),
    lambda error: print(f"Processing failed: {error}")
)
```

## Advanced Patterns

### Combining Multiple Results

```python
# result_combinators.py
from typing import List, TypeVar, Generic, Callable, Tuple
from result import Result

T = TypeVar('T')
U = TypeVar('U')
V = TypeVar('V')
E = TypeVar('E')

class ResultCombinators:
    @staticmethod
    def combine2(result1: Result[T, E], result2: Result[U, E]) -> Result[Tuple[T, U], E]:
        """Combine two results into a tuple."""
        return result1.flat_map(lambda value1: 
            result2.map(lambda value2: (value1, value2))
        )
    
    @staticmethod
    def combine3(result1: Result[T, E], result2: Result[U, E], result3: Result[V, E]) -> Result[Tuple[T, U, V], E]:
        """Combine three results into a tuple."""
        return ResultCombinators.combine2(result1, result2).flat_map(
            lambda values: result3.map(lambda value3: (*values, value3))
        )
    
    @staticmethod
    def sequence(results: List[Result[T, E]]) -> Result[List[T], E]:
        """Turn a list of Results into a Result of list."""
        values = []
        
        for result in results:
            if result.is_failure():
                return result  # Return the first failure
            values.append(result.value)
        
        return Result.success(values)
    
    @staticmethod
    def traverse(results: List[Result[T, E]]) -> Result[List[T], List[E]]:
        """Collect all successes and failures."""
        successes = []
        failures = []
        
        for result in results:
            if result.is_success():
                successes.append(result.value)
            else:
                failures.append(result.error)
        
        return Result.failure(failures) if failures else Result.success(successes)
    
    @staticmethod
    def traverse_array(items: List[T], fn: Callable[[T], Result[U, E]]) -> Result[List[U], E]:
        """Process a list of items with a fallible function."""
        results = [fn(item) for item in items]
        return ResultCombinators.sequence(results)
```

### Async Operations with Promises

```python
# async_result.py
import asyncio
from typing import List, TypeVar, Callable, Awaitable
from result import Result
from result_combinators import ResultCombinators

T = TypeVar('T')
U = TypeVar('U')
E = TypeVar('E')

class AsyncResult:
    @staticmethod
    async def from_awaitable(awaitable: Awaitable[T], error_mapper: Callable[[Exception], E] = lambda e: e) -> Result[T, E]:
        """Create a Result from an awaitable."""
        try:
            value = await awaitable
            return Result.success(value)
        except Exception as e:
            return Result.failure(error_mapper(e))
    
    @staticmethod
    async def map_async(result: Result[T, E], fn: Callable[[T], Awaitable[U]]) -> Result[U, E]:
        """Transform success value using an async function."""
        if result.is_success():
            try:
                new_value = await fn(result.value)
                return Result.success(new_value)
            except Exception as e:
                return Result.failure(e)
        return Result.failure(result.error)
    
    @staticmethod
    async def flat_map_async(result: Result[T, E], fn: Callable[[T], Awaitable[Result[U, E]]]) -> Result[U, E]:
        """Chain operations that return awaitable Results."""
        if result.is_success():
            try:
                return await fn(result.value)
            except Exception as e:
                return Result.failure(e)
        return Result.failure(result.error)
    
    @staticmethod
    async def sequence_async(awaitables: List[Awaitable[Result[T, E]]]) -> Result[List[T], E]:
        """Turn a list of awaitable Results into a Result of list."""
        results = await asyncio.gather(*awaitables, return_exceptions=True)
        
        # Handle any exceptions from awaiting
        processed_results = []
        for result in results:
            if isinstance(result, Exception):
                return Result.failure(result)
            processed_results.append(result)
        
        return ResultCombinators.sequence(processed_results)

# Async example
async def fetch_user_data(user_id: str) -> Result[dict, str]:
    try:
        # Simulate async API call
        await asyncio.sleep(1)
        # Return mock data
        return Result.success({
            "id": user_id,
            "name": f"User {user_id}",
            "email": f"user{user_id}@example.com"
        })
    except Exception as e:
        return Result.failure(f"Failed to fetch user: {str(e)}")

async def process_user_async():
    result = await fetch_user_data("123")
    
    result.match(
        lambda user: print(f"User: {user['name']}"),
        lambda error: print(f"Error: {error}")
    )
```

## Best Practices

### 1. Use Specific Error Types

```python
# domain_errors.py
from dataclasses import dataclass
from typing import Optional

@dataclass
class DomainError:
    type: str
    message: str

@dataclass
class ValidationError(DomainError):
    field: str
    
    def __init__(self, field: str, message: str):
        super().__init__(type="validation", message=message)
        self.field = field

@dataclass
class NotFoundError(DomainError):
    entity_type: str
    entity_id: str
    
    def __init__(self, entity_type: str, entity_id: str):
        super().__init__(
            type="not_found", 
            message=f"{entity_type} with ID '{entity_id}' not found"
        )
        self.entity_type = entity_type
        self.entity_id = entity_id

@dataclass
class ServiceError(DomainError):
    original_error: Optional[Exception] = None
    
    def __init__(self, message: str, original_error: Optional[Exception] = None):
        super().__init__(type="service", message=message)
        self.original_error = original_error


class DomainErrors:
    @staticmethod
    def validation(field: str, message: str) -> ValidationError:
        return ValidationError(field=field, message=message)
    
    @staticmethod
    def not_found(entity_type: str, entity_id: str) -> NotFoundError:
        return NotFoundError(entity_type=entity_type, entity_id=entity_id)
    
    @staticmethod
    def service(message: str, original_error: Optional[Exception] = None) -> ServiceError:
        return ServiceError(message=message, original_error=original_error)
```

### 2. Create Domain-Specific Result Types

```python
# domain_result.py
from typing import TypeVar, Generic
from result import Result
from domain_errors import DomainError, ValidationError, NotFoundError, ServiceError, DomainErrors

T = TypeVar('T')
DomainResult = Result[T, DomainError]

class DomainResultFactory:
    @staticmethod
    def success(value: T) -> DomainResult[T]:
        return Result.success(value)
    
    @staticmethod
    def failure(error: DomainError) -> DomainResult[T]:
        return Result.failure(error)
    
    @staticmethod
    def invalid(field: str, message: str) -> DomainResult[T]:
        return Result.failure(DomainErrors.validation(field, message))
    
    @staticmethod
    def not_found(entity_type: str, entity_id: str) -> DomainResult[T]:
        return Result.failure(DomainErrors.not_found(entity_type, entity_id))
    
    @staticmethod
    def from_error(message: str, error: Optional[Exception] = None) -> DomainResult[T]:
        return Result.failure(DomainErrors.service(message, error))
    
    @staticmethod
    def ensure(value: Optional[T], entity_type: str, entity_id: str) -> DomainResult[T]:
        return (DomainResultFactory.success(value) 
                if value is not None 
                else DomainResultFactory.not_found(entity_type, entity_id))
```

### 3. Use Extension Methods for Domain-Specific Operations

```python
# domain_result_extensions.py
from typing import TypeVar, Callable
from result import Result
from domain_result import DomainResult, DomainResultFactory
from domain_errors import DomainError, ValidationError, NotFoundError, DomainErrors

T = TypeVar('T')

def ensure_valid(
    result: DomainResult[T], 
    predicate: Callable[[T], bool], 
    field: str, 
    message: str
) -> DomainResult[T]:
    return result.flat_map(lambda value: 
        DomainResultFactory.success(value) if predicate(value) 
        else DomainResultFactory.invalid(field, message)
    )

def ensure_exists(
    result: DomainResult[T], 
    predicate: Callable[[T], bool], 
    entity_type: str, 
    entity_id: str
) -> DomainResult[T]:
    return result.flat_map(lambda value: 
        DomainResultFactory.success(value) if predicate(value) 
        else DomainResultFactory.not_found(entity_type, entity_id)
    )

def with_error_context(
    result: DomainResult[T], 
    context: str
) -> DomainResult[T]:
    return result.map_error(lambda error: 
        DomainError(
            type=error.type,
            message=f"{context}: {error.message}"
        )
    )
```

### 4. Use Builder Pattern for Complex Flows

```python
# user_registration_flow.py
import uuid
import hashlib
from datetime import datetime
from typing import Dict, Optional
from dataclasses import dataclass

from result import Result
from domain_result import DomainResult, DomainResultFactory
from domain_result_extensions import ensure_valid
from domain_errors import DomainError

@dataclass
class User:
    id: str
    email: str
    password_hash: str
    created_at: datetime

class UserRepository:
    def exists_by_email(self, email: str) -> bool:
        # Implementation
        pass
    
    def save(self, user: User) -> None:
        # Implementation
        pass

class EmailService:
    def send_welcome_email(self, email: str) -> None:
        # Implementation
        pass

class UserRegistrationFlow:
    def __init__(self, user_repository: UserRepository, email_service: EmailService):
        self.user_repository = user_repository
        self.email_service = email_service
    
    def register(self, email: str, password: str) -> DomainResult[User]:
        # Start with a value object
        return (DomainResultFactory.success({"email": email, "password": password})
                .flat_map(self.validate_request)
                .flat_map(self.check_user_does_not_exist)
                .flat_map(self.create_user)
                .flat_map(self.send_welcome_email))
    
    def validate_request(self, request: Dict[str, str]) -> DomainResult[Dict[str, str]]:
        return (DomainResultFactory.success(request)
                .flat_map(lambda req: ensure_valid(
                    DomainResultFactory.success(req), 
                    lambda r: bool(r["email"]), 
                    "email", 
                    "Email is required"
                ))
                .flat_map(lambda req: ensure_valid(
                    DomainResultFactory.success(req), 
                    lambda r: "@" in r["email"], 
                    "email", 
                    "Invalid email format"
                ))
                .flat_map(lambda req: ensure_valid(
                    DomainResultFactory.success(req), 
                    lambda r: bool(r["password"]), 
                    "password", 
                    "Password is required"
                ))
                .flat_map(lambda req: ensure_valid(
                    DomainResultFactory.success(req), 
                    lambda r: len(r["password"]) >= 8, 
                    "password", 
                    "Password must be at least 8 characters"
                )))
    
    def check_user_does_not_exist(self, request: Dict[str, str]) -> DomainResult[Dict[str, str]]:
        if self.user_repository.exists_by_email(request["email"]):
            return DomainResultFactory.invalid("email", "Email already registered")
        return DomainResultFactory.success(request)
    
    def create_user(self, request: Dict[str, str]) -> DomainResult[User]:
        try:
            user = User(
                id=str(uuid.uuid4()),
                email=request["email"],
                password_hash=self.hash_password(request["password"]),
                created_at=datetime.now()
            )
            
            self.user_repository.save(user)
            
            return DomainResultFactory.success(user)
        except Exception as error:
            return DomainResultFactory.from_error(
                "Failed to create user", 
                error
            )
    
    def send_welcome_email(self, user: User) -> DomainResult[User]:
        try:
            self.email_service.send_welcome_email(user.email)
            return DomainResultFactory.success(user)
        except Exception as error:
            # Continue with success but log error
            print(f"Failed to send welcome email: {str(error)}")
            return DomainResultFactory.success(user)
    
    def hash_password(self, password: str) -> str:
        return hashlib.sha256(password.encode()).hexdigest()
```

### 5. Testing ROP Flows

```python
# user_registration_flow_test.py
import unittest
from datetime import datetime
import uuid
import hashlib
from typing import List, Optional, Dict

from domain_result import DomainResult
from user_registration_flow import User, UserRepository, EmailService, UserRegistrationFlow

# Mock implementations for testing
class MockUserRepository(UserRepository):
    def __init__(self):
        self.users: List[User] = []
    
    def exists_by_email(self, email: str) -> bool:
        return any(u.email == email for u in self.users)
    
    def save(self, user: User) -> None:
        self.users.append(user)

class MockEmailService(EmailService):
    def __init__(self):
        self.email_sent = False
    
    def send_welcome_email(self, email: str) -> None:
        self.email_sent = True

class UserRegistrationFlowTests(unittest.TestCase):
    def setUp(self):
        self.user_repo = MockUserRepository()
        self.email_service = MockEmailService()
        self.flow = UserRegistrationFlow(self.user_repo, self.email_service)
    
    def test_register_valid_user_successfully(self):
        # Arrange
        email = "test@example.com"
        password = "password123"
        
        # Act
        result = self.flow.register(email, password)
        
        # Assert
        self.assertTrue(result.is_success())
        if result.is_success():
            self.assertEqual(result.value.email, email)
            self.assertTrue(self.email_service.email_sent)
    
    def test_fail_when_email_is_invalid(self):
        # Arrange
        email = "invalid-email"
        password = "password123"
        
        # Act
        result = self.flow.register(email, password)
        
        # Assert
        self.assertTrue(result.is_failure())
        if result.is_failure():
            self.assertEqual(result.error.type, "validation")
            self.assertEqual(result.error.field, "email")
    
    def test_fail_when_password_is_too_short(self):
        # Arrange
        email = "test@example.com"
        password = "short"
        
        # Act
        result = self.flow.register(email, password)
        
        # Assert
        self.assertTrue(result.is_failure())
        if result.is_failure():
            self.assertEqual(result.error.type, "validation")
            self.assertEqual(result.error.field, "password")

if __name__ == "__main__":
    unittest.main()
```