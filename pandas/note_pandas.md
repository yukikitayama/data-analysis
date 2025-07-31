# Pandas

Input and output
- `df.to_csv()` without filename, actually returns a string of all the data with line break.
- `pd.read_excel(file, sheet_name=["sheet1", "sheet2"])` gives a python dictionary with sheet name as key and value as pd.DataFrame.
- `pd.read_excel(... sheet_name=None)` actually reads all the sheets.

Visualization
- `plt.style.available` returns a list of color themes.
- `plt.style.use("theme")` sets a global setting
- 