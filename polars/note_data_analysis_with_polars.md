# Data Analysis with Polars

Up and running with Polars
- Polars is a dataframe library with a Python API 
- A faster alternative to Pandas, Dask or Modin
- 3 key reasons you should use Polars
  - Runs much faster than other dataframe libraries
  - Optimises your code for you
  - Polars code is easier to read and write than other dataframe libraries
- With Polars, the query optimizer means your code expresses your query rather than numerical optimizations.
  - This means you get code that is easier to read and write
- **Eager mode** is running each code line by line independently
  - Eager model is the same as working with Pandas DataFrame
- **Lazy mode** is using only the columns/rows required to do a specific calculation, apply necessary optimization, 
- `collect()` means we want to evaluate our query, process the calculation, return the result
  - `collect()` returns DataFrame 
- **LayFrame** is a query plan.
- **naive plan** is without any optimization
- **Query plan** is read from bottom to top.
- Chaining or re-assigning?
  - Polars typically uses method chaining.
  - In terms of performance, both are the same.
  - Polars users believe chaining is easier to read.
- **Query optimization** that polars query optimizer finds are
  - Limiting the number of columns or rows that need to be read from a file
  - Reducing the number of rows needed for a query as early as possible
  - Avoiding carrying out the same calculation twice
  - Detecting when operations can be run in parallel
- `.explain()` shows optimized query plan
  - `print(... .explain())` is easier to read.

Data types
- Polars data types in a `Series` or `DataFrame` come from the **Apache Arrow** project
  - The data types in Pandas come from a mix of Numpy, Python, Apache Arrow, and some custom extension types
- **Apache Arrow** is an open source cross-language project to figure out the best way to represent tabular data in memory
  - Apache Arrow is a specification for how data should be represented in memory
  - Apache Arrow is a set of libraries in different languages that implement that specification
  - Polars use the implementation of the Arrow specification from the **Rust library Arrow2**.
- Why does Polars use Apache Arrow?
  - Arrow allows for sharing data without copying (**zero-copy**)
  - Faster vectorized calculations
  - Consistent representation of missing data
- With these, Polars can load and process data more quickly and with less memory usage

- `df.select("col")` turns it to one column dataframe.
- `to_series()` and `to_frame()` allows for shifting back and forth.

Convert between Polars and Numpy, convert between Polars and Pandas
- Choose the 64-bit floating point columns only for conversion to Numpy
```
df
.select(
    pl.col(pl.Float64)
)
.to_numpy()
```



