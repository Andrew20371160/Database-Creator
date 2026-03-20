# Database-Creator

This library provides:

- A **generic Binary Search Tree (BST)** implementation (`bst<T>`) with:
  - insertion, search, deletion (node / subtree / whole tree)
  - multiple traversals (DFS orders + breadth-first)
  - structure-aware equality
  - conversion to string + stream operators
  - an interactive traverser “cursor” (`move()`)
- **File persistence** for BSTs:
  - **in-order save/load** with an accompanying **binary index file** (`.bin`) to enable **O(N)** rebuild for sorted persistence
  - **breadth-first save/load** to preserve *shape* exactly
- A schema/abstraction layer for “database records”:
  - `custom_data<UniqueT>`: a record with a **unique key** and typed “columns”
  - `other_data`: storage for typed arrays (long/double/string/char/bool)
- Helper utilities:
  - safe input helpers (`getInput`)
  - multiple `to_string` overloads (including vectors, bst, and custom_data)
  - internal dynamic-array helpers used by the abstraction layer

---

## 1) Core Topic: Binary Search Tree (BST)

### 1.1 Data Structure: `node<DataType>`

Defined in `bst.h`.

#### `template<typename DataType> struct node`
**Purpose:** A tree node storing one `DataType` value and links to children and parent.

**Fields**
- `DataType data` — value stored in this node
- `node<DataType>* left` — left child
- `node<DataType>* right` — right child
- `node<DataType>* parent` — parent pointer (used for navigation and deletion fixes)

---

### 1.2 Class: `template<typename DataType> class bst`

Defined in `bst.h` and implemented in `bst.cpp`.

#### Overview / Requirements for `DataType`
To be stored in `bst<DataType>`, `DataType` must support:

- Comparisons: `operator<`, `operator>`, `operator==`
- Stream I/O for persistence and printing:
  - `operator<<` (for saving and printing)
  - `operator>>` (for loading and parsing from files)

For correct file persistence behavior, `operator>>` must parse the same format that `operator<<` writes.

---

### 1.3 Construction, Copying, Destruction

#### `bst::bst()`
**Meaning:** Creates an empty BST.

**Behavior**
- `root = NULL`
- `traverser = NULL`
- `size = 0`

---

#### `bst::bst(const DataType* arr, const long long size)`
**Meaning:** Builds a BST from an array.

**Behavior**
- If the array is sorted ascending (`is_sorted(arr, size)` returns true):
  - builds a balanced BST in **O(N)** using `fill_sorted`
- Otherwise:
  - inserts each item using `insert`, cost **O(N log N)** on average
- Sets `root->parent = NULL` when done (if non-empty)

**Notes / Caveats**
- The comment indicates duplicates are not removed when “sorted”; however the provided fill algorithm constructs nodes directly from positions and does not check duplicates.

---

#### `bst::bst(const vector<DataType>& arr)`
**Meaning:** Builds a BST from a vector, with the same sorted optimization as the array constructor.

---

#### `bst::bst(const bst& src)`
**Meaning:** Copy constructor.

**Behavior**
- Creates a deep copy of `src` using `copy_bst(src.root)`
- Copies `size`

---

#### `bst::~bst()`
**Meaning:** Destructor.

**Behavior**
- Deletes the whole tree using `del_tree(root)`
- Nulls internal pointers and resets size

---

#### `void bst::operator=(const bst& src)`
**Meaning:** Assignment operator (deep copy).

**Behavior**
- If assigning from a non-empty tree:
  - deletes current tree, then deep-copies `src`
- Else:
  - deletes current tree and becomes empty

**Important Note**
- This assignment operator empties the destination if `src.root == NULL`.

---

## 1.4 BST Properties & Utility Checks

#### `bool bst::is_sorted(const DataType* arr, long long size)`
**Meaning:** Checks if an array is non-decreasing by using `>` comparisons.

**Returns**
- `true` if each `arr[i] <= arr[i+1]`
- `false` otherwise

---

#### `bool bst::is_sorted(const vector<DataType>& arr)`
Same logic as the array overload.

---

#### `bool bst::is_sorted(ifstream& file)`
**Meaning:** Checks if values parsed from a file are sorted.

**How it works**
- Reads two consecutive values (`d1`, `d2`) repeatedly using `operator>>`
- Ignores `'\0'` characters inside the stream (persistence writes null separators)
- Returns `false` if it finds `d1 > d2`, else `true`

**Notes**
- Treats empty files as sorted.
- Depends on correct `operator>>` for `DataType`.

---

## 1.5 Core Operations: Insert, Search, Remove

#### `bool bst::insert(const DataType& data)`
**Meaning:** Inserts `data` into BST if it is unique.

**Returns**
- `true` if inserted
- `false` if an equivalent value (`==`) already exists

**Algorithm**
- Iterative traversal from root
- Places new node in a leaf position
- Uses `get_node` for allocation and sets parent pointers

**Side effects**
- Increments `size` when attempting insert; decrements it back if duplicate

---

#### `bool bst::search(const DataType& data, node<DataType>* ptr = NULL) const`
**Meaning:** Finds a node containing `data`.

**Returns**
- `true` if found
- `false` if not found

**Behavior**
- Sets `traverser` to:
  - `root` if `ptr == NULL`
  - else `ptr`
- Iteratively compares and moves left/right
- If found, leaves `traverser` pointing at the matching node

**Note**
- `traverser` is declared `mutable` so it can be set inside `const` search.

---

#### `bool bst::remove(const DataType& data, node<DataType>* ptr = NULL)`
**Meaning:** Removes the node containing `data` (if found).

**Returns**
- `true` if a node was removed
- `false` otherwise

**Deletion cases handled**
1. **Leaf node**: delete directly; parent link fixed
2. **Only left child**: replace node with left child
3. **Only right child**: replace node with right child
4. **Two children**:
   - chooses predecessor from left subtree (`get_max(traverser->left)`)
   - removes that predecessor node
   - copies predecessor data into the originally removed position

**Notes**
- Uses `fix_parent` and parent pointers to repair links.
- Resets `traverser = root` at end.

---

#### `bool bst::remove_tree()`
**Meaning:** Deletes the entire BST.

**Returns**
- `true` if the tree was non-empty and deleted
- `false` if already empty

---

#### `bool bst::remove_subtree(const DataType& data)`
**Meaning:** Deletes the node matching `data` and the entire subtree under it.

**Returns**
- `true` if found and deleted
- `false` otherwise

**Behavior**
- Uses `search(data)` to set `traverser`
- Disconnects subtree from parent with `fix_parent(traverser, NULL)`
- Deletes subtree by `del_tree(traverser)`

---

## 1.6 Traversals / Printing

All traversal methods default to starting at `root` if passed `NULL`.

#### `void bst::inorder(node<DataType>* ptr = NULL) const`
**Meaning:** In-order traversal (Left, Node, Right) printing to `cout`.

---

#### `void bst::preorder(node<DataType>* ptr = NULL) const`
**Meaning:** Pre-order traversal (Node, Left, Right) printing to `cout`.

---

#### `void bst::postorder(node<DataType>* ptr = NULL) const`
**Meaning:** Post-order traversal (Left, Right, Node) printing to `cout`.

---

#### `void bst::breadth_first(node<DataType>* ptr = NULL) const`
**Meaning:** Level-order traversal using a queue, printing to `cout`.

---

## 1.7 Equality and Ordering Operators

#### `bool bst::operator==(const bst& src) const`
**Meaning:** Checks structural equality and data equality.

**Returns**
- `true` if:
  - both empty, or
  - both have identical shape and each corresponding node has equal data
- `false` otherwise

**Method**
- Breadth-first parallel traversal
- Ensures left/right child existence matches (`l1 == l2`, `r1 == r2`)
- Compares node data using `==`

---

#### `bool bst::operator<(const bst& src) const`
**Meaning:** Compares *maximum* element of each tree.

**Intended behavior**
- Returns true if `max(this) < max(src)`

**Implementation note**
- The `.cpp` calls `get_max()` without arguments, but the declared helper is `get_max(node*)`. This suggests the comparator may be incomplete/incorrect as written.

---

#### `bool bst::operator>(const bst& src) const`
Same intent as `<` but for `>` comparison of maximum elements. Same caveat regarding `get_max()` call signature.

---

## 1.8 Traverser Cursor / Access

#### `void bst::move()`
**Meaning:** Interactive navigation of the BST using a cursor (`traverser`).

**Controls**
- `w` — move to parent
- `a` — move to left child
- `d` — move to right child
- `s` — show tree (breadth-first)
- `r` — reset to root
- `q` — quit (resets traverser to root)
- Enter (`'\r'`) — stop and return at current

**Dependencies**
- Uses `getch()` from `<conio.h>` (platform-specific, typically Windows).

---

#### `DataType bst::access_traverser()`
**Meaning:** Returns a copy of the element currently pointed to by `traverser`.

**Behavior**
- If `traverser` is null, returns a default-constructed local variable `data` (uninitialized for primitive types).

---

#### `long long bst::get_size()`
**Meaning:** Returns the number of nodes currently in the tree.

---

## 1.9 Internal BST Helper Functions (Implementation Details)

These are private and used internally.

#### `node<DataType>* bst::get_node(const DataType& data)`
Allocates and initializes a new node:
- sets `data`
- sets `left/right/parent = NULL`

---

#### `void bst::del_tree(node<DataType>* ptr)`
Deletes an entire tree/subtree using breadth-first deletion.

Behavior highlights:
- If `ptr` has a parent, disconnects it using `fix_parent(ptr, NULL)`
- Deletes every node reachable from `ptr`
- Decrements `size` by the number of deleted nodes
- If deleting the root subtree, sets `root = NULL`

---

#### `node<DataType>* bst::copy_bst(const node<DataType>* ptr)`
Performs breadth-first deep copy of a subtree, preserving structure and parent pointers.

Behavior:
- Returns the new root pointer
- Updates `size` while copying

---

#### `node<DataType>* bst::get_max(node<DataType>* ptr) const`
Returns rightmost node in the subtree rooted at `ptr`.

**Precondition:** `ptr != NULL`

---

#### `node<DataType>* bst::get_min(node<DataType>* ptr) const`
Returns leftmost node in the subtree rooted at `ptr`.

**Precondition:** `ptr != NULL`

---

#### `bool bst::is_left(node<DataType>* ptr) const`
Returns true if `ptr` exists and is the left child of its parent.

---

#### `bool bst::is_right(node<DataType>* ptr) const`
Returns true if `ptr` exists and is the right child of its parent.

---

#### `bool bst::is_root(node<DataType>* ptr) const`
Returns true if `ptr == root`.

---

#### `void bst::fix_parent(node<DataType>* ptr, node<DataType>* dest)`
Fixes the parent’s pointer to `ptr` so it points to `dest` instead.

Used in deletion and subtree removal.

---

#### `node<DataType>* bst::fill_sorted(...)`
Builds a balanced BST in **O(N)** from already sorted data.

Overloads:
- `fill_sorted(const DataType* arr, long long beg, long long end, node* ptr)`
- `fill_sorted(const vector<DataType>& arr, long long beg, long long end, node* ptr)`
- `fill_sorted(ifstream& file, ifstream& index_file, long long beg, long long end, node* ptr)`

The file-based version uses the `.bin` index to `seekg()` directly to the serialized element position, enabling O(N) rebuild without re-reading everything sequentially.

---

#### `void bst::to_string(string& out, node<DataType>* ptr = NULL) const`
Produces an in-order concatenation of elements into `out`.

Implementation detail:
- Appends each element string plus a `'\0'` separator:
  - `out.append(::to_string(ptr->data) + '\0')`
- This design matches file persistence which also uses null separators.

---

---

## 2) Core Topic: File Persistence (Save/Load)

This library supports two persistence modes:

1. **In-order persistence with index file** (fast O(N) rebuild when index exists)
2. **Breadth-first persistence** (preserves exact shape)

### 2.1 In-order Save / Load (with `.bin` index)

#### `bool bst::save() const`
**Meaning:** Saves the tree in-order into a user-provided file path, and writes a companion binary index file at `filePath + ".bin"`.

**Output format**
- Data file:
  - serialized as text using `operator<<` per element
  - followed by `'\0'` after each element
- Index file (`.bin`):
  - for each element in in-order sequence, stores the `std::streampos` (byte offset) of the start of that element in the data file

**Returns**
- `true` on success
- `false` if tree empty or file could not be opened

**User interaction**
- Prompts for file path via `getline(cin, file_path)`

---

#### `bool bst::load()`
**Meaning:** Loads a tree from a file path (prompted). Chooses best loading strategy depending on file availability and whether data is sorted.

**Strategy**
- If both data file and index file exist:
  - uses `load_inorder` → **O(N)** balanced reconstruction
- Else if only data file exists:
  - if file data is sorted (`is_sorted(file)`):
    - reads all into a vector then uses `fill_sorted` → **O(N)**
  - else:
    - uses `load_unorder` (inserts as read) → **O(N log N)** average

**Returns**
- `true` if some loading strategy succeeded
- `false` otherwise

**Important behavior**
- If the current tree is not empty, it calls `del_to_load()` which:
  - prompts whether to save current tree
  - deletes current tree before loading

---

#### `void bst::inorder_save(node<DataType>* ptr, ofstream& file, ofstream& index_file) const`
**Meaning:** Internal recursion to save nodes in-order while writing index positions.

**Key behavior**
- Before writing node data, captures `file.tellp()` and writes it to index file.

---

#### `bool bst::load_inorder(ifstream& file, ifstream& index_file)`
**Meaning:** Loads a tree using the index file.

**Behavior**
- Determines number of elements by:
  - `count = (index_file_size / sizeof(std::streampos))`
- Calls:
  - `root = fill_sorted(file, index_file, 0, count-1, root)`
- Sets:
  - `traverser = root`
  - `size = count`

---

#### `bool bst::load_unorder(ifstream& file)`
**Meaning:** Loads data assuming it is not ordered; inserts each parsed element via `insert`.

**Parsing detail**
- Skips `'\0'` characters inside the file.

---

### 2.2 Breadth-first Save / Load (shape-preserving)

These functions serialize the tree in level order and include placeholders for null children.

#### Global: `string filling_string = "NULL"`
Used as a sentinel token written to file for missing nodes.

---

#### `void bst::save_breadth_first() const`
**Meaning:** Saves the tree in level order so it can be reconstructed with identical structure.

**File format**
- One token per line:
  - either a node’s `operator<<` representation
  - or `filling_string` for null nodes

**Termination**
- Uses helper `is_null_queue(q)` to detect when the remaining queue contains only nulls, then stops.

**User interaction**
- Prompts for file path.

---

#### `void bst::load_breadth_first()`
**Meaning:** Loads a tree saved in breadth-first format.

**Behavior**
- If current tree non-empty:
  - prompts to save
  - deletes current tree
- Reads first line as root
- Uses a queue to attach left and right children in order
- For each child token:
  - if token equals `filling_string`, child is treated as null and not created
  - otherwise, a new node is created

**Notes**
- Requires `DataType` to be constructible from parsing a line using `operator>>` via `istringstream`.

---

#### Helper: `template<typename DataType> bool is_null_queue(queue<node<DataType>*>& q)`
**Meaning:** Returns true if the queue contains only `NULL` pointers.

Used to stop breadth-first save when the remaining tree frontier is empty.

---

---

## 3) Core Topic: Abstraction Layer for Database-like Records

This topic is primarily implemented by:

- `other_data`
- `custom_data<unique_data_type>`

The intended usage is to store `custom_data<KeyT>` objects inside `bst<custom_data<KeyT>>` as a “database”.

### 3.1 Enumerators for type slots

Defined in `bst.h`:

```cpp
enum { unique_element = 0,
       long_count    = 1,
       double_count  = 2,
       string_count  = 3,
       char_count    = 4,
       bool_count    = 5 };
```

These enumerators are used to refer to typed arrays inside `other_data` / `custom_data`.

---

## 4) Class: `other_data`

Defined in `bst.h`, implemented in `bst.cpp`.

### Purpose
Stores dynamically allocated arrays for a fixed set of primitive-like types:

- `long* longs`
- `double* doubles`
- `string* strings`
- `char* characters`
- `bool* booleans`

Also tracks the allocated sizes in `data_type_count[5]`.

### Constructors / Destructor

#### `other_data::other_data()`
**Meaning:** Empty initialization.

**Behavior**
- sets all counts to 0
- sets all pointers to `NULL`

---

#### `other_data::other_data(int arr[5])`
**Meaning:** Allocates each typed array using sizes from `arr` in the `ldscb` order.

**Behavior**
- allocates each vector only if size > 0 (otherwise keeps pointer `NULL`)
- sets `data_type_count` accordingly

**Mapping**
- `arr[long_count-1]` → size of `longs`
- `arr[double_count-1]` → size of `doubles`
- `arr[string_count-1]` → size of `strings`
- `arr[char_count-1]` → size of `characters`
- `arr[bool_count-1]` → size of `booleans`

---

#### `other_data::other_data(int integer_array_size, int floating_point_array_size, int string_array_size, int char_array_size, int bool_array_size)`
Same intent as the array constructor, but with explicit parameters.

---

#### `other_data::~other_data()`
Deletes each allocated array (if not null).

---

### Operators / Methods

#### `void other_data::operator=(const other_data& other)`
Deep-copies each typed array from `other` into `this`.

Uses helper `copy_vec<T>`.

---

#### `string other_data::to_string() const`
Serializes the object into a single string:

**Format**
1. The 5 counts as decimal strings, separated by spaces
2. All elements of each array appended in order:
   - longs, doubles, strings, characters, booleans
3. Each element is converted via the library’s `::to_string(...)` overloads and appended with a trailing space

---

---

## 5) Class: `template<typename unique_data_type> custom_data`

Defined in `bst.h`, implemented in `bst.cpp`.

### Purpose
Represents a “record” with:

- `unique_parameter` — the key used for BST ordering and uniqueness
- `other` (`other_data`) — the typed “columns” storage

### Constructors / Destructor

#### `custom_data::custom_data()`
Creates an “empty schema” record (all arrays sized to 0).

---

#### `custom_data::custom_data(const custom_data& src)`
Copy constructor: copies `other` and `unique_parameter`.

---

#### `custom_data::custom_data(int* arr, int size)`
Allocates `other_data` arrays based on `arr[5]`. The `size` argument is not used in the implementation.

**Precondition**
- `arr` must have at least 5 valid entries.

---

#### `custom_data::custom_data(int integer_array_size, int floating_point_array_size, int string_array_size, int char_array_size, int bool_array_size)`
Allocates typed arrays with explicit sizes.

---

#### `custom_data::~custom_data()`
Empty (relies on `other_data` destructor).

---

### Schema/Resizing

#### `void custom_data::set_data_type_size(int data_type_enum_value, int size)`
Resizes one of the arrays inside `other_data`.

**Parameters**
- `data_type_enum_value`: one of `long_count`, `double_count`, `string_count`, `char_count`, `bool_count`
- `size`: new element count (if `>= 0`)

**Behavior**
- Calls `resize_vec<T>` for the matching type
- If `size == 0`, the array becomes `NULL` and count becomes 0 (via resize helper allocating null)

---

### Comparison Operators (BST Key Semantics)

All comparisons only use `unique_parameter`:

- `operator<`
- `operator>`
- `operator<=`
- `operator>=`
- `operator==`
- `operator!=`

**Meaning**
- Two records compare equal if and only if their keys are equal, regardless of other fields.

**Important**
- For use in `bst<custom_data<KeyT>>`, the key must be unique per record.

---

### Serialization

#### `string custom_data::to_string() const`
Serializes a record as:

1. `unique_parameter` converted to string using `std::to_string(unique_parameter)` (note: this assumes numeric type)
2. a space
3. `other.to_string()`

---

### Accessors

#### `unique_data_type& at_unique()`
Returns mutable reference to the unique key.

#### `const unique_data_type& at_unique() const`
Const overload.

---

#### `long& at_longs(int index)` / `const long& at_longs(int index) const`  
#### `double& at_doubles(int index)` / `const double& at_doubles(int index) const`  
#### `string& at_strings(int index)` / `const string& at_strings(int index) const`  
#### `char& at_chars(int index)` / `const char& at_chars(int index) const`  
#### `bool& at_bools(int index)` / `const bool& at_bools(int index) const`

**Meaning**
Accesses a column slot by index.

**Bounds**
- If `index` is out of bounds, the function does not return a valid reference (undefined behavior). Correct usage requires checking indices.

---

#### `int get_data_type_count(int dt) const`
Returns the allocated size of the selected array.

**Parameter**
- `dt` should be one of 1..5 (the enumerator values for types).

**Returns**
- size if valid dt
- `-1` otherwise

---

### Stream Operators

#### `ostream& operator<<(ostream& os, const custom_data<unique_data_type>& obj)`
Writes `obj.to_string()` to stream.

---

#### `istream& operator>>(istream& is, custom_data<unique_data_type>& obj)`
Reads a serialized record in this order:

1. unique parameter
2. size of longs array
3. size of doubles array
4. size of strings array
5. size of chars array
6. size of bools array
7. reads longs elements
8. reads doubles elements
9. reads strings elements
10. reads chars elements
11. reads bools elements

**Important**
- This operator dynamically resizes arrays to match the serialized schema before reading values.

---

---

## 6) Topic: Helper Functions and Conversions

### 6.1 Input Helpers

#### `bool getInput(char& choice)`
Reads a `char` from `cin` and validates stream state.

**Returns**
- `true` if read succeeded
- `false` if `cin.fail()`; clears error and discards remaining input line

---

#### `bool getInput(int& choice)`
Same behavior for `int`.

---

### 6.2 File Open Helpers

#### `bool openFileForWriting(const string& filePath)`
Tries to open an `ofstream` to `filePath` (default mode).

**Returns**
- `true` if open succeeded
- `false` otherwise

---

#### `bool openFileForReading(const string& filePath)`
Tries to open an `ifstream` to `filePath`.

**Returns**
- `true` if open succeeded
- `false` otherwise

---

### 6.3 `to_string` Overloads

Declared in `bst.h` and implemented in `bst.cpp`.

#### Scalar overloads
- `string to_string(const int&)`
- `string to_string(const unsigned int&)`
- `string to_string(const float&)`
- `string to_string(const double&)`
- `string to_string(const long&)`
- `string to_string(const char&)`
- `string to_string(const bool&)`
- `string to_string(const string&)`

**Notes**
- `to_string(const long&)` and `to_string(const unsigned int&)` internally cast to `int`, which can truncate large values.

---

#### `template<typename DataType> string to_string(const vector<DataType>& vec)`
Converts a vector into a space-separated string by calling `::to_string` for each element.

---

#### `template<typename DataType> string to_string(const bst<DataType>& tree)`
Returns a string representation of the tree by calling `tree.to_string(str)`.

Tree string format:
- in-order traversal
- each element appended with a `'\0'` separator

---

#### `template<typename unique_data_type> string to_string(const custom_data<unique_data_type>& p)`
Returns `p.to_string()`.

---

### 6.4 Stream Operators for BST

#### `template<typename DataType> ostream& operator<<(ostream& os, const bst<DataType>& obj)`
Writes the tree’s in-order `to_string` output into the stream.

---

#### `template<typename DataType> istream& operator>>(istream& is, bst<DataType>& obj)`
Reads lines until an empty line is encountered; for each line, extracts tokens of `DataType` and inserts them.

**Stop condition**
- stops when `getline` returns an empty line

---

---

## 7) Example / Demo Layer: `civil_record`

This section is included in the same files as a usage example. It demonstrates how to build a small database using:

- `bst<custom_data<int>> data_base;`
- `custom_data<int> person;`

### Enumerators for string fields
```cpp
enum { first_name = 0, last_name, birth_date, city };
```

### `class civil_record` methods

#### `civil_record::civil_record()`
Initializes `person` with schema sizes:
- longs: 1
- doubles: 1
- strings: 4
- chars: 1
- bools: 1

---

#### `civil_record::~civil_record()`
Prints database size and calls `system("pause")`.

---

#### `void insert_person()`
Prompts for ID and record fields, then inserts into BST.

Key behavior:
- loops until the ID is not found (`while(data_base.search(person))`)
- then collects fields and calls `data_base.insert(person)`

---

#### `void delete_person()`
Prompts for ID and calls `data_base.remove(person)`.

---

#### `void search_person()`
Prompts for ID, searches, and if found:
- assigns `person = data_base.access_traverser()`
- prints fields

---

#### `void save_data_base()`
Calls `data_base.save()`.

---

#### `void load_data_base()`
Calls `data_base.load()` with a note that schema must match.

---

#### `void delete_data_base()`
Prompts to save and then calls `data_base.remove_tree()`.

---

#### `void show_data_base()`
Calls `data_base.inorder()`.

---

## 8) Summary of Internal Dynamic-Array Helpers (Used by Abstraction Layer)

These are defined in `bst.cpp` and are not part of the public API but are crucial to `other_data` / `custom_data`.

#### `template<typename T> T* get_vec(const int size)`
Allocates a dynamic array of `T[size]` if `size > 0`, else returns `NULL`.

---

#### `template<typename T> void copy_vec(T*& dest, const T* src, int& dest_size, const int src_size)`
Deep copies `src` into `dest`:
- deletes old `dest` if needed
- allocates new dest if `src` is non-null
- copies elements
- updates `dest_size`

---

#### `template<typename T> string vec_to_string(T* vec, int size)`
Serializes an allocated array to a space-separated string using `::to_string`.

---

#### `template<typename T> void resize_vec(T*& vec, int& old_vec_size, int wanted_size)`
Resizes `vec` to `wanted_size`:
- allocates a new array (or NULL if `wanted_size <= 0`)
- copies min(old_size, new_size) elements
- deletes old array
- updates `old_vec_size`

---

## 9) File Index

- `bst.h` — declarations and in-header documentation for:
  - `node<T>`
  - `bst<T>`
  - persistence interfaces
  - `other_data`
  - `custom_data<T>`
  - example `civil_record`
- `bst.cpp` — implementations for all above, plus a `main()` demo

---

## 10) Practical Usage Patterns

### 10.1 Simple BST of primitive types
- `bst<int> t;`
- `t.insert(5);`
- `t.save();` / `t.load();`

### 10.2 “Database” BST keyed by integer
- `bst<custom_data<int>> db;`
- Each `custom_data<int>` has:
  - key: `at_unique()`
  - typed fields stored in `other_data` arrays

### 10.3 Persistence rules of thumb
- Use `save()` / `load()` for ordered persistence with `.bin` index acceleration
- Use `save_breadth_first()` / `load_breadth_first()` if structure must be preserved exactly (not just in-order content)

---
