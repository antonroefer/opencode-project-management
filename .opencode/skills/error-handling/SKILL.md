---
name: error-handling
description: Robust error handling and resilience patterns. Includes try/catch strategies, error propagation, logging, monitoring, circuit breakers, retries, and graceful degradation. Use when implementing error handling, designing resilience patterns, adding retries, setting up circuit breakers, or when the user mentions error handling, exceptions, resilience, retries, circuit breaker, or graceful degradation.
---

# Error Handling Skill

Implement robust error handling and resilience patterns across languages.

## When to Use
- Designing error handling strategy
- Implementing retry logic
- Adding circuit breakers
- Setting up logging and monitoring
- Handling graceful degradation
- When the user mentions error handling, exceptions, or resilience

## Error Handling Patterns

### Try/Catch Fundamentals

**Python**
```python
try:
    result = risky_operation()
except ValueError as e:
    logger.error(f"Invalid value: {e}")
    raise  # Re-raise if needed
except ConnectionError as e:
    logger.error(f"Connection failed: {e}")
    return default_value
finally:
    cleanup()  # Always runs
```

**JavaScript/TypeScript**
```javascript
try {
    const result = await riskyOperation();
} catch (error) {
    if (error instanceof TypeError) {
        console.error('Type error:', error.message);
    } else {
        console.error('Unexpected error:', error);
    }
} finally {
    cleanup();
}
```

**Go (Explicit error returns)**
```go
result, err := riskyOperation()
if err != nil {
    log.Printf("Operation failed: %v", err)
    return nil, fmt.Errorf("risky operation: %w", err)
}
```

### Error Types/Categories

Define clear error types for different scenarios:

```python
class ValidationError(Exception):
    """Input validation failed"""
    pass

class NotFoundError(Exception):
    """Resource not found"""
    pass

class PermissionError(Exception):
    """Insufficient permissions"""
    pass
```

## Error Propagation

### Wrap and Propagate
```python
# Python - Chain exceptions
try:
    user = get_user(user_id)
except DatabaseError as e:
    raise ServiceError(f"Failed to get user {user_id}") from e
```

```javascript
// JavaScript - Async/await error propagation
async function getUser(id) {
    try {
        return await db.users.findById(id);
    } catch (error) {
        throw new ServiceError(`Failed to get user ${id}`, { cause: error });
    }
}
```

## Retry Patterns

### Basic Retry with Exponential Backoff
```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1, backoff=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            attempts = 0
            current_delay = delay
            while attempts < max_attempts:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    attempts += 1
                    if attempts == max_attempts:
                        raise
                    time.sleep(current_delay)
                    current_delay *= backoff
        return wrapper
    return decorator

@retry(max_attempts=3, delay=1)
def call_external_api():
    return requests.get('https://api.example.com/data')
```

### idempotency Key for Retries
```javascript
// Ensure retries don't cause duplicate side effects
async function createOrder(order, idempotencyKey) {
    return await fetch('https://api.example.com/orders', {
        method: 'POST',
        headers: {
            'Idempotency-Key': idempotencyKey
        },
        body: JSON.stringify(order)
    });
}
```

## Circuit Breaker Pattern

Prevent cascading failures by failing fast when a service is down.

```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            if self.state == 'OPEN':
                if time.time() - self.last_failure_time > self.recovery_timeout:
                    self.state = 'HALF_OPEN'
                else:
                    raise Exception("Circuit breaker is OPEN")

            try:
                result = func(*args, **kwargs)
                if self.state == 'HALF_OPEN':
                    self.state = 'CLOSED'
                    self.failure_count = 0
                return result
            except Exception as e:
                self.failure_count += 1
                self.last_failure_time = time.time()
                if self.failure_count >= self.failure_threshold:
                    self.state = 'OPEN'
                raise
        return wrapper
```

## Graceful Degradation

### Fallback Responses
```javascript
async function getUserProfile(userId) {
    try {
        // Try primary source
        return await cache.get(`user:${userId}`);
    } catch (cacheError) {
        try {
            // Fallback to database
            return await db.users.findById(userId);
        } catch (dbError) {
            // Return cached stale data or default
            return getDefaultProfile();
        }
    }
}
```

### Feature Flags for Degradation
```python
def process_payment(order):
    if feature_enabled('new_payment_service'):
        try:
            return new_payment_service.process(order)
        except Exception:
            pass  # Fall through to legacy
    return legacy_payment_service.process(order)
```

## Logging Best Practices

### Structured Logging
```python
import structlog

logger = structlog.get_logger()

logger.info("User login", user_id=123, ip="192.168.1.1", success=True)
logger.error("Payment failed", order_id=456, error="insufficient_funds", amount=100)
```

### Log Levels
- **DEBUG**: Detailed info for debugging
- **INFO**: Normal operational messages
- **WARNING**: Something unexpected but not critical
- **ERROR**: Error that needs attention
- **CRITICAL**: System-wide failure

### What to Log
- [ ] Request ID / Trace ID (for correlation)
- [ ] User ID (when applicable)
- [ ] Timestamp (ISO 8601)
- [ ] Error messages and stack traces
- [ ] Context (what was happening)

### What NOT to Log
- Passwords, API keys, tokens
- PII (Personally Identifiable Information) unless encrypted
- Excessive debug data in production

## Monitoring and Alerting

### Key Metrics
- **Error rate**: Percentage of requests resulting in errors
- **Response time**: p95, p99 latencies
- **Availability**: Uptime percentage
- **Saturation**: Resource usage

### Health Check Endpoint
```python
@app.route('/health')
def health_check():
    try:
        db.execute('SELECT 1')
        cache.ping()
        return {'status': 'healthy'}, 200
    except Exception as e:
        return {'status': 'unhealthy', 'error': str(e)}, 503
```

## Error Response Format (API)

Standardize error responses:

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested user was not found",
    "details": {
      "resource": "user",
      "id": "123"
    },
    "request_id": "req_abc123"
  }
}
```

HTTP Status Codes:
- **400**: Bad Request (client error)
- **401**: Unauthorized (not authenticated)
- **403**: Forbidden (not authorized)
- **404**: Not Found
- **409**: Conflict (e.g., duplicate)
- **429**: Too Many Requests (rate limit)
- **500**: Internal Server Error
- **503**: Service Unavailable

## Verification

After implementing error handling:
1. Test happy path (no errors)
2. Test error paths (force each error type)
3. Verify retries work (mock failures)
4. Test circuit breaker (trip it manually)
5. Check logs have required context
6. Verify error responses match format
