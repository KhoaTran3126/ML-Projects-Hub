# Polars-Tutorial
A repository for tutorials on Polars.

## Customization
<pre>
  <code>
import polars as pl
import polars.selectors as pls
    
def format_pl():
  """FLOAT DISPLAY FORMATTING"""
  pl.Config.set_fmt_float("mixed")
  """STRING FORMATTING"""
  pl.Config.set_fmt_str_lengths(50)
  """TABLE FORMATTING"""
  pl.Config.set_tbl_rows(8)
  pl.Config.set_tbl_cols(30)
  pl.Config.set_tbl_width_chars(200)
  pl.Config.set_tbl_cell_alignment("RIGHT")
  pl.Config.set_tbl_hide_dtype_separator(True)
  pl.Config.set_tbl_hide_column_data_types(True)

format_pl()
  </code>
</pre>

## Null Count
<pre>
  <code>
  data.select(
    pl.all().is_null().sum()
  ).unpivot(
    value_name="NA_count"
  )
  </code>
</pre>

<pre>
  <code>
train_NA_counts = train.select(
    pl.all().is_null().sum()
).unpivot(
    value_name="train_NA_count",
    variable_name="feature"
).sort(
    by="train_NA_count",
    descending=True
).filter(
    pl.col("train_NA_count")>0
).with_columns(
    (100*pl.col("train_NA_count")/len(train)).alias("train_NA_pct")
)

dtypes_df = pl.DataFrame({
    "feature": list(train.schema.keys()), 
    "dtype": list(train.schema.values())
})

train_NA_counts = train_NA_counts.join(dtypes_df, on="feature", how="left")
display(train_NA_counts)
  </code>
</pre>

## Unique Count
<pre>
  <code>
  data.select(
    pl.all().n_unique().sum()
  ).unpivot(
    value_name="unique_count"
  )
  </code>
</pre>

## Selecting Numeric/Categorical columns
<pre>
  <code>
data.select(pls.numeric())
data.select(pl.col(pl.Utf8))
  </code>
</pre>

## Selecting Repetitive Feature Names with Regex
<pre>
  <code>
// ^    is text's head anchor
// .*   means "select >= 0 of any characters in between
// $    is text's tail anchor

// Exclude any features starting with "day_" Example(day_1, day_ofWeek, day_last)
data.select(pl.all().exclude("^day_.*$"))

// Selects col_1, any columns with ending of "_high" and "_low"
df.select(pl.col("col_1", "^.*_high$", "^.*_low$"))
  </code>
</pre>

## Basic Data transformations
<pre>
  <code>
data.with_columns(
  np.log1p(pl.col(A)).alias("log_A"),
  (np.exp(pl.col(B))+1).alias("add1_exp_B")
)
  </code>
</pre>

## Fast Row-Wise Operations Across Multiple Columns
<pre>
  <code>
"""Product over rows of columns"""
data.with_columns(
    pl.fold(
    acc = pl.lit(1), ## accumulator (initial/identity value of operation)
    function = lambda acc,x: acc*x, ## Multiplies accumulated result with next value
    exprs = pl.col("^amount_.*$")   ## Operates over any columns starting with "amount_"
    ).alias("prod_amounts")
)

"""Mean over rows of columns"""
data.with_columns(
    (pl.fold(
    acc = pl.lit(0), ## accumulator (initial/identity value of operation)
    function = lambda acc,x: acc+x, ## Adds accumulated result with next value
    exprs = pl.col("^temp_.*$")     ## Operates over any columns starting with "temp_"
    ) / col_count).alias("mean_temp")
)
    
"""Magnitude over rows of columns"""
data.with_columns(
    (
    pl.fold(
    acc = pl.lit(0), ## accumulator (initial/identity value of operation)
    function = lambda acc,x: acc+x**2, ## Adds accumulated result with next squared value
    exprs = pl.col("^temp_.*$")     ## Operates over any columns starting with "temp_"
    ).sqrt()
    ).alias("magnitude_temp")
)
  </code>
</pre>

## Custom Transformations
<pre>
  <code>
  data.with_columns(
    pl.when(pl.col(A)==... & ... | ...)
    .then(...)
    .otherwise(...)
    .alias(...)
  )
  </code>
</pre>

## Aggregated statistics
<pre>
  <code>
  data.group_by(
    pl.col(A, ...)
  ).agg(
    pl.col(B).operation().alias("operation_name")
  )
  </code>
</pre>



