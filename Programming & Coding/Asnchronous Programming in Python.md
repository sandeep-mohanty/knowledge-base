# Asynchronous and Parallel Programming in Python 12+

This tutorial covers modern approaches to concurrent programming in Python, including async/await patterns, parallel processing, and reactive programming with RxPython.

## Table of Contents
1. Understanding Concurrency vs Parallelism
2. Async/Await Programming
3. Parallel Processing with multiprocessing
4. Threading in Python
5. Reactive Programming with RxPython
6. Advanced RxPython Patterns
7. Combining RxPython with Async/Await
8. Real-World Use Cases
9. Best Practices and Performance Considerations

## 1. Understanding Concurrency vs Parallelism

Before diving into code, it's important to understand the distinction between concurrency and parallelism:

- **Concurrency**: Multiple tasks making progress, but not necessarily simultaneously. Tasks may be interleaved.
- **Parallelism**: Multiple tasks executing simultaneously on different CPU cores.

Python's Global Interpreter Lock (GIL) prevents true parallelism for CPU-bound tasks in threads, but we can achieve it through multiprocessing or use concurrency for I/O-bound tasks.

## 2. Async/Await Programming

Python's asyncio library provides a framework for writing concurrent code using async/await syntax.

### Basic Async Function

```python
import asyncio
import time

async def fetch_data(id: int) -> str:
    """Simulate an async I/O operation"""
    print(f"Starting to fetch data for ID {id}")
    await asyncio.sleep(2)  # Simulate network delay
    return f"Data for ID {id}"

async def main():
    # Run tasks concurrently
    start = time.time()
    
    # Method 1: Using gather
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    )
    
    print(f"Results: {results}")
    print(f"Time taken: {time.time() - start:.2f} seconds")

# Run the async function
asyncio.run(main())
```

### Working with Tasks

```python
import asyncio

async def process_item(item: int) -> int:
    await asyncio.sleep(1)
    return item * 2

async def main():
    # Create tasks manually
    tasks = []
    for i in range(5):
        task = asyncio.create_task(process_item(i))
        tasks.append(task)
    
    # Wait for all tasks to complete
    results = await asyncio.gather(*tasks)
    print(f"Processed results: {results}")
    
    # Alternative: Process as completed
    tasks = [asyncio.create_task(process_item(i)) for i in range(5)]
    for coro in asyncio.as_completed(tasks):
        result = await coro
        print(f"Got result: {result}")

asyncio.run(main())
```

### Async Context Managers and Iterators

```python
import asyncio
from typing import AsyncIterator

class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource")
        await asyncio.sleep(0.5)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        await asyncio.sleep(0.5)
    
    async def do_work(self):
        return "Work completed"

async def data_generator() -> AsyncIterator[int]:
    """Async generator example"""
    for i in range(5):
        await asyncio.sleep(0.5)
        yield i * i

async def main():
    # Using async context manager
    async with AsyncResource() as resource:
        result = await resource.do_work()
        print(result)
    
    # Using async iterator
    async for value in data_generator():
        print(f"Generated: {value}")

asyncio.run(main())
```

### Handling Timeouts and Cancellation

```python
import asyncio

async def long_running_task():
    try:
        await asyncio.sleep(10)
        return "Task completed"
    except asyncio.CancelledError:
        print("Task was cancelled")
        raise

async def main():
    # Timeout example
    try:
        result = await asyncio.wait_for(long_running_task(), timeout=2.0)
        print(result)
    except asyncio.TimeoutError:
        print("Task timed out")
    
    # Cancellation example
    task = asyncio.create_task(long_running_task())
    await asyncio.sleep(1)
    task.cancel()
    
    try:
        await task
    except asyncio.CancelledError:
        print("Task cancellation confirmed")

asyncio.run(main())
```

### Async Queues and Producer-Consumer Pattern

```python
import asyncio
import random

async def producer(queue: asyncio.Queue, producer_id: int):
    """Produce items and put them in the queue"""
    for i in range(5):
        item = f"Item {i} from producer {producer_id}"
        await queue.put(item)
        print(f"Producer {producer_id} produced: {item}")
        await asyncio.sleep(random.uniform(0.5, 1.5))

async def consumer(queue: asyncio.Queue, consumer_id: int):
    """Consume items from the queue"""
    while True:
        item = await queue.get()
        if item is None:  # Sentinel value
            break
        print(f"Consumer {consumer_id} consumed: {item}")
        await asyncio.sleep(random.uniform(0.5, 1.0))
        queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=10)
    
    # Create producers and consumers
    producers = [asyncio.create_task(producer(queue, i)) for i in range(2)]
    consumers = [asyncio.create_task(consumer(queue, i)) for i in range(3)]
    
    # Wait for producers to finish
    await asyncio.gather(*producers)
    
    # Send sentinel values to stop consumers
    for _ in consumers:
        await queue.put(None)
    
    # Wait for consumers to finish
    await asyncio.gather(*consumers)

asyncio.run(main())
```

## 3. Parallel Processing with multiprocessing

For CPU-bound tasks, use multiprocessing to achieve true parallelism.

### Basic Multiprocessing

```python
import multiprocessing as mp
import time

def cpu_bound_task(n: int) -> int:
    """Simulate CPU-intensive work"""
    total = 0
    for i in range(n * 1000000):
        total += i
    return total

def main():
    # Sequential execution
    start = time.time()
    results = [cpu_bound_task(i) for i in range(4)]
    print(f"Sequential time: {time.time() - start:.2f} seconds")
    
    # Parallel execution
    start = time.time()
    with mp.Pool(processes=4) as pool:
        results = pool.map(cpu_bound_task, range(4))
    print(f"Parallel time: {time.time() - start:.2f} seconds")
    print(f"Results: {results}")

if __name__ == "__main__":
    main()
```

### Sharing Data Between Processes

```python
import multiprocessing as mp
from multiprocessing import Queue, Value, Array
import time

def worker_with_queue(q: Queue, worker_id: int):
    """Worker that processes items from a queue"""
    while True:
        item = q.get()
        if item is None:  # Sentinel value
            break
        print(f"Worker {worker_id} processing {item}")
        time.sleep(0.5)
        q.task_done()

def increment_counter(counter: Value, lock: mp.Lock):
    """Safely increment a shared counter"""
    for _ in range(1000):
        with lock:
            counter.value += 1

def main():
    # Using Queue for communication
    queue = mp.Queue()
    processes = []
    
    # Start worker processes
    for i in range(3):
        p = mp.Process(target=worker_with_queue, args=(queue, i))
        p.start()
        processes.append(p)
    
    # Add work items
    for i in range(10):
        queue.put(f"Task {i}")
    
    # Add sentinel values to stop workers
    for _ in processes:
        queue.put(None)
    
    # Wait for all processes to complete
    for p in processes:
        p.join()
    
    # Using shared memory
    counter = mp.Value('i', 0)
    lock = mp.Lock()
    
    processes = []
    for _ in range(4):
        p = mp.Process(target=increment_counter, args=(counter, lock))
        p.start()
        processes.append(p)
    
    for p in processes:
        p.join()
    
    print(f"Final counter value: {counter.value}")

if __name__ == "__main__":
    main()
```

### Process Pool Executor

```python
from concurrent.futures import ProcessPoolExecutor, as_completed
import time

def process_data(data: tuple) -> dict:
    """Process a chunk of data"""
    id, value = data
    time.sleep(0.5)  # Simulate processing
    return {"id": id, "result": value ** 2}

def main():
    data = [(i, i * 10) for i in range(10)]
    
    with ProcessPoolExecutor(max_workers=4) as executor:
        # Submit all tasks
        future_to_data = {executor.submit(process_data, d): d for d in data}
        
        # Process results as they complete
        for future in as_completed(future_to_data):
            original_data = future_to_data[future]
            try:
                result = future.result()
                print(f"Processed {original_data}: {result}")
            except Exception as exc:
                print(f"Data {original_data} generated exception: {exc}")

if __name__ == "__main__":
    main()
```

## 4. Threading in Python

While limited by the GIL for CPU-bound tasks, threading is still useful for I/O-bound operations.

### Basic Threading

```python
import threading
import time
import requests
from queue import Queue

def fetch_url(url: str, results: Queue):
    """Fetch URL and store result in queue"""
    try:
        response = requests.get(url, timeout=5)
        results.put((url, response.status_code))
    except Exception as e:
        results.put((url, str(e)))

def main():
    urls = [
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/2",
        "https://httpbin.org/delay/1",
        "https://httpbin.org/status/200",
    ]
    
    results = Queue()
    threads = []
    
    start = time.time()
    
    # Create and start threads
    for url in urls:
        thread = threading.Thread(target=fetch_url, args=(url, results))
        thread.start()
        threads.append(thread)
    
    # Wait for all threads to complete
    for thread in threads:
        thread.join()
    
    # Collect results
    while not results.empty():
        url, status = results.get()
        print(f"{url}: {status}")
    
    print(f"Total time: {time.time() - start:.2f} seconds")

if __name__ == "__main__":
    main()
```

### Thread Pool Executor

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def io_task(task_id: int) -> str:
    """Simulate I/O-bound task"""
    time.sleep(1)  # Simulate I/O wait
    return f"Task {task_id} completed"

def main():
    with ThreadPoolExecutor(max_workers=5) as executor:
        # Submit tasks
        futures = [executor.submit(io_task, i) for i in range(10)]
        
        # Process results as they complete
        for future in as_completed(futures):
            result = future.result()
            print(result)

if __name__ == "__main__":
    main()
```

## 5. Reactive Programming with RxPython

RxPython brings reactive programming paradigms to Python, allowing you to work with asynchronous data streams using functional programming concepts.

### Installation

```bash
pip install reactivex
```

### Basic Observables and Operators

```python
import reactivex as rx
from reactivex import operators as ops
import time

def main():
    # Create a simple observable
    source = rx.from_([1, 2, 3, 4, 5])
    
    # Subscribe with observer functions
    source.subscribe(
        on_next=lambda x: print(f"Received: {x}"),
        on_error=lambda e: print(f"Error: {e}"),
        on_completed=lambda: print("Completed")
    )
    
    # Using operators for functional transformations
    print("\nWith operators:")
    source.pipe(
        ops.map(lambda x: x * 2),
        ops.filter(lambda x: x > 5),
        ops.scan(lambda acc, x: acc + x, 0)  # Running sum
    ).subscribe(on_next=lambda x: print(f"Result: {x}"))

if __name__ == "__main__":
    main()
```

### Creating Custom Observables

```python
import reactivex as rx
from reactivex import operators as ops
from typing import Optional
import time

def create_temperature_sensor() -> rx.Observable:
    """Create an observable that emits temperature readings"""
    def subscribe(observer, scheduler=None):
        import random
        
        def emit_temperature():
            for i in range(10):
                temperature = 20 + random.uniform(-5, 5)
                observer.on_next(temperature)
                time.sleep(0.5)
            observer.on_completed()
        
        # Run in a separate thread
        import threading
        thread = threading.Thread(target=emit_temperature)
        thread.start()
        
        # Return a disposable
        return rx.Disposable(lambda: thread.join())
    
    return rx.create(subscribe)

def main():
    # Use the custom observable
    temperature_stream = create_temperature_sensor()
    
    temperature_stream.pipe(
        ops.map(lambda temp: round(temp, 2)),
        ops.filter(lambda temp: temp > 22),
        ops.take(5)
    ).subscribe(
        on_next=lambda temp: print(f"High temperature alert: {temp}°C"),
        on_completed=lambda: print("Monitoring completed")
    )
    
    # Keep main thread alive
    time.sleep(6)

if __name__ == "__main__":
    main()
```

### Hot vs Cold Observables

```python
import reactivex as rx
from reactivex import operators as ops
from reactivex.subject import Subject
import time
import threading

def demonstrate_cold_observable():
    """Cold observable - each subscriber gets its own sequence"""
    print("Cold Observable Demo:")
    
    cold = rx.interval(1.0).pipe(
        ops.take(5),
        ops.map(lambda x: f"Cold value: {x}")
    )
    
    # First subscriber
    print("First subscriber starting...")
    sub1 = cold.subscribe(on_next=lambda x: print(f"Sub1: {x}"))
    
    time.sleep(2)
    
    # Second subscriber starts later
    print("Second subscriber starting...")
    sub2 = cold.subscribe(on_next=lambda x: print(f"Sub2: {x}"))
    
    # Keep main thread alive
    time.sleep(6)
    sub1.dispose()
    sub2.dispose()

def demonstrate_hot_observable():
    """Hot observable - all subscribers share the same sequence"""
    print("\nHot Observable Demo:")
    
    # Create a subject (which is hot)
    subject = Subject()
    
    # Add a subscriber
    print("First subscriber starting...")
    sub1 = subject.subscribe(on_next=lambda x: print(f"Sub1: {x}"))
    
    # Emit values
    for i in range(3):
        subject.on_next(f"Hot value: {i}")
        time.sleep(1)
    
    # Add another subscriber (misses first 3 values)
    print("Second subscriber starting...")
    sub2 = subject.subscribe(on_next=lambda x: print(f"Sub2: {x}"))
    
    # Emit more values
    for i in range(3, 5):
        subject.on_next(f"Hot value: {i}")
        time.sleep(1)
    
    subject.on_completed()

def main():
    demonstrate_cold_observable()
    demonstrate_hot_observable()

if __name__ == "__main__":
    main()
```

## 6. Advanced RxPython Patterns

### Error Handling and Retry

```python
import reactivex as rx
from reactivex import operators as ops
import random

def unreliable_operation(x):
    """Operation that might fail"""
    if random.random() < 0.3:  # 30% chance of failure
        raise Exception(f"Failed processing {x}")
    return x * 2

def main():
    source = rx.from_([1, 2, 3, 4, 5])
    
    # Retry on error
    source.pipe(
        ops.map(lambda x: unreliable_operation(x)),
        ops.retry(3),  # Retry up to 3 times
        ops.catch(lambda err, src: rx.just(f"Error handled: {err}"))
    ).subscribe(
        on_next=lambda x: print(f"Result: {x}"),
        on_error=lambda e: print(f"Unhandled error: {e}")
    )

if __name__ == "__main__":
    main()
```

### Combining Observables

```python
import reactivex as rx
from reactivex import operators as ops
import time

def main():
    # Source observables
    integers = rx.interval(1.0).pipe(
        ops.take(5),
        ops.map(lambda i: f"Integer: {i}")
    )
    
    letters = rx.interval(1.5).pipe(
        ops.take(4),
        ops.map(lambda i: f"Letter: {chr(65 + i)}")  # A, B, C, D
    )
    
    # 1. Merge - combine items as they arrive
    print("Merge Example:")
    rx.merge(integers, letters).subscribe(
        on_next=lambda x: print(f"Merged: {x}"),
        on_completed=lambda: print("Merge completed")
    )
    time.sleep(5)
    print()
    
    # 2. Concat - run sequences one after another
    print("Concat Example:")
    rx.concat(integers, letters).subscribe(
        on_next=lambda x: print(f"Concat: {x}"),
        on_completed=lambda: print("Concat completed")
    )
    time.sleep(10)
    print()
    
    # 3. Combine latest - combine latest values from each source
    print("Combine Latest Example:")
    numbers = rx.interval(1.0).pipe(ops.take(5))
    characters = rx.interval(1.5).pipe(
        ops.take(4),
        ops.map(lambda i: chr(65 + i))  # A, B, C, D
    )
    
    rx.combine_latest(numbers, characters).subscribe(
        on_next=lambda x: print(f"Combined: {x}"),
        on_completed=lambda: print("Combine latest completed")
    )
    time.sleep(6)
    print()
    
    # 4. Zip - pair items from sources by index
    print("Zip Example:")
    rx.zip(numbers, characters).subscribe(
        on_next=lambda x: print(f"Zipped: {x}"),
        on_completed=lambda: print("Zip completed")
    )
    time.sleep(6)

if __name__ == "__main__":
    main()
```

### Handling Backpressure

```python
import reactivex as rx
from reactivex import operators as ops
import time
import random

def slow_consumer(x):
    """Simulate a slow consumer"""
    processing_time = random.uniform(0.5, 1.5)
    time.sleep(processing_time)
    return x

def main():
    # Fast producer
    fast_producer = rx.interval(0.1).pipe(
        ops.take(20)
    )
    
    # Without backpressure handling
    print("Without backpressure handling:")
    fast_producer.pipe(
        ops.map(lambda x: f"Processing item {x}"),
        ops.map(slow_consumer)
    ).subscribe(
        on_next=lambda x: print(f"Consumed: {x}"),
        on_completed=lambda: print("Consumption completed")
    )
    time.sleep(5)
    print()
    
    # With backpressure handling using buffer
    print("With buffer backpressure handling:")
    fast_producer.pipe(
        ops.buffer_with_count(5),  # Buffer items in groups of 5
        ops.map(lambda batch: f"Batch processing {batch}"),
        ops.map(slow_consumer)
    ).subscribe(
        on_next=lambda x: print(f"Consumed: {x}"),
        on_completed=lambda: print("Consumption completed")
    )
    time.sleep(5)
    print()
    
    # With sample backpressure handling
    print("With sample backpressure handling:")
    fast_producer.pipe(
        ops.sample(1.0),  # Take one item per second
        ops.map(lambda x: f"Sampled item {x}"),
        ops.map(slow_consumer)
    ).subscribe(
        on_next=lambda x: print(f"Consumed: {x}"),
        on_completed=lambda: print("Consumption completed")
    )
    time.sleep(5)

if __name__ == "__main__":
    main()
```

## 7. Combining RxPython with Async/Await

RxPython can be integrated with asyncio for more powerful concurrency patterns.

### Using RxPython with asyncio

```python
import reactivex as rx
from reactivex import operators as ops
import asyncio
import time

# Create an observable from an async generator
async def async_data_source():
    for i in range(5):
        await asyncio.sleep(0.5)
        yield i

def observe_async_generator(async_gen):
    def subscribe(observer, scheduler=None):
        async def process_async_gen():
            try:
                async for item in async_gen:
                    observer.on_next(item)
                observer.on_completed()
            except Exception as e:
                observer.on_error(e)
        
        # Create and run task
        loop = asyncio.get_event_loop()
        task = loop.create_task(process_async_gen())
        
        # Return disposable to cancel task
        return rx.Disposable(lambda: task.cancel())
    
    return rx.create(subscribe)

async def process_with_rx():
    # Create an observable from an async generator
    source = observe_async_generator(async_data_source())
    
    # Process with RxPython
    results = []
    done_event = asyncio.Event()
    
    source.pipe(
        ops.map(lambda x: x * 2),
        ops.filter(lambda x: x > 2)
    ).subscribe(
        on_next=lambda x: results.append(x),
        on_completed=lambda: done_event.set()
    )
    
    # Wait for observable to complete
    await done_event.wait()
    return results

async def main():
    results = await process_with_rx()
    print(f"Processed results: {results}")

if __name__ == "__main__":
    asyncio.run(main())
```

### Converting Between RxPython and asyncio

```python
import reactivex as rx
from reactivex import operators as ops
import asyncio
import time

# Convert an observable to an async iterator
def to_async_iterator(observable):
    queue = asyncio.Queue(maxsize=1)
    done = asyncio.Event()
    error = None
    
    def on_next(item):
        asyncio.create_task(queue.put(item))
    
    def on_completed():
        done.set()
    
    def on_error(err):
        nonlocal error
        error = err
        done.set()
    
    # Subscribe to the observable
    disposable = observable.subscribe(
        on_next=on_next,
        on_completed=on_completed,
        on_error=on_error
    )
    
    async def aiter():
        try:
            while not done.is_set() or not queue.empty():
                try:
                    yield await queue.get()
                except asyncio.CancelledError:
                    disposable.dispose()
                    raise
            if error:
                raise error
        finally:
            disposable.dispose()
    
    return aiter()

async def main():
    # Create an observable
    source = rx.interval(0.5).pipe(
        ops.take(5),
        ops.map(lambda x: f"Item {x}")
    )
    
    # Convert to async iterator
    async_iter = to_async_iterator(source)
    
    # Use in async code
    print("Using observable as async iterator:")
    async for item in async_iter:
        print(f"Received: {item}")
        await asyncio.sleep(0.2)
    
    print("Done processing")

if __name__ == "__main__":
    asyncio.run(main())
```

## 8. Real-World Use Cases

### Event-Driven Microservice

```python
import reactivex as rx
from reactivex import operators as ops
import json
import time
import threading
from typing import Dict, Any

# Simulate message queue or event bus
class EventBus:
    def __init__(self):
        self.subjects = {}
    
    def get_subject(self, topic):
        if topic not in self.subjects:
            self.subjects[topic] = rx.Subject()
        return self.subjects[topic]
    
    def publish(self, topic, data):
        subject = self.get_subject(topic)
        subject.on_next(data)
    
    def subscribe(self, topic):
        return self.get_subject(topic)

# Microservice components
class UserService:
    def __init__(self, event_bus: EventBus):
        self.event_bus = event_bus
        self.user_db = {}  # Mock database
        
        # Subscribe to user creation events
        self.event_bus.subscribe('user.register').pipe(
            ops.map(self.create_user)
        ).subscribe(
            on_next=lambda user: self.event_bus.publish('user.created', user)
        )
    
    def create_user(self, data: Dict[str, Any]) -> Dict[str, Any]:
        user_id = len(self.user_db) + 1
        user = {
            'id': user_id,
            'name': data['name'],
            'email': data['email'],
            'created_at': time.time()
        }
        self.user_db[user_id] = user
        print(f"UserService: Created user {user['name']}")
        return user

class NotificationService:
    def __init__(self, event_bus: EventBus):
        self.event_bus = event_bus
        
        # Subscribe to user creation events
        self.event_bus.subscribe('user.created').subscribe(
            on_next=self.send_welcome_email
        )
    
    def send_welcome_email(self, user: Dict[str, Any]):
        print(f"NotificationService: Sending welcome email to {user['email']}")
        # Simulate email sending
        time.sleep(0.5)
        self.event_bus.publish('email.sent', {
            'user_id': user['id'],
            'template': 'welcome',
            'sent_at': time.time()
        })

class AnalyticsService:
    def __init__(self, event_bus: EventBus):
        self.event_bus = event_bus
        self.stats = {'users_created': 0, 'emails_sent': 0}
        
        # Track multiple event types
        rx.merge(
            self.event_bus.subscribe('user.created').pipe(
                ops.map(lambda _: ('users_created', 1))
            ),
            self.event_bus.subscribe('email.sent').pipe(
                ops.map(lambda _: ('emails_sent', 1))
            )
        ).pipe(
            ops.scan(lambda acc, value: self.update_stats(acc, value), self.stats)
        ).subscribe(
            on_next=lambda stats: print(f"AnalyticsService: Stats update - {stats}")
        )
    
    def update_stats(self, stats, update):
        key, increment = update
        stats[key] += increment
        return stats

def main():
    # Create event bus and services
    event_bus = EventBus()
    user_service = UserService(event_bus)
    notification_service = NotificationService(event_bus)
    analytics_service = AnalyticsService(event_bus)
    
    # Register some users
    for i in range(3):
        event_bus.publish('user.register', {
            'name': f"User {i+1}",
            'email': f"user{i+1}@example.com"
        })
        time.sleep(1)

if __name__ == "__main__":
    main()
```

### Real-time Data Processing Pipeline

```python
import reactivex as rx
from reactivex import operators as ops
import time
import random
import json
from datetime import datetime

# Simulate a data source
def sensor_data_source():
    def subscribe(observer, scheduler=None):
        def emit():
            for _ in range(20):
                # Generate random sensor reading
                data = {
                    'timestamp': datetime.now().isoformat(),
                    'device_id': f"sensor-{random.randint(1, 5)}",
                    'temperature': round(15 + 10 * random.random(), 2),
                    'humidity': round(40 + 30 * random.random(), 2),
                    'pressure': round(1000 + 50 * random.random(), 2)
                }
                observer.on_next(data)
                time.sleep(0.5)
            observer.on_completed()
        
        thread = threading.Thread(target=emit)
        thread.start()
        
        return rx.Disposable(lambda: None)  # No cleanup needed
    
    return rx.create(subscribe)

# Processing steps
def parse_and_validate(data):
    # Validate data format
    required_fields = ['timestamp', 'device_id', 'temperature', 'humidity']
    if not all(field in data for field in required_fields):
        raise ValueError("Missing required fields")
    
    # Validate temperature range
    if not (0 <= data['temperature'] <= 50):
        raise ValueError(f"Temperature out of range: {data['temperature']}")
    
    return data

def enrich_data(data):
    # Add calculated fields
    data['heat_index'] = round(data['temperature'] * 1.1 + data['humidity'] * 0.2, 2)
    
    # Add region based on device ID
    device_num = int(data['device_id'].split('-')[1])
    data['region'] = 'north' if device_num <= 2 else 'south'
    
    return data

def detect_anomalies(data):
    # Detect high temperature
    if data['temperature'] > 22:
        data['alerts'] = ['HIGH_TEMPERATURE']
    else:
        data['alerts'] = []
    return data

# Output sinks
def save_to_database(data):
    # Simulate database operation
    print(f"Saving to database: {data['device_id']}, temp: {data['temperature']}")
    return data

def send_alerts(data):
    # Send alerts if any
    if data['alerts']:
        alerts = ', '.join(data['alerts'])
        print(f"ALERT! {alerts} detected from {data['device_id']} in {data['region']}")
    return data

def main():
    # Create the processing pipeline
    sensor_data_source().pipe(
        # Process each reading
        ops.map(lambda x: json.dumps(x)),  # Simulate serialization
        ops.map(lambda x: json.loads(x)),   # Simulate deserialization
        ops.map(parse_and_validate),
        ops.map(enrich_data),
        ops.map(detect_anomalies),
        
        # Split the pipeline for different outputs
        ops.do_action(save_to_database),
        ops.do_action(send_alerts),
        
        # Aggregate data by region
        ops.group_by(lambda x: x['region']),
        ops.flat_map(lambda group: group.pipe(
            ops.window_with_time(5.0),
            ops.flat_map(lambda window: window.pipe(
                ops.to_list(),
                ops.map(lambda batch: {
                    'region': group.key,
                    'count': len(batch),
                    'avg_temp': round(sum(x['temperature'] for x in batch) / len(batch), 2)
                })
            ))
        ))
    ).subscribe(
        on_next=lambda x: print(f"Region summary: {x}"),
        on_error=lambda err: print(f"Error in pipeline: {err}"),
        on_completed=lambda: print("Processing completed")
    )
    
    # Let the simulation run
    time.sleep(15)

if __name__ == "__main__":
    main()
```

## 9. Best Practices and Performance Considerations

### Choosing the Right Approach

- **Use asyncio when**:
  - You have I/O-bound tasks
  - You need fine-grained control over concurrency
  - Your code performs a lot of network operations

- **Use threading when**:
  - You have I/O-bound tasks
  - You're working with libraries that aren't asyncio-compatible
  - You need simple background operations

- **Use multiprocessing when**:
  - You have CPU-bound tasks
  - You need to bypass the GIL
  - You need to utilize multiple cores

- **Use RxPython when**:
  - You have event streams or asynchronous data sources
  - You need complex transformations on data streams
  - You're building event-driven architectures

### Performance Tips

```python
import asyncio
import time
import multiprocessing as mp
from concurrent.futures import ProcessPoolExecutor
import reactivex as rx
from reactivex import operators as ops

# CPU-bound function for testing
def cpu_intensive(n):
    """Calculate sum of squares from 0 to n-1"""
    return sum(i * i for i in range(n))

# Demonstrate correct approach for CPU-bound tasks
async def main():
    start = time.time()
    
    # BAD: Running CPU-intensive work with asyncio
    print("Running CPU-intensive tasks with asyncio:")
    tasks = []
    for i in range(4):
        # This blocks the event loop!
        tasks.append(asyncio.create_task(
            asyncio.to_thread(cpu_intensive, 10000000)
        ))
    await asyncio.gather(*tasks)
    print(f"Asyncio time: {time.time() - start:.2f} seconds\n")
    
    # GOOD: Running CPU-intensive work with ProcessPoolExecutor
    start = time.time()
    print("Running CPU-intensive tasks with ProcessPoolExecutor:")
    with ProcessPoolExecutor() as executor:
        loop = asyncio.get_event_loop()
        tasks = [
            loop.run_in_executor(executor, cpu_intensive, 10000000)
            for _ in range(4)
        ]
        await asyncio.gather(*tasks)
    print(f"ProcessPoolExecutor time: {time.time() - start:.2f} seconds\n")
    
    # GOOD: Properly using asyncio for I/O-bound tasks
    start = time.time()
    print("Running I/O-bound tasks with asyncio:")
    
    async def io_task(i):
        await asyncio.sleep(1)  # Simulate I/O wait
        return f"Task {i} done"
    
    results = await asyncio.gather(*[io_task(i) for i in range(10)])
    print(f"Results: {len(results)} tasks")
    print(f"I/O tasks time: {time.time() - start:.2f} seconds\n")
    
    # RxPython with proper concurrency
    start = time.time()
    print("Running mixed workload with RxPython:")
    
    # Create observable of work items
    source = rx.from_(range(10))
    
    # Process I/O-bound and CPU-bound differently
    results_io = []
    results_cpu = []
    
    # I/O pipeline with concurrency
    io_pipeline = source.pipe(
        ops.filter(lambda x: x % 2 == 0),  # Even numbers
        ops.flat_map(lambda i: rx.just(i).pipe(
            ops.do_action(lambda x: time.sleep(1)),  # Simulate I/O
            ops.map(lambda x: f"I/O {x} done")
        )),
    )
    
    # CPU pipeline with process pool
    def cpu_pipeline_factory(source):
        def subscribe(observer, scheduler=None):
            # Use process pool for CPU-bound tasks
            with ProcessPoolExecutor() as executor:
                # Filter odd numbers
                items = [i for i in range(10) if i % 2 == 1]
                
                # Process in parallel
                futures = [executor.submit(cpu_intensive, 5000000) for _ in items]
                
                # Collect results
                for i, future in enumerate(futures):
                    observer.on_next(f"CPU {items[i]} done")
                
                observer.on_completed()
        
        return rx.create(subscribe)
    
    # Combine pipelines
    rx.merge(io_pipeline, cpu_pipeline_factory(source)).subscribe(
        on_next=lambda x: print(f"Result: {x}"),
        on_completed=lambda: print(f"RxPython mixed time: {time.time() - start:.2f} seconds")
    )
    
    # Wait for completion
    time.sleep(2)  # Add enough time for processes to complete

if __name__ == "__main__":
    asyncio.run(main())
```

### Best Practices Summary

1. **Use the right tool for the job**:
   - Match your concurrency approach to your workload type
   - Don't use asyncio for CPU-bound work without offloading

2. **Avoid blocking the event loop**:
   - Use `asyncio.to_thread()` or executors for blocking operations
   - Keep CPU-intensive work out of the asyncio event loop

3. **Handle errors appropriately**:
   - Always use try/except in async code
   - With RxPython, use `catch` and proper error handlers

4. **Limit concurrency**:
   - Use semaphores to control maximum concurrent operations
   - Don't create unlimited tasks or processes

5. **Clean up resources**:
   - Always dispose RxPython subscriptions
   - Cancel tasks when they're no longer needed
   - Use context managers for proper resource cleanup

6. **Test and benchmark**:
   - Different approaches have different overhead
   - Measure actual performance to choose the best approach