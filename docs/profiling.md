# Profiling Guide

This document explains how to use the profiling features in mmore to analyze performance and identify bottlenecks.

## Overview

The profiling system provides comprehensive performance monitoring capabilities including:
- **Function-level profiling** using cProfile
- **Timing decorators** for measuring execution time
- **Context managers** for profiling code blocks
- **Automatic profiling** integration in all main entry points

## Enabling Profiling

### Method 1: Environment Variables

Set environment variables to enable profiling:

```bash
export MMORE_PROFILING_ENABLED=true
export MMORE_PROFILING_OUTPUT_DIR=./profiling_output
export MMORE_PROFILING_SORT_BY=cumulative
export MMORE_PROFILING_MAX_RESULTS=50
```

Then run your command normally:
```bash
python -m mmore process --config-file examples/process/config.yaml
```

### Method 2: CLI Flag

Use the `--enable-profiling` flag:

```bash
python -m mmore process --config-file examples/process/config.yaml --enable-profiling --profiling-output-dir ./my_profiling_results
```

### Method 3: Programmatic Configuration

In your Python code:

```python
from mmore.profiling import configure_profiling

configure_profiling(
    enabled=True,
    output_dir="./profiling_output",
    sort_by="cumulative",
    max_results=50
)
```

## Profiling Output

Profiling results are saved as `.prof` files in the specified output directory. Each profiling session creates a file with a timestamp:

```
profiling_output/
  ├── process_1234567890.prof
  ├── index_1234567891.prof
  └── rag_1234567892.prof
```

## Analyzing Profiling Results

### Using pstats

```python
import pstats

stats = pstats.Stats('profiling_output/process_1234567890.prof')
stats.sort_stats('cumulative')
stats.print_stats(20)  # Show top 20 functions
```

### Using snakeviz (Visualization)

Install snakeviz:
```bash
pip install snakeviz
```

Visualize profiling results:
```bash
snakeviz profiling_output/process_1234567890.prof
```

This will open an interactive visualization in your browser.

## Using Profiling Decorators

### Function Decorator

```python
from mmore.profiling import profile_function

@profile_function()
def my_function():
    # Your code here
    pass
```

### Timing Decorator

```python
from mmore.profiling import time_function

@time_function()
def my_function():
    # Your code here
    pass
```

## Using Context Managers

### Profiling Context

```python
from mmore.profiling import profile_context

with profile_context("my_operation"):
    # Code to profile
    do_something()
```

### Timing Context

```python
from mmore.profiling import time_context

with time_context("my_operation"):
    # Code to time
    do_something()
```

## Using the Profiler Class

```python
from mmore.profiling import Profiler

profiler = Profiler(enabled=True, output_dir="./profiling_output")
profiler.start()

# Your code here

profiler.stop(name="my_session")
```

Or as a context manager:

```python
with Profiler(enabled=True, output_dir="./profiling_output") as profiler:
    # Your code here
    pass
```

## Environment Variables Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `MMORE_PROFILING_ENABLED` | `false` | Enable/disable profiling |
| `MMORE_PROFILING_OUTPUT_DIR` | `./profiling_output` | Directory for profiling results |
| `MMORE_PROFILING_SORT_BY` | `cumulative` | Sort results by (cumulative, time, calls, etc.) |
| `MMORE_PROFILING_MAX_RESULTS` | `50` | Maximum number of results to show |

## Sort Options

The `sort_by` parameter accepts:
- `cumulative` - Total time including sub-calls
- `time` - Internal time (excluding sub-calls)
- `calls` - Number of calls
- `ncalls` - Number of calls (with recursion)
- `tottime` - Total time
- `percall` - Time per call

## Best Practices

1. **Enable profiling only when needed** - Profiling adds overhead, so disable it in production unless necessary.

2. **Use specific output directories** - Organize profiling results by date or experiment.

3. **Analyze cumulative time first** - This shows where the most time is spent overall.

4. **Focus on hot paths** - Look for functions with high cumulative time that are called frequently.

5. **Compare before/after** - Use profiling to measure the impact of optimizations.

## Example Workflow

1. Enable profiling:
   ```bash
   export MMORE_PROFILING_ENABLED=true
   ```

2. Run your command:
   ```bash
   python -m mmore process --config-file examples/process/config.yaml
   ```

3. Analyze results:
   ```bash
   snakeviz profiling_output/process_*.prof
   ```

4. Identify bottlenecks and optimize

5. Re-run with profiling to verify improvements

