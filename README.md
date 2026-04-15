# Individual Reflection

## Differences between sequential and parallel execution

Sequential algorithms execute instructions one after another on a single CPU core, making them simple and deterministic. Parallel algorithms split work across multiple processes that run simultaneously, requiring careful coordination through queues and process management. The main trade-off is simplicity versus potential speedup.

## Performance behavior across dataset sizes

For 1,000 elements, sequential sorting (~0.001s) outperformed parallel (~0.05s) because creating processes took longer than the actual sorting. At 100,000 elements, parallel became slightly faster (~0.3s vs ~0.4s). At 1,000,000 elements, parallel sorting (~2.5s) significantly outperformed sequential (~8.0s), achieving about 3x speedup.

## Challenges encountered

Merging sorted chunks correctly was difficult - I initially lost data by overwriting results. Race conditions occurred in parallel search when multiple processes found the target simultaneously. I solved this using a Queue and terminating all processes once a result was found.

## Insights about overhead and synchronization

Process creation overhead is substantial - about 0.05-0.1 seconds per 4 processes. Communication through Queues adds latency. Parallelism only helps when computation time exceeds this overhead. For sorting, the O(n log n) complexity eventually outweighs overhead. For linear search O(n), parallelism helps but speedup is limited by sequential merging.

## When parallelism is beneficial vs unnecessary

Parallelism is beneficial for: datasets over 100,000 elements, computationally expensive operations, and embarrassingly parallel problems. It is unnecessary for: small datasets, simple O(n) operations on fast hardware, or when data cannot be easily partitioned.
