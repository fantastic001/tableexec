

# tableexec

`tableexec` is a command-line tool that reads tabular data from a spreadsheet or CSV file and executes a shell command for each row, with column values substituted into the command using Jinja-style `{{column_name}}` syntax. Optionally, it can join two sheets on specified keys and supports introspection of column names.

---

## Installation

Clone this repository or copy the script directly. Install the required dependencies with:

```bash
pip install pandas openpyxl xlrd odfpy
```

---

## Usage

```bash
tableexec --document path/to/file.xlsx --sheet Sheet1 --command "echo {{Name}} {{Age}}"
```

---

## Command-line Arguments

| Argument           | Description |
|--------------------|-------------|
| `--document`       | **(Required)** Path to the input file. Supported formats: `.csv`, `.xlsx`, `.ods`. |
| `--sheet`          | Name of the sheet to load (for Excel/ODS files). Required unless input is `.csv`. |
| `--command`        | **(Required)** The shell command to execute for each row. Use `{{column_name}}` to substitute values. |
| `--join-sheet`     | Name of another sheet in the same document to join with the primary one. |
| `--join-primary`   | Column name in the main sheet used as the join key. |
| `--join-foreign`   | Column name in the joined sheet used as the foreign key. |
| `--verbosity`      | Controls logging verbosity. Set to `1` to print the loaded table. |
| `--action`         | Either `exec` to run the command per row (default), or `columns` to print the column names only. |

---

## How It Works

1. Loads data from a `.csv`, `.xlsx`, or `.ods` file using `pandas`.
2. If `--join-sheet` is provided, performs a left join between the main and secondary sheet on the specified key columns.
3. Optionally prints the data if `--verbosity=1`.
4. Replaces placeholders in the command string with actual values from each row.
5. Executes the resulting command using the system shell.
6. If `--action=columns` is selected, it just prints the column names without executing anything.

---

## Example

### Spreadsheet (example.xlsx, Sheet1)

| Name   | Age |
|--------|-----|
| Alice  | 30  |
| Bob    | 25  |

### Command

```bash
tableexec --document example.xlsx --sheet Sheet1 --command "echo Hello, {{Name}}! You are {{Age}} years old."
```

### Output

```
Hello, Alice! You are 30 years old.
Hello, Bob! You are 25 years old.
```

---

## Sheet Join Example

### Sheet: `People`

| ID | Name  |
|----|-------|
| 1  | Alice |
| 2  | Bob   |

### Sheet: `Ages`

| PersonID | Age |
|----------|-----|
| 1        | 30  |
| 2        | 25  |

### Command

```bash
tableexec \
  --document people.xlsx \
  --sheet People \
  --join-sheet Ages \
  --join-primary ID \
  --join-foreign PersonID \
  --command "echo {{Name}} is {{Age}} years old."
```

### Output

```
Alice is 30 years old.
Bob is 25 years old.
```

---

## Debugging / Column Discovery

```bash
tableexec --document example.xlsx --sheet Sheet1 --action columns
```

Outputs:

```
Name
Age
```

---

## Notes

- Column names are stripped of whitespace.
- Placeholders like `{{ column }}` must match column names **exactly** after whitespace trimming.
- If an unsupported file extension is given, the script will raise an error.
- Joining works only with Excel/ODS files (not CSV).
