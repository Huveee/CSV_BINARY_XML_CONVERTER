# CSV → Binary → XML Converter

A C program that takes bank customer records through three steps. It reads them from a CSV file into an array of structs, writes that array to a binary file, then turns the binary file into an XML document and checks the XML against an XSD schema. The XML work is done with [libxml2](https://gitlab.gnome.org/GNOME/libxml2). The program is meant to be built and run on Linux.

## The three steps

1. **CSV → binary** (`records.csv` → `records.dat`): splits each CSV row into its 12 fields, stores them in a `Customer` struct (empty fields stay empty) and writes the struct array to the binary file with `fwrite`.
2. **Binary → XML** (`records.dat` → `records.xml`): reads the structs back and builds an XML tree with libxml2. Each customer becomes a `<row id="…">` element with a `customer_info` block and a `bank_account_info` block. The `total_balance_available` element gets two attributes: `currency_unit`, and `bigEnd_Version`, which is the balance with its byte order converted from little-endian to big-endian.
3. **XSD validation**: checks `records.xml` against `control.xsd` and prints whether it validates.

## Record fields

| Field | Rule in `control.xsd` |
|-------|-----------------------|
| `name`, `surname` | required, up to 21 / 31 characters |
| `gender` | `M` or `F` |
| `occupancy` | up to 31 characters |
| `level_of_education` | `PhD`, `MSc`, `BSc`, `HS` or `PS` |
| `email` | must end in `@gmail.com`, `@hotmail.com` or `@yahoo.com` |
| `bank_account_number` | `1234-5678901` format |
| `IBAN` | `TR12 3456 7890 1234 5678 90` format |
| `account_type` | `deposit`, `drawing` or `daily_deposit` |
| `currency_unit` | `€`, `₺` or `$` |
| `total_balance_available` | the balance |
| `available_for_loan` | `😊` or `☹` |

## Files

| File | What it is |
|------|------------|
| `main.c` | The program |
| `records.csv` | Sample input with 50 made-up customers |
| `records.dat`, `records.xml` | Outputs of steps 1 and 2 |
| `control.xsd` | The XSD schema used in step 3 |
| `myConverter` | A compiled Linux x86-64 binary |
| `libxml/` | A copy of the libxml2 header files |

## Building

On Ubuntu or Debian:

```bash
sudo apt install build-essential libxml2-dev
gcc main.c -o myConverter -I/usr/include/libxml2 -lxml2 -lm
```

## Running it

Run the steps in order, from the repository folder:

```bash
./myConverter records.csv records.dat 1   # step 1: CSV -> binary
./myConverter records.dat records.xml 2   # step 2: binary -> XML
./myConverter records.xml records.xsd 3   # step 3: validate the XML
```

The program compares its arguments with these exact strings, so keep in mind:

- Start it as `./myConverter` from the folder that contains `records.csv`. Every step opens `records.csv`, even steps 2 and 3.
- Use exactly the file names and step numbers above. Other arguments do nothing, and running it with fewer than three arguments crashes.
- In step 3 the second argument must be `records.xsd`, but the schema it actually loads is `control.xsd`.
- The code is written for the sample data. It has room for 50 customers and always writes 50 structs to the binary file.
