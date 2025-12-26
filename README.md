# python-recutils

A Python implementation of [GNU recutils](https://www.gnu.org/software/recutils/), a set of tools and libraries to access human-editable, text-based databases called recfiles.

## Installation

```bash
# Clone the repository
git clone <repository-url>
cd python-recutils

# Install with uv
uv sync
```

## Usage

### Parsing Rec Files

```python
from python_recutils import parse, parse_file

# Parse from string
data = """
Name: Ada Lovelace
Age: 36

Name: Peter the Great
Age: 53
"""

record_sets = parse(data)
for rs in record_sets:
    for record in rs.records:
        print(record.get_field('Name'), record.get_field('Age'))

# Parse from file
with open('contacts.rec') as f:
    record_sets = parse_file(f)
```

### Using recsel

The `recsel` function mirrors the interface of the `recsel` command-line utility.

```python
from python_recutils import recsel, format_recsel_output

data = """
%rec: Book
%mandatory: Title

Title: GNU Emacs Manual
Author: Richard M. Stallman
Location: home

Title: The Colour of Magic
Author: Terry Pratchett
Location: loaned

Title: Mio Cid
Author: Anonymous
Location: home
"""

# Select all books
result = recsel(data, record_type='Book')
print(format_recsel_output(result))

# Select with expression (like recsel -e)
result = recsel(data, record_type='Book', expression="Location = 'home'")

# Select by position (like recsel -n)
result = recsel(data, record_type='Book', indexes='0,2')

# Print specific fields (like recsel -p)
result = recsel(data, record_type='Book', print_fields='Title,Author')

# Print values only (like recsel -P)
result = recsel(data, record_type='Book', print_values='Title')
# Returns: "GNU Emacs Manual\nThe Colour of Magic\nMio Cid"

# Count records (like recsel -c)
count = recsel(data, record_type='Book', count=True)
# Returns: 3

# Sort output (like recsel -S)
result = recsel(data, record_type='Book', sort='Title')

# Random selection (like recsel -m)
result = recsel(data, record_type='Book', random_count=2)
```

### recsel Options

| Option | CLI Equivalent | Description |
|--------|---------------|-------------|
| `record_type` | `-t TYPE` | Select records of this type |
| `indexes` | `-n INDEXES` | Select by position (e.g., "0,2,4-9") |
| `expression` | `-e EXPR` | Selection expression filter |
| `quick` | `-q STR` | Quick substring search |
| `random_count` | `-m NUM` | Select N random records |
| `print_fields` | `-p FIELDS` | Print fields with names |
| `print_values` | `-P FIELDS` | Print field values only |
| `print_row` | `-R FIELDS` | Print values space-separated |
| `count` | `-c` | Return count of matches |
| `include_descriptors` | `-d` | Include record descriptors |
| `collapse` | `-C` | Don't separate with blank lines |
| `case_insensitive` | `-i` | Case-insensitive matching |
| `sort` | `-S FIELDS` | Sort by fields |
| `group_by` | `-G FIELDS` | Group by fields |
| `uniq` | `-U` | Remove duplicate fields |

### Selection Expressions

Selection expressions filter records based on field values:

```python
# Numeric comparisons
recsel(data, expression="Age < 18")
recsel(data, expression="Score >= 90")

# String equality
recsel(data, expression="Name = 'John'")
recsel(data, expression="Status != 'inactive'")

# Regex matching
recsel(data, expression=r"Email ~ '\.org$'")

# Logical operators
recsel(data, expression="Age > 18 && Status = 'active'")
recsel(data, expression="Role = 'admin' || Role = 'superuser'")
recsel(data, expression="!Disabled")

# Field count
recsel(data, expression="#Email > 1")  # Records with multiple Email fields

# Field subscripts
recsel(data, expression="Email[0] ~ 'primary'")  # First Email field

# Implies operator
recsel(data, expression="Premium => Discount")  # If Premium, must have Discount

# Ternary conditional
recsel(data, expression="Age > 18 ? 1 : 0")

# String concatenation
recsel(data, expression="First & ' ' & Last = 'John Doe'")

# Arithmetic
recsel(data, expression="Price * Quantity > 100")
```

### Working with Records

```python
from python_recutils import Record, Field

# Create a record
record = Record(fields=[
    Field('Name', 'John Doe'),
    Field('Email', 'john@example.com'),
    Field('Email', 'john.doe@work.com'),  # Multiple fields with same name
])

# Access fields
name = record.get_field('Name')           # First value: 'John Doe'
emails = record.get_fields('Email')       # All values: ['john@example.com', 'john.doe@work.com']
count = record.get_field_count('Email')   # Count: 2
has_phone = record.has_field('Phone')     # False

# Convert to string (rec format)
print(str(record))
# Output:
# Name: John Doe
# Email: john@example.com
# Email: john.doe@work.com
```

### Evaluating Expressions Directly

```python
from python_recutils import evaluate_sex, Record, Field

record = Record(fields=[
    Field('Age', '25'),
    Field('Status', 'active'),
])

# Evaluate expression against a record
matches = evaluate_sex("Age > 18 && Status = 'active'", record)
# Returns: True
```

## Rec Format Overview

Recfiles are text files with a simple format:

```
# Comments start with #

# Record descriptor (optional, defines record type)
%rec: Contact
%mandatory: Name
%type: Age int

# Records are separated by blank lines
Name: Alice Smith
Email: alice@example.com
Age: 30

Name: Bob Jones
Email: bob@example.com
Email: bob.jones@work.com
Age: 25
Phone: +1 555-1234
```

Key concepts:
- **Fields**: `Name: Value` pairs
- **Records**: Groups of fields separated by blank lines
- **Multi-line values**: Use `+` continuation or `\` line continuation
- **Record descriptors**: Special records starting with `%rec:` that define record types
- **Comments**: Lines starting with `#`

## Running Tests

```bash
uv run pytest tests/ -v
```

## License

See LICENSE file for details.
