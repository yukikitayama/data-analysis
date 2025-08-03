# Pandas

Input and output
- `df.to_csv()` without filename, actually returns a string of all the data with line break.
- `pd.read_excel(file, sheet_name=["sheet1", "sheet2"])` gives a python dictionary with sheet name as key and value as pd.DataFrame.
- `pd.read_excel(... sheet_name=None)` actually reads all the sheets.

Visualization
- `plt.style.available` returns a list of color themes.
- `plt.style.use("theme")` sets a global setting

Settings and options
- `pd.describe_option("display")` shows all the explanations for each option, default and current value.
- `pd.set_option("display.max_columns", 2)` and `pd.options.display.max_columns = 2` are the same.
- `pd.reset_option("display.max_rows")` resets to the default setting.
- For floating decimal points, `pd.options.display.precision = 2`.