# Database Creator (C++)

A customizable, template-based **database creator** built on a **Binary Search Tree (BST)**.  
Define your own record “schema” (unique key + typed fields), then use BST operations to **insert**, **search**, **delete**, **traverse**, and **save/load** your database.

## Features

- **Template BST**: `bst<DataType>` supports primitive types and custom types (as long as comparison + stream ops exist).
- **Custom record type**: `custom_data<unique_data_type>` for “rows” with:
  - **Unique key** (`unique_parameter`) used for ordering + dedup (BST insert rejects duplicate keys)
  - Optional extra fields in **LDSCB** order:
    - **L**ong(s)
    - **D**ouble(s)
    - **S**tring(s)
    - **C**har(s)
    - **B**ool(s)
- **Core operations**
  - `insert`, `search`, `remove`
  - `remove_tree`, `remove_subtree`
- **Traversal / display**
  - `inorder`, `preorder`, `postorder`, `breadth_first`
  - `move()` interactive navigation (uses `getch()` / `<conio.h>`)
- **Persistence**
  - `save()` writes an **in-order** file and an **index file** (`.bin`) of offsets (enables fast load)
  - `load()` loads in **O(N)** when index file exists, otherwise loads in **O(N log N)** (or O(N) if it detects sorted data and rebuilds balanced)
  - `save_breadth_first()` / `load_breadth_first()` to preserve exact tree structure (uses `"NULL"` placeholders)

## Repository contents

- `bst.h` — Template declarations for `bst`, `custom_data`, and `other_data`
- `bst.cpp` — Implementations (BST ops, save/load logic, helpers)
- `DOCUMENTATION.MD` — Full documentation and usage guide
- `DEMO.md` / `readme.md` — Demo + short description

## Demo

- YouTube demo: https://www.youtube.com/watch?v=8JBbMGRZeOE  
  (also linked here: https://youtu.be/8JBbMGRZeOE?si=fnPthB-uIAYm9E4v)

## Quick start

This project is implemented in `bst.h` + `bst.cpp`. Include the header and compile the `.cpp`.

### Build (example)

```bash
g++ -std=c++11 bst.cpp -o database_creator
```

> Note: `move()` depends on `<conio.h>` (commonly available on Windows / some toolchains).  
> If you’re on Linux/macOS, you may need to remove/replace interactive parts that rely on `getch()`.

## Usage examples

### 1) Simple database with primitive types

```cpp
#include "bst.h"

int main() {
    bst<int> tree;

    tree.insert(50);
    tree.insert(30);
    tree.insert(70);

    if (tree.search(30)) {
        std::cout << "Found 30\n";
    }

    std::cout << "Inorder: ";
    tree.inorder();   // prints: 30 , 50 , 70 ,
    std::cout << "\n";

    tree.remove(30);

    std::cout << "Size: " << tree.get_size() << "\n";
}
```

### 2) Custom database with `custom_data`

Example schema: person record keyed by **ID (int)**, with:
- 1 long (age)
- 1 double (height)
- 4 strings (first name, last name, birth date, city)
- 1 char (hair color)
- 1 bool (life status)

```cpp
#include "bst.h"

enum { FIRST_NAME = 0, LAST_NAME, BIRTH_DATE, CITY };

int main() {
    custom_data<int> person(1, 1, 4, 1, 1);
    bst<custom_data<int>> database;

    person.at_unique() = 12345;
    person.at_longs(0) = 28;
    person.at_doubles(0) = 5.9;
    person.at_strings(FIRST_NAME) = "John";
    person.at_strings(LAST_NAME) = "Doe";
    person.at_strings(BIRTH_DATE) = "1995/05/15";
    person.at_strings(CITY) = "New York";
    person.at_chars(0) = 'B';
    person.at_bools(0) = 1;

    database.insert(person);

    if (database.search(person)) {
        auto found = database.access_traverser();
        std::cout << "Found: " << found.at_strings(FIRST_NAME) << "\n";
    }
}
```

## Save / Load behavior (important)

### `save()`
- Prompts for a file path and writes:
  1. **Data file**: in-order traversal, with records separated by `'\0'`
  2. **Index file**: same path + `.bin`, storing file offsets (`std::streampos`) for each record

### `load()`
- Prompts for a file path
- If the `.bin` index exists, loads using the index and builds a balanced tree in **O(N)**
- If the index does not exist:
  - checks if file is sorted
  - if sorted, loads into a vector then builds balanced tree in **O(N)**
  - otherwise inserts sequentially in **O(N log N)** average

## Requirements for custom types stored in `bst<T>`

For `bst<T>` to work correctly, your `T` should support:
- Comparisons used by BST ordering and dedup:
  - `operator<`, `operator>`, `operator==` (at minimum those used by insert/search/remove)
- Stream operators for persistence:
  - `operator<<` and `operator>>`
- A `to_string()` path is used in some helpers (the repo includes overloads and conversions)

`custom_data<unique_data_type>` already provides these behaviors based on its `unique_parameter`.

## Documentation

See `DOCUMENTATION.MD` for:
- Full architecture overview
- Class/function breakdown
- Performance table
- Civil records example design
