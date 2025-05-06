# OpenSearch Benchmark Ingestion Rate Control Guide

This guide provides simple instructions for controlling the ingestion rate in opensearch-benchmark to avoid "429 Too Many Requests" errors.

## Understanding 429 Errors

When running benchmarks, you may encounter "429 Too Many Requests" errors. This happens when the OpenSearch cluster cannot keep up with the ingestion rate. The default benchmark settings are often optimized for performance testing, not for production-like workloads.

## Simple Configuration Options

To control the ingestion rate, use these command-line options:

```bash
opensearch-benchmark \
  execute-test \
  --pipeline=benchmark-only \
  --workload=your_workload \
  --target-host=http://localhost:9200 \
  --workload-params '{
    "bulk_size": "500",
    "bulk_indexing_clients": "2",
    "retries": "5",
    "retry-wait-time": "2"
  }' \
  --results-file=./results.md
```

## Key Configuration Parameters

### Rate Control Parameters

- **bulk_size**: Number of documents per bulk request (default: often 5000+)
  - Smaller values (500-1000) generate less pressure
  
- **bulk_indexing_clients**: Number of concurrent clients (default: often 8 or more)
  - Fewer clients (1-3) mean lower concurrency and less pressure

### Error Handling Parameters

- **retries**: Maximum number of retries when 429 errors occur (default: 3)
  - Increase for more resilient workloads

- **retry-wait-time**: Initial wait time between retries in seconds (default: 1.0)
  - Higher values give the cluster more time to recover

## Implementation Details

Recent changes to the codebase have added exponential backoff with automatic retries when 429 errors are detected. The wait time between retries increases exponentially with each attempt.

## Monitoring

Watch the benchmark logs for messages like:
```
Received 429 Too Many Requests error. Retrying in 2.00 seconds (attempt 1/5)
```

This indicates the benchmark is throttling itself based on the cluster's capacity.


## Example Configurations

### Minimum Pressure (for small clusters)
```
--workload-params '{"bulk_size":"100","bulk_indexing_clients":"1","retries":"10"}'
```

### Medium Pressure (balance of speed/stability)
```
--workload-params '{"bulk_size":"500","bulk_indexing_clients":"2","retries":"5"}'
```

### Production-like (for larger clusters)
```
--workload-params '{"bulk_size":"1000","bulk_indexing_clients":"4","retries":"3"}'
``` 