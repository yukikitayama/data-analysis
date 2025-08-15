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

Query optimization
- Polars passes the naive query plan to its query optimizer, and runs the optimized query plan when `.collect()` method is called on df.
- When the `.fetch()` method is called on a LazyFrame `df`, polars run the optimized query plan and fetches a limited number of rows from the data source.

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
- Polar is generally faster than Numpy. If you need to feed numpy object to machine learning models, do data processing with Polar most of the time, and right before feeding to ML model, convert Polar dataframes to Numpy.
- In some casses, we can convert Series to Numpy without copying (**zero-copy**).
  - Zero-copy is only possible if there's no `Null` or `NaN` values, and pd.Series
  - Let an object refrece back to the data, without copying it.
  - Data wont't copied, itr wil refecemcer 
- With zero-copy conversion, the Numpy array is read-only and you cannot overridel
- `.to_pandas()` converts to **Numpy-baked** Pandas dataframe.
- Conversion of Pandas required you PyArrow
- Convert to PyArrow-backed Pandas dataframe
  - Use pandas without copying 
  - If there is a function you want from Pandas, apply the function and revert back to Polars. Only works in eager mode.
- Should not call `pd.DataFrame(pl_df)`, because may have bugs.

Visualization
- The built-in visualization library of Polars is **Altair**.
- You should limit the number of columns in the data passed to the visualization libraries, so that plots use less memory.

Filter rows
- Polars doesn't have an explicit index, but it has an implicit integer row number index.
- We cannot pass a list of Boolean values or a Boolean Series in `[]`, while Pandas can.
- Main use of `[]` to select rows is when inspecting data in interactive mode.
- **`filter` is the primary way to filter rows in Polars**.
- Boolean series can be passed to `filter()`
- `with_columns()` and `filter()` creates a condition column and filter rows based on that
```python
(
    df
    .with_columns(
        less_than_30_boolean = pl.col("Age") < 30
    )
    .filter(
        pl.col("less_than_30_boolean")
    )
)
```
- `~` is negation
- `partition_by("col", as_dict=True)` can avoid expensive large dataframe
  - returns a dictionary where a key is a value of the column and a value is subset dataframe
- In query plan output, `SELECTION: None` when no filters have been applied.
- Optimized query plan run when LazyFrame is evaluated with `collect` or `fetch`
- Polars will apply the filter on some column as the CSV is being read.
- In the optimized plan, only the selected rows of CSV that meet the filter conditions are read into a DataFrame. This is memory efficient.
- In lazy mode, if we pass multiple `filter` calls, the query optimizer combines these into a single condition inside `SELECTION`.

Selecting columns
- `[]` rules
  - If we pass numeric value, we get rows
  - If we pass string, we get columns
  - If we pass a tuple like `[numeric, string]`, we get rows and columns
- Cannot create a new column by `df["new_col"] = 1`
- To create a column, we need to use `with_columns` method.
- The output of `select()` is always `DataFrame` rather than `Series` even if one column is selected.
  - Use `to_series()`
- `select(["col1", "col2"])` to select multiple columns
- **`select` can be used in lazy mode, but `[]` indenxing can be used in eager mode only**.
- **Expression in `select` can be optimized in lazy mode by the query optimizer**.
- **Multiple expressions in `select` can be run in parallel**.
- For these reasons, use `select` rather than `[]` indexing in general.
- Using `select` with `scan_csv()` in lazy mode, Polars only loads necessary columns into memory when reading the CSV and changes the `PROJECT` part of the optimized query plan.
- Reducing the number of columns reduces time and memory usage.
- Polars has 2 ways for selecting multiple colums
  - Expression API with `pl.col` and `pl.all`
  - Selectors API with polars selectors `polars.selectors`
    - For less verbose and consistency
- `select(pl.exclude([]))` is shorthand for `select(pl.all().exclude([]))`
- We can select columns with regex if the regex starts with `^` and ends with `$`
  - `select("^something$")` or `select(pl.col("^something$))`
- `select(pl.col(pl.Utf8))` selects columns by dtypes, here string
- `pl.NUMERIC_DTYPES` to select all numeric dtypes
- `.select(cs.numeric())` can select columns of all integers and floating points
- `cs.by_name("col1", "col2", ...)` can select columns by name
- Simpler alternatives to regex
  - `contains`
  - `starts_with`
  - `end_with`
  - `matches` doesn't need `^` and `$` that we need for the expression API
- `cs.string() - cs.starts_with("T")` select all string columns other than any columns beginning with T
- `with_columns` allows us to transform the existing column
- **The difference between `select` and `with_columns` is that `select` returns only the columns listed while `with_columns` returns the entire dataframe
  - Both can transform columns using expressions.
- `with_columns` and `alias` can add a new column
- **Polars doesn't support add a new column using `[]` indexing**.
- `with_columns` returns all of the columns, but `select` returns a subset of the columns.
- `plt.lit` (literal function) and `alias` can add a new column with a constant value
- `pl.when` syntax
```
pl.when(Boolean expression)
.then(value if true)
.otherwise(value if false)
.alias(new column name)
```
- For a condition on multiple other columns, `() & ()` in `pl.when`
- A new column based on `if-elif-else` condition
```
pl.when(boolean expression)
.then(value if true)
.when(boolean expression)
.then(value if true)
.otherwise(value if false)
.alias(new column name)
```
- Polars can use a **fast track algorithm** if it knows the data in a column is **sorted**.
  - If we want the max value on a sorted column, we just take the last non-null value
- If we transform a column with a sorting operation, Polars will update `flags`
- `df.rename({"current_name": "new_name"})`
- `df.drop(["col1", "col2"])`
- `df.select([new list of columns])` reorders columns
- Use `pipe` method to transform dataframe by function
```
def uppercase_all_strings(df):
    return (
        df
        .with_columns(
            pl.col(pl.Utf8).str.to_uppercase()
        )
    )

(
    df
    .pipe(uppercase_all_strings)
)
```
- The transformations in `pipe` are passed to the query optimizer in lazy mode.
- To use function arguments in `pipe`, a `DataFrame` needs to be the first argument, the rest of the arguments follows it, only a `DataFrame` is output
```
def _multiply_floats(df, multiplication_factor):
    return df.select(
        pl.col(pl.Float64)
    ) * multiplication_factor


(
    df
    .pipe(
        _multiply_floats,
        multiplication_factor=3
    )
    .head(3)
)
```
- `df.rows()` and `df.iter_rows()` can iterate rows
  - When we call `rows`, the **entire** `DataFrame` is materialized as a list of tuple
  - When we call `iter_rows`, Polars materializes each row as a Python tuple when we iterate over it rather than materializing the whole `DataFrame` at the outset.
    - Can reduce memory usage
- `df.rows(named=True)` and `df.iter_rows(named=True)` returns a list of dictionary, so that column name can access by key

Missing values
- Missing values in Polars are `null` for all data types.
- `df.null_count()` stores metadata
  - Polars keeps track of this always, so it's a cheap operation regardless of the size of columns
- Use `filter()` and `is_not_null()` to filter null values.
- Use `drop_nulls()` to remove missing values.
- Replace all the missing values with 0
```
(
    df
    .with_columns(
        pl.all().fill_null(0).name.suffix("_new")
    )
)
```
- Replace missing values with strategy
```
(
    df
    .with_columns(
        pl.all().fill_null(strategy="forward").name.suffix("_new")
    )
)
```
- Replacing missing values by group needs **window function** `over()`.
  - Group by operation
```
(
    df
    .with_columns(
        pl.col("col1").fill_null(strategy="forward").over("group").name.suffix("_filled")
    )
)
```
- `pl.coalesce(["col1", "col2", ...])` can use a sequence of columns to fill missing values

Settings
- `pl.Config.set_tbl_rows(4)` will always print 4 rows

Data types and precision
- **Polars creates integer and float columns as 64-bit by default**.
- `shrink_dtype` allows us to detect if the actual data in a column can fit in a lower precision dtype and cast the column to that dtype
- **Working at a lower precision may be more effective for some analysis**.
- `df.estimated_size(unit="b")` returns the estimated size in bytes
- Casting 64-bit representation dataframe to 32-bit representation can halve memory usage
- Math calculation will be faster with 32-bit data than 64-bit data.
- **Moving to a lower precision than 32-bit does not always lead to faster performance**. 
  - Many CPUs do not have native support for 8-bit and 16-bit operations and so they emulate it with 32-bit operations.

