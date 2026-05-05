---
name: performance-profiling
description: Language-agnostic performance analysis. Covers profiling tools, benchmarking, memory analysis, bottleneck identification, and optimization strategies across different runtimes. Use when analyzing performance issues, profiling applications, identifying bottlenecks, optimizing code, or when the user mentions profiling, benchmarking, performance, optimization, slow queries, or memory leaks.
---

# Performance Profiling Skill

Identify and resolve performance bottlenecks across languages and runtimes.

## When to Use
- Application is running slowly
- Need to optimize code
- Identifying memory leaks
- Benchmarking changes
- Load testing and capacity planning
- When the user mentions performance, profiling, or optimization

## Profiling Strategies

### 1. CPU Profiling
Identify where code spends time.

**Python (cProfile)**
```python
import cProfile
cProfile.run('my_function()', 'output.prof')

# Analyze with pstats
import pstats
p = pstats.Stats('output.prof')
p.sort_stats('cumulative').print_stats(10)
```

**Node.js (built-in)**
```bash
# Run with profiling
node --prof app.js

# Process log file
node --prof-process isolate-*.log > processed.txt
```

**Go (pprof)**
```go
import _ "net/http/pprof"

// Visit http://localhost:6060/debug/pprof/
// Or use go tool:
go tool pprof http://localhost:6060/debug/pprof/profile
```

### 2. Memory Profiling
Find memory leaks and excessive allocations.

**Python (memory_profiler)**
```python
from memory_profiler import profile

@profile
def my_func():
    x = [1] * 1000000
    return x
```

**Node.js (heapdump)**
```javascript
const heapdump = require('heapdump');
heapdump.writeSnapshot('./heap-' + Date.now() + '.heapsnapshot');
// Analyze in Chrome DevTools
```

**Go (pprof heap)**
```bash
go tool pprof http://localhost:6060/debug/pprof/heap
```

### 3. Database Query Profiling

**PostgreSQL**
```sql
-- Enable query logging
SET log_min_duration_statement = 100;  -- Log queries > 100ms

-- Use EXPLAIN ANALYZE
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Check slow query log
SELECT query, mean_time, calls FROM pg_stat_statements ORDER BY mean_time DESC;
```

**MongoDB**
```javascript
// Enable profiling
db.setProfilingLevel(1, { slowms: 100 })

// Check profile data
db.system.profile.find().sort({ ts: -1 }).limit(10)
```

## Benchmarking

### Micro-benchmarks
```python
# Python (timeit)
import timeit
timeit.timeit('"-".join(str(n) for n in range(100))', number=10000)
```

```javascript
// JavaScript (console.time)
console.time('myFunction');
myFunction();
console.timeEnd('myFunction');
```

```go
// Go (testing.B)
func BenchmarkMyFunction(b *testing.B) {
    for i := 0; i < b.N; i++ {
        myFunction()
    }
}
// Run: go test -bench=.
```

## Identifying Bottlenecks

### The USE Method (Utilization, Saturation, Errors)
| Resource | Utilization | Saturation | Errors |
|----------|-------------|------------|--------|
| CPU | `top`, `htop` | Load average | Errors in logs |
| Memory | `free -h` | Swap usage | OOM kills |
| Disk | `iostat` | Wait queue | I/O errors |
| Network | `iftop` | Packet drops | Timeouts |

### Common Bottlenecks
1. **Database N+1 queries** - Use eager loading, batch queries
2. **Unoptimized loops** - Move invariant code out, use built-in functions
3. **Synchronous I/O** - Use async/await, promises
4. **Large JSON responses** - Paginate, compress, use sparse fieldsets
5. **Missing indexes** - Add database indexes on frequently queried columns
6. **Memory leaks** - Remove event listeners, clear caches, avoid closures

## Optimization Checklist

### Code Level
- [ ] Algorithm complexity reduced (O(n²) → O(n log n))
- [ ] Unnecessary computations moved out of loops
- [ ] String concatenation uses efficient method (join vs +)
- [ ] Database queries batched or eliminated (N+1 problem)
- [ ] Caching implemented for expensive operations

### Database Level
- [ ] Queries use indexes (check EXPLAIN)
- [ ] SELECT only needed columns (avoid SELECT *)
- [ ] Pagination implemented for large result sets
- [ ] Connections pooled properly
- [ ] Unused indexes removed

### Infrastructure Level
- [ ] CDN configured for static assets
- [ ] Gzip/Brotli compression enabled
- [ ] Caching headers set correctly
- [ ] Load balancer configured
- [ ] Auto-scaling set up for traffic spikes

## Load Testing

### Tools
- **Apache Bench**: `ab -n 1000 -c 10 http://localhost:8000/`
- **wrk**: `wrk -t12 -c400 -d30s http://localhost:8000/`
- **JMeter**: GUI-based, good for complex scenarios
- **k6**: Scriptable, modern load testing

### Load Test Workflow
```bash
# 1. Start with baseline
wrk -t2 -c10 -d10s http://localhost:8000/api/users

# 2. Increase load gradually
wrk -t4 -c50 -d30s http://localhost:8000/api/users

# 3. Find breaking point
wrk -t8 -c200 -d60s http://localhost:8000/api/users

# 4. Monitor system resources during test
# (in another terminal)
top
iostat -x 1
```

## Performance Monitoring

### Key Metrics to Track
- **Response time**: p50, p95, p99 latencies
- **Throughput**: Requests per second
- **Error rate**: Percentage of failed requests
- **Saturation**: CPU, memory, disk usage

### Tools
- **Prometheus + Grafana**: Metrics collection and visualization
- **Jaeger/Zipkin**: Distributed tracing
- **New Relic/Datadog**: APM (Application Performance Monitoring)

## Verification

After performance optimization:
1. Run benchmarks before and after (compare numbers)
2. Profile to ensure bottleneck is resolved
3. Run load test to verify under pressure
4. Monitor for regressions in production
5. Document optimization and results
