# Railway Oriented Programming in TypeScript: A Complete Tutorial

## Introduction

Railway Oriented Programming (ROP) is a functional programming pattern for handling errors in a composable way. It models operations as railway tracks: success cases continue on the main track, while failures divert to a parallel error track. This tutorial shows you how to implement ROP in TypeScript using a custom Result type.

## Table of Contents

1. [Understanding the Concept](#understanding-the-concept)
2. [Creating the Result Type](#creating-the-result-type)
3. [Basic Operations](#basic-operations)
4. [Chaining Operations](#chaining-operations)
5. [Practical Examples](#practical-examples)
6. [Advanced Patterns](#advanced-patterns)
7. [Best Practices](#best-practices)

## Understanding the Concept

In traditional TypeScript error handling, we often see nested try-catch blocks, multiple if-else statements, or exceptions. ROP provides a cleaner alternative by encapsulating success and failure in a single type that can be composed.

## Creating the Result Type

Let's start by creating our core Result type using TypeScript's discriminated unions:

### Basic Result Type

```typescript
// Discriminated union type for Result
type Result<T, E> = Success<T, E> | Failure<T, E>;

// Success case
interface Success<T, E> {
  readonly type: 'success';
  readonly value: T;
}

// Failure case
interface Failure<T, E> {
  readonly type: 'failure';
  readonly error: E;
}

// Factory functions
function success<T, E>(value: T): Result<T, E> {
  return { type: 'success', value };
}

function failure<T, E>(error: E): Result<T, E> {
  return { type: 'failure', error };
}

// Type guards for safe type checking
function isSuccess<T, E>(result: Result<T, E>): result is Success<T, E> {
  return result.type === 'success';
}

function isFailure<T, E>(result: Result<T, E>): result is Failure<T, E> {
  return result.type === 'failure';
}
```

### Type-Safe Access Functions

```typescript
// Safe access to value and error
function getValue<T, E>(result: Result<T, E>): T {
  if (isSuccess(result)) {
    return result.value;
  }
  throw new Error('Cannot get value from a failure result');
}

function getError<T, E>(result: Result<T, E>): E {
  if (isFailure(result)) {
    return result.error;
  }
  throw new Error('Cannot get error from a success result');
}

// Non-throwing accessors with default values
function getValueOrDefault<T, E>(result: Result<T, E>, defaultValue: T): T {
  return isSuccess(result) ? result.value : defaultValue;
}

function getValueOrElse<T, E>(result: Result<T, E>, defaultFn: (error: E) => T): T {
  return isSuccess(result) ? result.value : defaultFn(result.error);
}
```

## Basic Operations

Now let's implement the core operations for our Result type:

### Map Operation

The map operation transforms the success value while preserving failures:

```typescript
function map<T, E, U>(result: Result<T, E>, fn: (value: T) => U): Result<U, E> {
  return isSuccess(result)
    ? success(fn(result.value))
    : result as unknown as Result<U, E>;
}

// Method-style usage with prototype extension
declare global {
  interface Object {
    map<T, E, U>(this: Result<T, E>, fn: (value: T) => U): Result<U, E>;
  }
}

Object.prototype.map = function<T, E, U>(this: Result<T, E>, fn: (value: T) => U): Result<U, E> {
  return map(this, fn);
};
```

### FlatMap (Bind) Operation

The flatMap operation allows chaining operations that return Results:

```typescript
function flatMap<T, E, U>(result: Result<T, E>, fn: (value: T) => Result<U, E>): Result<U, E> {
  return isSuccess(result)
    ? fn(result.value)
    : result as unknown as Result<U, E>;
}

declare global {
  interface Object {
    flatMap<T, E, U>(this: Result<T, E>, fn: (value: T) => Result<U, E>): Result<U, E>;
  }
}

Object.prototype.flatMap = function<T, E, U>(
  this: Result<T, E>, 
  fn: (value: T) => Result<U, E>
): Result<U, E> {
  return flatMap(this, fn);
};
```

### MapError Operation

Transform error values:

```typescript
function mapError<T, E, F>(result: Result<T, E>, fn: (error: E) => F): Result<T, F> {
  return isFailure(result)
    ? failure(fn(result.error))
    : result as unknown as Result<T, F>;
}

declare global {
  interface Object {
    mapError<T, E, F>(this: Result<T, E>, fn: (error: E) => F): Result<T, F>;
  }
}

Object.prototype.mapError = function<T, E, F>(
  this: Result<T, E>, 
  fn: (error: E) => F
): Result<T, F> {
  return mapError(this, fn);
};
```

### Match Operation

Pattern matching for handling both success and failure cases:

```typescript
function match<T, E, R>(
  result: Result<T, E>, 
  onSuccess: (value: T) => R, 
  onFailure: (error: E) => R
): R {
  return isSuccess(result)
    ? onSuccess(result.value)
    : onFailure(result.error);
}

declare global {
  interface Object {
    match<T, E, R>(
      this: Result<T, E>, 
      onSuccess: (value: T) => R, 
      onFailure: (error: E) => R
    ): R;
  }
}

Object.prototype.match = function<T, E, R>(
  this: Result<T, E>, 
  onSuccess: (value: T) => R, 
  onFailure: (error: E) => R
): R {
  return match(this, onSuccess, onFailure);
};
```

## Chaining Operations

Let's create a more complete version of our Result type with additional utility functions:

### Complete Result Implementation

```typescript
// result.ts
export type Result<T, E> = Success<T, E> | Failure<T, E>;

export interface Success<T, E> {
  readonly type: 'success';
  readonly value: T;
}

export interface Failure<T, E> {
  readonly type: 'failure';
  readonly error: E;
}

export const Result = {
  success<T, E>(value: T): Result<T, E> {
    return { type: 'success', value };
  },

  failure<T, E>(error: E): Result<T, E> {
    return { type: 'failure', error };
  },

  isSuccess<T, E>(result: Result<T, E>): result is Success<T, E> {
    return result.type === 'success';
  },

  isFailure<T, E>(result: Result<T, E>): result is Failure<T, E> {
    return result.type === 'failure';
  },

  map<T, E, U>(result: Result<T, E>, fn: (value: T) => U): Result<U, E> {
    return Result.isSuccess(result)
      ? Result.success(fn(result.value))
      : result as unknown as Result<U, E>;
  },

  flatMap<T, E, U>(result: Result<T, E>, fn: (value: T) => Result<U, E>): Result<U, E> {
    return Result.isSuccess(result)
      ? fn(result.value)
      : result as unknown as Result<U, E>;
  },

  mapError<T, E, F>(result: Result<T, E>, fn: (error: E) => F): Result<T, F> {
    return Result.isFailure(result)
      ? Result.failure(fn(result.error))
      : result as unknown as Result<T, F>;
  },

  tap<T, E>(result: Result<T, E>, fn: (value: T) => void): Result<T, E> {
    if (Result.isSuccess(result)) {
      fn(result.value);
    }
    return result;
  },

  tapError<T, E>(result: Result<T, E>, fn: (error: E) => void): Result<T, E> {
    if (Result.isFailure(result)) {
      fn(result.error);
    }
    return result;
  },

  match<T, E, R>(
    result: Result<T, E>, 
    onSuccess: (value: T) => R, 
    onFailure: (error: E) => R
  ): R {
    return Result.isSuccess(result)
      ? onSuccess(result.value)
      : onFailure(result.error);
  },

  getValueOrDefault<T, E>(result: Result<T, E>, defaultValue: T): T {
    return Result.isSuccess(result) ? result.value : defaultValue;
  },

  getValueOrElse<T, E>(result: Result<T, E>, defaultFn: (error: E) => T): T {
    return Result.isSuccess(result) ? result.value : defaultFn(result.error);
  },

  getValueOrThrow<T, E>(result: Result<T, E>, errorFn?: (error: E) => Error): T {
    if (Result.isSuccess(result)) {
      return result.value;
    }
    
    if (errorFn) {
      throw errorFn(result.error);
    }
    
    if (result.error instanceof Error) {
      throw result.error;
    }
    
    throw new Error(String(result.error));
  },

  recover<T, E>(result: Result<T, E>, fn: (error: E) => T): Result<T, E> {
    return Result.isFailure(result)
      ? Result.success<T, E>(fn(result.error))
      : result;
  },

  recoverWith<T, E>(result: Result<T, E>, fn: (error: E) => Result<T, E>): Result<T, E> {
    return Result.isFailure(result)
      ? fn(result.error)
      : result;
  },
  
  // Async operations
  fromPromise<T, E = Error>(promise: Promise<T>): Promise<Result<T, E>> {
    return promise
      .then(value => Result.success<T, E>(value))
      .catch(error => Result.failure<T, E>(error as E));
  },
  
  mapAsync<T, E, U>(result: Result<T, E>, fn: (value: T) => Promise<U>): Promise<Result<U, E>> {
    return Result.isSuccess(result)
      ? fn(result.value).then(newValue => Result.success<U, E>(newValue))
      : Promise.resolve(result as unknown as Result<U, E>);
  },
  
  flatMapAsync<T, E, U>(
    result: Result<T, E>, 
    fn: (value: T) => Promise<Result<U, E>>
  ): Promise<Result<U, E>> {
    return Result.isSuccess(result)
      ? fn(result.value)
      : Promise.resolve(result as unknown as Result<U, E>);
  },
};
```

## Practical Examples

### Example 1: User Registration Flow

```typescript
// types.ts
export interface User {
  id: string;
  email: string;
  passwordHash: string;
  createdAt: Date;
}

export type ValidationError = {
  field: string;
  message: string;
};

// userService.ts
import { Result } from './result';
import { User, ValidationError } from './types';
import * as crypto from 'crypto';

export class UserService {
  private users: Map<string, User> = new Map();
  
  registerUser(email: string, password: string): Result<User, ValidationError | string> {
    return this.validateEmail(email)
      .flatMap(() => this.validatePassword(password))
      .flatMap(() => this.checkEmailNotExists(email))
      .flatMap(() => this.createUser(email, password))
      .flatMap(user => this.sendWelcomeEmail(user))
      .tap(user => console.log(`User registered: ${user.email}`));
  }
  
  private validateEmail(email: string): Result<string, ValidationError> {
    if (!email || !email.includes('@')) {
      return Result.failure({
        field: 'email',
        message: 'Invalid email format'
      });
    }
    return Result.success(email);
  }
  
  private validatePassword(password: string): Result<string, ValidationError> {
    if (!password || password.length < 8) {
      return Result.failure({
        field: 'password',
        message: 'Password must be at least 8 characters'
      });
    }
    return Result.success(password);
  }
  
  private checkEmailNotExists(email: string): Result<string, ValidationError> {
    // Check if email exists in our "database"
    const exists = Array.from(this.users.values()).some(user => user.email === email);
    
    if (exists) {
      return Result.failure({
        field: 'email',
        message: 'Email already registered'
      });
    }
    
    return Result.success(email);
  }
  
  private createUser(email: string, password: string): Result<User, string> {
    try {
      const user: User = {
        id: crypto.randomUUID(),
        email,
        passwordHash: this.hashPassword(password),
        createdAt: new Date()
      };
      
      // Save to our "database"
      this.users.set(user.id, user);
      
      return Result.success(user);
    } catch (error) {
      return Result.failure(`Failed to create user: ${error instanceof Error ? error.message : String(error)}`);
    }
  }
  
  private sendWelcomeEmail(user: User): Result<User, string> {
    try {
      // Simulate sending an email
      console.log(`Sending welcome email to ${user.email}`);
      
      return Result.success(user);
    } catch (error) {
      return Result.failure(`Failed to send welcome email: ${error instanceof Error ? error.message : String(error)}`);
    }
  }
  
  private hashPassword(password: string): string {
    // Simple password hashing (not for production)
    return crypto.createHash('sha256').update(password).digest('hex');
  }
}

// usage.ts
import { UserService } from './userService';
import { Result } from './result';

const userService = new UserService();

// Success case
const successResult = userService.registerUser('test@example.com', 'password123');
Result.match(
  successResult,
  user => console.log(`Registration successful: ${user.email}`),
  error => console.error('Registration failed:', error)
);

// Failure case - invalid email
const failureResult = userService.registerUser('invalid-email', 'password123');
Result.match(
  failureResult,
  user => console.log(`Registration successful: ${user.email}`),
  error => console.error('Registration failed:', error)
);
```

### Example 2: Order Processing Pipeline

```typescript
// orderTypes.ts
export interface OrderItem {
  productId: number;
  quantity: number;
  unitPrice?: number;
}

export interface OrderRequest {
  customerId: number;
  items: OrderItem[];
}

export interface PricedOrder {
  request: OrderRequest;
  totalAmount: number;
  discount: number;
  finalAmount: number;
}

export interface PaymentResult {
  order: PricedOrder;
  transactionId: string;
  status: 'APPROVED' | 'DECLINED';
}

export interface Order {
  orderId: string;
  customerId: number;
  items: OrderItem[];
  totalAmount: number;
  discount: number;
  finalAmount: number;
  paymentTransactionId: string;
  orderStatus: 'CONFIRMED' | 'CANCELLED';
  createdAt: Date;
}

export interface OrderError {
  code: string;
  message: string;
  timestamp: Date;
  traceId?: string;
}

// orderService.ts
import { Result } from './result';
import { 
  OrderRequest, 
  OrderItem, 
  PricedOrder, 
  PaymentResult, 
  Order, 
  OrderError 
} from './orderTypes';
import * as crypto from 'crypto';

export class OrderService {
  processOrder(request: OrderRequest): Result<Order, OrderError> {
    return this.validateOrderRequest(request)
      .flatMap(req => this.checkInventory(req))
      .flatMap(req => this.calculatePricing(req))
      .flatMap(pricedOrder => this.applyDiscounts(pricedOrder))
      .flatMap(pricedOrder => this.processPayment(pricedOrder))
      .flatMap(paymentResult => this.createOrder(paymentResult))
      .flatMap(order => this.updateInventory(order))
      .mapError(error => this.enrichError(error))
      .tap(order => this.logSuccess(order))
      .tapError(error => this.logError(error));
  }
  
  private validateOrderRequest(request: OrderRequest): Result<OrderRequest, OrderError> {
    if (!request.items || request.items.length === 0) {
      return Result.failure({
        code: 'EMPTY_ORDER',
        message: 'Order has no items',
        timestamp: new Date()
      });
    }
    return Result.success(request);
  }
  
  private checkInventory(request: OrderRequest): Result<OrderRequest, OrderError> {
    for (const item of request.items) {
      if (!this.hasStock(item)) {
        return Result.failure({
          code: 'OUT_OF_STOCK',
          message: `Item ${item.productId} is out of stock`,
          timestamp: new Date()
        });
      }
    }
    return Result.success(request);
  }
  
  private calculatePricing(request: OrderRequest): Result<PricedOrder, OrderError> {
    try {
      // Add prices to items first
      const itemsWithPrices = request.items.map(item => ({
        ...item,
        unitPrice: this.getPrice(item.productId)
      }));
      
      const totalAmount = itemsWithPrices.reduce(
        (sum, item) => sum + (item.quantity * (item.unitPrice || 0)), 
        0
      );
      
      const pricedOrder: PricedOrder = {
        request: {
          ...request,
          items: itemsWithPrices
        },
        totalAmount,
        discount: 0, // Will be calculated in applyDiscounts
        finalAmount: totalAmount // Will be updated in applyDiscounts
      };
      
      return Result.success(pricedOrder);
    } catch (error) {
      return Result.failure({
        code: 'PRICING_ERROR',
        message: `Failed to calculate price: ${error instanceof Error ? error.message : String(error)}`,
        timestamp: new Date()
      });
    }
  }
  
  private applyDiscounts(pricedOrder: PricedOrder): Result<PricedOrder, OrderError> {
    try {
      let discount = 0;
      
      // $10 off for orders over $100
      if (pricedOrder.totalAmount > 100) {
        discount += 10;
      }
      
      // Additional $5 off for loyal customers
      if (this.isLoyalCustomer(pricedOrder.request.customerId)) {
        discount += 5;
      }
      
      const updatedOrder = {
        ...pricedOrder,
        discount,
        finalAmount: pricedOrder.totalAmount - discount
      };
      
      return Result.success(updatedOrder);
    } catch (error) {
      return Result.failure({
        code: 'DISCOUNT_ERROR',
        message: `Failed to apply discounts: ${error instanceof Error ? error.message : String(error)}`,
        timestamp: new Date()
      });
    }
  }
  
  private processPayment(pricedOrder: PricedOrder): Result<PaymentResult, OrderError> {
    try {
      // Simulate payment processing
      const paymentResult: PaymentResult = {
        order: pricedOrder,
        transactionId: crypto.randomUUID(),
        status: 'APPROVED'
      };
      
      return Result.success(paymentResult);
    } catch (error) {
      return Result.failure({
        code: 'PAYMENT_ERROR',
        message: `Payment processing failed: ${error instanceof Error ? error.message : String(error)}`,
        timestamp: new Date()
      });
    }
  }
  
  private createOrder(paymentResult: PaymentResult): Result<Order, OrderError> {
    try {
      const order: Order = {
        orderId: crypto.randomUUID(),
        customerId: paymentResult.order.request.customerId,
        items: paymentResult.order.request.items,
        totalAmount: paymentResult.order.totalAmount,
        discount: paymentResult.order.discount,
        finalAmount: paymentResult.order.finalAmount,
        paymentTransactionId: paymentResult.transactionId,
        orderStatus: 'CONFIRMED',
        createdAt: new Date()
      };
      
      // Save to database would happen here
      
      return Result.success(order);
    } catch (error) {
      return Result.failure({
        code: 'ORDER_CREATION_ERROR',
        message: `Failed to create order: ${error instanceof Error ? error.message : String(error)}`,
        timestamp: new Date()
      });
    }
  }
  
  private updateInventory(order: Order): Result<Order, OrderError> {
    try {
      // Update inventory for each item
      for (const item of order.items) {
        this.updateProductStock(item.productId, item.quantity);
      }
      
      return Result.success(order);
    } catch (error) {
      // Log error but still return success (might want different behavior in real system)
      this.logInventoryUpdateError(error, order);
      return Result.success(order);
    }
  }
  
  private enrichError(error: OrderError): OrderError {
    return {
      ...error,
      traceId: crypto.randomUUID()
    };
  }
  
  // Helper methods
  private hasStock(item: OrderItem): boolean {
    // Inventory check simulation
    return true;
  }
  
  private getPrice(productId: number): number {
    // Price lookup simulation
    return 10 * productId;
  }
  
  private isLoyalCustomer(customerId: number): boolean {
    // Customer status check simulation
    return customerId % 2 === 0;
  }
  
  private updateProductStock(productId: number, quantity: number): void {
    // Update inventory simulation
    console.log(`Updating stock for product ${productId}, quantity: -${quantity}`);
  }
  
  private logSuccess(order: Order): void {
    console.log(`Order ${order.orderId} processed successfully`);
  }
  
  private logError(error: OrderError): void {
    console.error(`Error: [${error.code}] ${error.message}`);
  }
  
  private logInventoryUpdateError(error: unknown, order: Order): void {
    console.warn(`Warning: Inventory update failed for order ${order.orderId}: ${
      error instanceof Error ? error.message : String(error)
    }`);
  }
}

// usage.ts
import { OrderService } from './orderService';
import { Result } from './result';

const orderService = new OrderService();

// Success case
const result = orderService.processOrder({
  customerId: 1,
  items: [
    { productId: 101, quantity: 2 },
    { productId: 202, quantity: 1 }
  ]
});

Result.match(
  result,
  order => console.log(`Order processed: ${order.orderId}, Total: $${order.finalAmount}`),
  error => console.error(`Order processing failed: ${error.code} - ${error.message}`)
);
```

### Example 3: File Processing with Error Recovery

```typescript
// fileTypes.ts
export interface FileData {
  id: string;
  name: string;
  values: number[];
}

export interface TransformedData {
  id: string;
  name: string;
  processedDate: Date;
  values: number[];
}

export interface ProcessedFile {
  originalId: string;
  outputPath: string;
  processedDate: Date;
}

export interface FileError {
  code: string;
  message: string;
}

// fileProcessor.ts
import { Result } from './result';
import { FileData, TransformedData, ProcessedFile, FileError } from './fileTypes';
import * as fs from 'fs';
import * as path from 'path';
import * as crypto from 'crypto';

export class FileProcessor {
  processFile(filePath: string): Result<ProcessedFile, FileError> {
    return this.readFile(filePath)
      .flatMap(content => this.validateFormat(content))
      .flatMap(content => this.parseContent(content))
      .flatMap(data => this.transform(data))
      .flatMap(transformed => this.save(transformed))
      .recoverWith(error => {
        if (error.code === 'PARSE_ERROR') {
          return this.tryAlternativeParser(filePath);
        }
        return Result.failure(error);
      });
  }
  
  private readFile(filePath: string): Result<string, FileError> {
    try {
      if (!fs.existsSync(filePath)) {
        return Result.failure({
          code: 'FILE_NOT_FOUND',
          message: `File not found: ${filePath}`
        });
      }
      
      const content = fs.readFileSync(filePath, 'utf-8');
      return Result.success(content);
    } catch (error) {
      return Result.failure({
        code: 'READ_ERROR',
        message: `Cannot read file: ${error instanceof Error ? error.message : String(error)}`
      });
    }
  }
  
  private validateFormat(content: string): Result<string, FileError> {
    if (content.startsWith('<?xml') || 
        content.startsWith('{') || 
        content.startsWith('[')) {
      return Result.success(content);
    }
    
    return Result.failure({
      code: 'FORMAT_ERROR',
      message: 'Invalid file format'
    });
  }
  
  private parseContent(content: string): Result<FileData, FileError> {
    try {
      // Parsing logic depends on file type
      if (content.startsWith('<?xml')) {
        return this.parseXml(content);
      } else {
        return this.parseJson(content);
      }
    } catch (error) {
      return Result.failure({
        code: 'PARSE_ERROR',
        message: `Failed to parse content: ${error instanceof Error ? error.message : String(error)}`
      });
    }
  }
  
  private transform(data: FileData): Result<TransformedData, FileError> {
    try {
      // Apply business transformations
      const transformed: TransformedData = {
        id: data.id,
        name: data.name.toUpperCase(), // Simple transformation
        processedDate: new Date(),
        values: data.values.map(v => v * 2) // Double all values
      };
      
      return Result.success(transformed);
    } catch (error) {
      return Result.failure({
        code: 'TRANSFORM_ERROR',
        message: `Failed to transform data: ${error instanceof Error ? error.message : String(error)}`
      });
    }
  }
  
  private save(data: TransformedData): Result<ProcessedFile, FileError> {
    try {
      // Create output filename
      const outputPath = `processed_${data.id}_${Date.now()}.json`;
      
      // Simulate saving
      const jsonData = JSON.stringify(data, null, 2);
      // fs.writeFileSync(outputPath, jsonData); // Uncomment in real implementation
      
      const processedFile: ProcessedFile = {
        originalId: data.id,
        outputPath,
        processedDate: data.processedDate
      };
      
      return Result.success(processedFile);
    } catch (error) {
      return Result.failure({
        code: 'SAVE_ERROR',
        message: `Failed to save processed data: ${error instanceof Error ? error.message : String(error)}`
      });
    }
  }
  
  private tryAlternativeParser(filePath: string): Result<ProcessedFile, FileError> {
    console.log(`Trying alternative parser for ${filePath}`);
    
    return this.readFile(filePath)
      .flatMap(content => this.parseWithFallbackStrategy(content))
      .flatMap(data => this.transform(data))
      .flatMap(transformed => this.save(transformed));
  }
  
  private parseWithFallbackStrategy(content: string): Result<FileData, FileError> {
    try {
      // Implement a more forgiving parser
      // For this example, create fallback data
      const fallbackData: FileData = {
        id: crypto.randomUUID(),
        name: 'Recovered Data',
        values: [1, 2, 3]
      };
      
      return Result.success(fallbackData);
    } catch (error) {
      return Result.failure({
        code: 'FALLBACK_PARSE_ERROR',
        message: `Even fallback parsing failed: ${error instanceof Error ? error.message : String(error)}`
      });
    }
  }
  
  private parseXml(content: string): Result<FileData, FileError> {
    // XML parsing logic (simplified)
    return Result.success({
      id: crypto.randomUUID(),
      name: 'XML Data',
      values: [10, 20, 30]
    });
  }
  
  private parseJson(content: string): Result<FileData, FileError> {
    // JSON parsing logic (simplified)
    try {
      const parsed = JSON.parse(content);
      return Result.success({
        id: parsed.id || crypto.randomUUID(),
        name: parsed.name || 'JSON Data',
        values: Array.isArray(parsed.values) ? parsed.values : [100, 200, 300]
      });
    } catch {
      // If strict parsing fails, return mock data
      return Result.success({
        id: crypto.randomUUID(),
        name: 'JSON Data',
        values: [100, 200, 300]
      });
    }
  }
}

// usage.ts
import { FileProcessor } from './fileProcessor';
import { Result } from './result';

const processor = new FileProcessor();

// Success case (if the file exists)
const result = processor.processFile('./data.json');

Result.match(
  result,
  file => console.log(`File processed: ${file.outputPath}`),
  error => console.error(`Processing failed: ${error.code} - ${error.message}`)
);
```

## Advanced Patterns

### Combining Multiple Results

```typescript
// resultCombinators.ts
import { Result } from './result';

export class ResultCombinators {
  static combine2<T1, T2, E>(
    result1: Result<T1, E>,
    result2: Result<T2, E>
  ): Result<[T1, T2], E> {
    return Result.flatMap(result1, value1 => 
      Result.map(result2, value2 => [value1, value2] as [T1, T2])
    );
  }
  
  static combine3<T1, T2, T3, E>(
    result1: Result<T1, E>,
    result2: Result<T2, E>,
    result3: Result<T3, E>
  ): Result<[T1, T2, T3], E> {
    return Result.flatMap(
      ResultCombinators.combine2(result1, result2),
      ([value1, value2]) => Result.map(result3, value3 => [value1, value2, value3] as [T1, T2, T3])
    );
  }
  
  static sequence<T, E>(results: Result<T, E>[]): Result<T[], E> {
    const values: T[] = [];
    
    for (const result of results) {
      if (Result.isFailure(result)) {
        return result as unknown as Result<T[], E>;
      }
      values.push(result.value);
    }
    
    return Result.success(values);
  }
  
  static traverse<T, E>(results: Result<T, E>[]): Result<T[], E[]> {
    const successes: T[] = [];
    const failures: E[] = [];
    
    for (const result of results) {
      if (Result.isSuccess(result)) {
        successes.push(result.value);
      } else {
        failures.push(result.error);
      }
    }
    
    return failures.length > 0
      ? Result.failure(failures)
      : Result.success(successes);
  }
  
  // Process array of items with a fallible function
  static traverseArray<T, U, E>(
    items: T[], 
    fn: (item: T) => Result<U, E>
  ): Result<U[], E> {
    const results: Result<U, E>[] = items.map(fn);
    return ResultCombinators.sequence(results);
  }
}
```

### Async Operations with Promises

```typescript
// asyncResultExtensions.ts
import { Result } from './result';

export class AsyncResult {
  static fromPromise<T, E = Error>(promise: Promise<T>): Promise<Result<T, E>> {
    return promise
      .then(value => Result.success<T, E>(value))
      .catch(error => Result.failure<T, E>(error as E));
  }
  
  static mapAsync<T, E, U>(
    result: Result<T, E>, 
    fn: (value: T) => Promise<U>
  ): Promise<Result<U, E>> {
    return Result.isSuccess(result)
      ? fn(result.value).then(newValue => Result.success<U, E>(newValue))
      : Promise.resolve(result as unknown as Result<U, E>);
  }
  
  static flatMapAsync<T, E, U>(
    result: Result<T, E>, 
    fn: (value: T) => Promise<Result<U, E>>
  ): Promise<Result<U, E>> {
    return Result.isSuccess(result)
      ? fn(result.value)
      : Promise.resolve(result as unknown as Result<U, E>);
  }
  
  static sequenceAsync<T, E>(promises: Promise<Result<T, E>>[]): Promise<Result<T[], E>> {
    return Promise.all(promises).then(results => {
      return ResultCombinators.sequence(results);
    });
  }
}

// Async example
async function fetchUserData(userId: string): Promise<Result<User, string>> {
  return AsyncResult.fromPromise<User, string>(
    fetch(`https://api.example.com/users/${userId}`)
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.json();
      })
  );
}

async function processUserAsync(): Promise<void> {
  const result = await fetchUserData('123');
  
  Result.match(
    result,
    user => console.log(`User: ${user.name}`),
    error => console.error(`Error: ${error}`)
  );
}
```

## Best Practices

### 1. Use Specific Error Types

```typescript
// domainErrors.ts
export interface DomainError {
  readonly type: string;
  readonly message: string;
}

export interface ValidationError extends DomainError {
  readonly type: 'validation';
  readonly field: string;
}

export interface NotFoundError extends DomainError {
  readonly type: 'not_found';
  readonly entityType: string;
  readonly entityId: string;
}

export interface ServiceError extends DomainError {
  readonly type: 'service';
  readonly originalError?: Error;
}

export class DomainErrors {
  static validation(field: string, message: string): ValidationError {
    return {
      type: 'validation',
      field,
      message
    };
  }
  
  static notFound(entityType: string, entityId: string): NotFoundError {
    return {
      type: 'not_found',
      entityType,
      entityId,
      message: `${entityType} with ID '${entityId}' not found`
    };
  }
  
  static service(message: string, originalError?: Error): ServiceError {
    return {
      type: 'service',
      message,
      originalError
    };
  }
}
```

### 2. Create Domain-Specific Result Types

```typescript
// domainResult.ts
import { Result } from './result';
import { DomainError, ValidationError, NotFoundError, ServiceError, DomainErrors } from './domainErrors';

export type DomainResult<T> = Result<T, DomainError>;

export class DomainResultFactory {
  static success<T>(value: T): DomainResult<T> {
    return Result.success(value);
  }
  
  static failure<T>(error: DomainError): DomainResult<T> {
    return Result.failure(error);
  }
  
  static invalid<T>(field: string, message: string): DomainResult<T> {
    return Result.failure(DomainErrors.validation(field, message));
  }
  
  static notFound<T>(entityType: string, entityId: string): DomainResult<T> {
    return Result.failure(DomainErrors.notFound(entityType, entityId));
  }
  
  static fromError<T>(message: string, error?: Error): DomainResult<T> {
    return Result.failure(DomainErrors.service(message, error));
  }
  
  static ensure<T>(
    value: T | null | undefined, 
    entityType: string, 
    entityId: string
  ): DomainResult<T> {
    return value != null
      ? Result.success(value)
      : DomainResultFactory.notFound<T>(entityType, entityId);
  }
}
```

### 3. Use Extension Methods for Domain-Specific Operations

```typescript
// domainResultExtensions.ts
import { Result } from './result';
import { DomainResult } from './domainResult';
import { DomainError, ValidationError, DomainErrors } from './domainErrors';

export function ensureValid<T>(
  result: DomainResult<T>,
  predicate: (value: T) => boolean,
  field: string,
  message: string
): DomainResult<T> {
  return Result.flatMap(result, value => 
    predicate(value)
      ? Result.success(value)
      : Result.failure(DomainErrors.validation(field, message))
  );
}

export function ensureExists<T>(
  result: DomainResult<T>,
  predicate: (value: T) => boolean,
  entityType: string,
  entityId: string
): DomainResult<T> {
  return Result.flatMap(result, value => 
    predicate(value)
      ? Result.success(value)
      : Result.failure(DomainErrors.notFound(entityType, entityId))
  );
}

export function withErrorContext<T>(
  result: DomainResult<T>, 
  context: string
): DomainResult<T> {
  return Result.mapError(result, error => ({
    ...error,
    message: `${context}: ${error.message}`
  }));
}
```

### 4. Use Builder Pattern for Complex Flows

```typescript
// userRegistrationFlow.ts
import { Result } from './result';
import { DomainResult, DomainResultFactory } from './domainResult';
import { ensureValid } from './domainResultExtensions';
import { User } from './types';
import * as crypto from 'crypto';

export class UserRegistrationFlow {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly emailService: EmailService
  ) {}
  
  register(email: string, password: string): DomainResult<User> {
    // Start with a value object
    return DomainResultFactory.success({ email, password })
      .flatMap(this.validateRequest.bind(this))
      .flatMap(this.checkUserDoesNotExist.bind(this))
      .flatMap(this.createUser.bind(this))
      .flatMap(this.sendWelcomeEmail.bind(this));
  }
  
  private validateRequest(request: { email: string, password: string }): DomainResult<typeof request> {
    return DomainResultFactory.success(request)
      .flatMap(req => ensureValid(
        Result.success(req), 
        r => Boolean(r.email), 
        'email', 
        'Email is required'
      ))
      .flatMap(req => ensureValid(
        Result.success(req), 
        r => r.email.includes('@'), 
        'email', 
        'Invalid email format'
      ))
      .flatMap(req => ensureValid(
        Result.success(req), 
        r => Boolean(r.password), 
        'password', 
        'Password is required'
      ))
      .flatMap(req => ensureValid(
        Result.success(req), 
        r => r.password.length >= 8, 
        'password', 
        'Password must be at least 8 characters'
      ));
  }
  
  private checkUserDoesNotExist(request: { email: string, password: string }): DomainResult<typeof request> {
    return this.userRepository.existsByEmail(request.email)
      ? DomainResultFactory.invalid('email', 'Email already registered')
      : DomainResultFactory.success(request);
  }
  
  private createUser(request: { email: string, password: string }): DomainResult<User> {
    try {
      const user: User = {
        id: crypto.randomUUID(),
        email: request.email,
        passwordHash: this.hashPassword(request.password),
        createdAt: new Date()
      };
      
      this.userRepository.save(user);
      
      return DomainResultFactory.success(user);
    } catch (error) {
      return DomainResultFactory.fromError<User>(
        'Failed to create user', 
        error instanceof Error ? error : undefined
      );
    }
  }
  
  private sendWelcomeEmail(user: User): DomainResult<User> {
    try {
      this.emailService.sendWelcomeEmail(user.email);
      return DomainResultFactory.success(user);
    } catch (error) {
      // Continue with success but log error
      console.warn(`Failed to send welcome email: ${error instanceof Error ? error.message : String(error)}`);
      return DomainResultFactory.success(user);
    }
  }
  
  private hashPassword(password: string): string {
    return crypto.createHash('sha256').update(password).digest('hex');
  }
}

interface UserRepository {
  existsByEmail(email: string): boolean;
  save(user: User): void;
}

interface EmailService {
  sendWelcomeEmail(email: string): void;
}
```

### 5. Testing ROP Flows

```typescript
// userRegistrationFlow.test.ts
import { UserRegistrationFlow } from './userRegistrationFlow';
import { Result } from './result';
import { User } from './types';

// Mock implementations
class MockUserRepository {
  private users: User[] = [];
  
  existsByEmail(email: string): boolean {
    return this.users.some(u => u.email === email);
  }
  
  save(user: User): void {
    this.users.push(user);
  }
}

class MockEmailService {
  emailSent = false;
  
  sendWelcomeEmail(email: string): void {
    this.emailSent = true;
  }
}

describe('UserRegistrationFlow', () => {
  let userRepo: MockUserRepository;
  let emailService: MockEmailService;
  let flow: UserRegistrationFlow;
  
  beforeEach(() => {
    userRepo = new MockUserRepository();
    emailService = new MockEmailService();
    flow = new UserRegistrationFlow(userRepo, emailService);
  });
  
  test('should register valid user successfully', () => {
    // Arrange
    const email = 'test@example.com';
    const password = 'password123';
    
    // Act
    const result = flow.register(email, password);
    
    // Assert
    expect(Result.isSuccess(result)).toBe(true);
    if (Result.isSuccess(result)) {
      expect(result.value.email).toBe(email);
      expect(emailService.emailSent).toBe(true);
    }
  });
  
  test('should fail when email is invalid', () => {
    // Arrange
    const email = 'invalid-email';
    const password = 'password123';
    
    // Act
    const result = flow.register(email, password);
    
    // Assert
    expect(Result.isFailure(result)).toBe(true);
    if (Result.isFailure(result)) {
      expect(result.error.type).toBe('validation');
      expect((result.error as any).field).toBe('email');
    }
  });
  
  test('should fail when password is too short', () => {
    // Arrange
    const email = 'test@example.com';
    const password = 'short';
    
    // Act
    const result = flow.register(email, password);
    
    // Assert
    expect(Result.isFailure(result)).toBe(true);
    if (Result.isFailure(result)) {
      expect(result.error.type).toBe('validation');
      expect((result.error as any).field).toBe('password');
    }
  });
});
```