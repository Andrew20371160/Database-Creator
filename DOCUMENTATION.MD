# Database Creator - Complete Documentation

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Core Data Structures](#core-data-structures)
4. [Main Classes](#main-classes)
5. [Key Functions](#key-functions)
6. [Usage Guide](#usage-guide)
7. [Data Storage & Persistence](#data-storage--persistence)
8. [Example Implementation](#example-implementation)

---

## Project Overview

The Database Creator is a C++ template-based database system that enables engineers to create custom databases with their own schema through a Binary Search Tree (BST) implementation. The system provides essential database operations including:

- **Insertion**: Add new records with unique identifiers
- **Search**: Find records by unique parameter
- **Deletion**: Remove individual records or entire subtrees
- **Persistence**: Save and load data to/from files
- **Traversal**: Navigate and display data in multiple formats (inorder, preorder, postorder, level-order)

**Repository**: `Andrew20371160/Database-Creator`  
**Language**: C++ (100%)  
**Files**: `bst.h`, `bst.cpp`

---

## Architecture Overview

The database is built on a **template-based Binary Search Tree (BST)** architecture with support for custom data types. The main architectural components are:

### 1. **Node Structure**
```cpp
template <typename DataType>
struct node {
    DataType data;
    node<DataType>* left;
    node<DataType>* right;
    node<DataType>* parent;
};
```

Each node stores a generic data type and maintains pointers to left, right, and parent nodes.

### 2. **Template Classes**
- **`bst<DataType>`**: Generic binary search tree template
- **`custom_data<unique_data_type>`**: Custom data container with unique parameter and additional data fields
- **`other_data`**: Auxiliary data storage for multiple data types (long, double, string, char, bool)

### 3. **Design Pattern**
The system uses a **template metaprogramming** approach with:
- Generic tree operations independent of data type
- Support for both primitive types (int, double, char) and complex types (custom structures)
- Separation of unique identifying data from auxiliary data

---

## Core Data Structures

### `node<DataType>` Template Structure

Represents a single BST node with:
- **data**: The actual payload
- **left**: Pointer to left subtree
- **right**: Pointer to right subtree
- **parent**: Pointer to parent node (for bidirectional traversal)

### `bst<DataType>` Template Class

The main template class providing all database operations.

#### Private Members:
```cpp
node<DataType>* root;              // Root of the tree
mutable node<DataType>* traverser; // Navigation pointer
long long size;                    // Number of nodes
```

#### Private Helper Methods:
- `get_node()`: Allocate new node
- `is_left()`, `is_right()`, `is_root()`: Node position checks
- `fix_parent()`: Update parent pointers
- `get_max()`, `get_min()`: Find extremal nodes
- `copy_bst()`: Deep copy operations
- `del_tree()`: Recursive tree deletion
- `fill_sorted()`: O(N) tree construction from sorted data
- `inorder_save()`: Save tree with index file

### `custom_data<unique_data_type>` Class

Enables creation of custom database records with:

#### Structure:
```cpp
template<typename unique_data_type>
class custom_data {
private:
    unique_data_type unique_parameter;  // Unique key
    other_data other;                   // Additional data
};
```

#### Supported Additional Data Types (LDSCB ordering):
- **L**: Long integers (long array)
- **D**: Double floating-point (double array)
- **S**: Strings (string array)
- **C**: Characters (char array)
- **B**: Booleans (bool array)

### `other_data` Class

Manages allocation and storage of multiple data type arrays:

```cpp
class other_data {
private:
    string* strings;
    double* doubles;
    long* longs;
    char* characters;
    bool* booleans;
    int data_type_count[5];  // Size of each array
};
```

---

## Main Classes

### 1. **bst<DataType> Class**

#### Constructors:
```cpp
bst()                                    // Empty tree
bst(const DataType* arr, long long size) // From array
bst(const vector<DataType>& arr)         // From vector
bst(const bst& src)                      // Copy constructor
```

#### Public Methods:

**Insertion & Deletion:**
```cpp
bool insert(const DataType& data)              // Insert element
bool remove(const DataType& data)              // Remove element
bool remove_tree()                             // Clear entire tree
bool remove_subtree(const DataType& data)      // Remove subtree
```

**Searching:**
```cpp
bool search(const DataType& data) const        // Find element
DataType access_traverser()                    // Access found element
```

**Traversal & Display:**
```cpp
void inorder() const                           // In-order traversal
void preorder() const                          // Pre-order traversal
void postorder() const                         // Post-order traversal
void breadth_first() const                     // Level-order (BFS)
void move()                                    // Interactive navigation
```

**File Operations:**
```cpp
bool save() const                              // Save with index file
bool load()                                    // Load from file
void save_breadth_first() const               // Save tree structure
void load_breadth_first()                      // Load tree structure
```

**Utility:**
```cpp
long long get_size()                           // Get element count
bool is_sorted(...)                            // Check if data is sorted
```

**Operators:**
```cpp
void operator=(const bst& src)                 // Assignment
bool operator==(const bst& src) const          // Equality
bool operator<(const bst& src) const           // Compare max values
bool operator>(const bst& src) const           // Compare max values
template<typename T>
ostream& operator<<(ostream&, const bst<T>&)  // Print tree
```

### 2. **custom_data<unique_data_type> Class**

#### Constructors:
```cpp
custom_data()                                           // Empty
custom_data(const custom_data& src)                     // Copy
custom_data(int arr[5])                                 // From array sizes
custom_data(int longs, int doubles, int strings,        // Individual sizes
            int chars, int bools)
```

#### Public Methods:

**Data Access (Accessors):**
```cpp
unique_data_type& at_unique()                           // Access unique key
long& at_longs(int index)                               // Access long at index
double& at_doubles(int index)                           // Access double at index
char& at_chars(int index)                               // Access char at index
string& at_strings(int index)                           // Access string at index
bool& at_bools(int index)                               // Access bool at index
```

**Configuration:**
```cpp
void set_data_type_size(int data_type_enum, int size)   // Resize array
int get_data_type_count(int dt) const                    // Get array size
```

**Comparison Operators:**
```cpp
bool operator<(const custom_data& src) const   // Based on unique_parameter
bool operator>(const custom_data& src) const
bool operator<=(const custom_data& src) const
bool operator>=(const custom_data& src) const
bool operator==(const custom_data& src) const
bool operator!=(const custom_data& src) const
```

**Conversion & I/O:**
```cpp
string to_string() const                                 // Convert to string
void operator=(const custom_data& src)                   // Assignment
template<typename T>
ostream& operator<<(ostream&, const custom_data<T>&)    // Output stream
```

### 3. **other_data Class**

#### Constructors:
```cpp
other_data()                                    // Empty
other_data(int arr[5])                          // From array of sizes
other_data(int longs, int doubles, int strings, // Individual parameters
           int chars, int bools)
```

#### Public Methods:
```cpp
~other_data()                                   // Destructor
void operator=(const other_data& other)         // Assignment
string to_string() const                        // Convert to string
bool operator==(const other_data& other) const  // Equality (not implemented)
```

---

## Key Functions

### Helper Functions for Input Handling

```cpp
bool getInput(char& choice)    // Safe character input
bool getInput(int& choice)     // Safe integer input
```

These functions handle input stream errors gracefully without crashing when invalid input is received.

### File Operations

```cpp
bool openFileForWriting(const string& filePath)
bool openFileForReading(const string& filePath)
```

Validate file access before operations.

### Data Type Conversion

```cpp
string to_string(const int& data)
string to_string(const double& data)
string to_string(const char& data)
string to_string(const bool& data)
string to_string(const string& data)
string to_string(const float& data)
string to_string(const long& data)
string to_string(const unsigned int& data)
template<typename DataType>
string to_string(const vector<DataType>& vec)
```

Convert various data types to string representation for serialization.

### Tree Utility Functions

```cpp
// Vector operations for custom_data
template<typename vector_data_type>
vector_data_type* get_vec(const int size)

template<typename vector_data_type>
void copy_vec(vector_data_type*& dest, const vector_data_type* src,
              int& dest_size, const int src_size)

template<typename vector_data_type>
void resize_vec(vector_data_type*& vec, int& old_vec_size, int wanted_size)

template<typename vector_data_type>
string vec_to_string(vector_data_type* vec, int size)

// Breadth-first save/load helper
template<typename DataType>
bool is_null_queue(queue<node<DataType>*>& q)

// Utility
void delay(int milliseconds)  // Sleep function for testing
```

---

## Usage Guide

### 1. Creating a Simple Database with Primitive Types

```cpp
// Create a BST for integers
bst<int> intTree;

// Insert values
intTree.insert(50);
intTree.insert(30);
intTree.insert(70);

// Search
if (intTree.search(30)) {
    cout << "Found 30\n";
}

// Display in-order
cout << "In-order traversal: ";
intTree.inorder();  // Output: 30 , 50 , 70 ,

// Remove element
intTree.remove(30);

// Get tree size
cout << "Tree size: " << intTree.get_size() << "\n";  // Output: 2
```

### 2. Creating Custom Database with custom_data

```cpp
// Define enumeration for string indices
enum { FIRST_NAME = 0, LAST_NAME, BIRTH_DATE, CITY };

// Create custom data type (int as unique ID)
// Structure: 1 long (age), 1 double (height), 4 strings, 1 char, 1 bool
custom_data<int> person(1, 1, 4, 1, 1);

// Set up BST for custom data
bst<custom_data<int>> database;

// Populate person record
person.at_unique() = 12345;           // ID
person.at_longs(0) = 28;              // Age
person.at_doubles(0) = 5.9;           // Height
person.at_strings(FIRST_NAME) = "John";
person.at_strings(LAST_NAME) = "Doe";
person.at_strings(BIRTH_DATE) = "1995/05/15";
person.at_strings(CITY) = "New York";
person.at_chars(0) = 'B';             // Hair color
person.at_bools(0) = 1;               // Alive status

// Insert into database
database.insert(person);

// Search by ID
if (database.search(person)) {
    cout << "Found: " << database.access_traverser().at_strings(FIRST_NAME) << "\n";
}
```

### 3. Loading Data from Array

```cpp
int data[] = {10, 20, 30, 40, 50};
bst<int> tree(data, 5);  // O(N) if sorted, O(N log N) otherwise
```

### 4. File Persistence

```cpp
// Save tree
bst<int> tree;
tree.insert(50);
tree.insert(30);
tree.insert(70);

if (tree.save()) {
    cout << "Saved successfully\n";
}

// Load tree
bst<int> loadedTree;
if (loadedTree.load()) {
    cout << "Loaded successfully\n";
    loadedTree.inorder();  // Display loaded data
}
```

### 5. Interactive Navigation

```cpp
bst<int> tree;
// ... populate tree ...

tree.move();  // Interactive mode with controls:
              // w: go up (parent)
              // a: go left
              // d: go right
              // s: show whole tree
              // r: reset to root
              // Enter: stop at current node
              // q: quit
```

### 6. Tree Comparison

```cpp
bst<int> tree1, tree2;
// ... populate both ...

if (tree1 == tree2) {
    cout << "Trees are identical\n";
}

if (tree1 < tree2) {
    cout << "Max of tree1 < Max of tree2\n";
}
```

---

## Data Storage & Persistence

### Save Format

The system uses **two files** for optimal performance:

#### 1. **Data File** (binary/text)
- Stores actual tree data in in-order traversal
- Format: `data₁\0data₂\0data₃\0...`
- Null characters (`\0`) separate records

#### 2. **Index File** (.bin)
- Binary file containing file positions
- Each entry stores starting position of corresponding data record
- Enables O(N) loading from sorted files
- Similar to stack pointer concept in memory management

### Loading Strategy

**Two-tier approach:**

1. **If index file exists** (.bin):
   - Assumes data is sorted (in-order)
   - Uses divide-and-conquer from sorted data
   - **Time complexity**: O(N)
   - Creates balanced tree structure

2. **If index file missing**:
   - Checks if data is sorted
   - **If sorted**: Loads into vector, then constructs O(N)
   - **If unsorted**: Builds tree incrementally
   - **Time complexity**: O(N log N)

### Breadth-First Save/Load

Alternative persistence method that preserves tree structure:

```cpp
tree.save_breadth_first()  // Save level-by-level structure
tree.load_breadth_first()  // Restore exact structure
```

Uses `"NULL"` placeholder for empty nodes to maintain tree topology.

---

## Example Implementation

### Civil Records Database (From codebase)

The project includes a practical example: a civil records database system.

```cpp
enum { FIRST_NAME = 0, LAST_NAME, BIRTH_DATE, CITY };

class civil_record {
private:
    bst<custom_data<int>> database;
    custom_data<int> person;

public:
    civil_record();
    ~civil_record();
    
    void insert_person();   // Add new record
    void delete_person();   // Remove record
    void search_person();   // Find and display record
    void save_database();   // Persist to file
    void load_database();   // Load from file
    void delete_database(); // Clear all records
    void show_database();   // Display all records
};
```

#### Main Menu System

```cpp
int main() {
    civil_record record;
    int choice;
    
    while (choice != 8) {
        cout << "\n1---Insert a new person\n";
        cout << "2---Search for a person\n";
        cout << "3---show all the entries of the database\n";
        cout << "4---Delete a person\n";
        cout << "5---Save current database\n";
        cout << "6---Load a database\n";
        cout << "7---Remove current database\n";
        cout << "8---Exit";
        cout << "\nchoice: ";
        cin >> choice;
        
        switch(choice) {
            case 1: record.insert_person(); break;
            case 2: record.search_person(); break;
            case 3: record.show_database(); break;
            case 4: record.delete_person(); break;
            case 5: record.save_database(); break;
            case 6: record.load_database(); break;
            case 7: record.delete_database(); break;
            case 8: return 0;
        }
    }
    return 0;
}
```

#### Record Structure

The civil records example stores:
- **Unique Parameter**: Person ID (int)
- **Age**: Long integer
- **Height**: Double
- **First Name, Last Name, Birth Date, City**: Strings
- **Hair Color**: Character
- **Life Status**: Boolean

---

## Performance Characteristics

| Operation | Best Case | Worst Case | Average Case |
|-----------|-----------|-----------|-----------|
| Insert | O(log N) | O(N) | O(log N) |
| Search | O(log N) | O(N) | O(log N) |
| Remove | O(log N) | O(N) | O(log N) |
| Load (sorted) | O(N) | O(N) | O(N) |
| Load (unsorted) | O(N log N) | O(N log N) | O(N log N) |
| Save | O(N) | O(N) | O(N) |
| Traversal | O(N) | O(N) | O(N) |

---

## Compilation Notes

The project uses:
- Standard C++ library (iostream, fstream, string, vector, queue)
- `<conio.h>` for interactive navigation (platform-specific)
- `<chrono>` and `<random>` for testing utilities

Compile with:
```bash
g++ -std=c++11 bst.cpp -o database_creator
```

---

This comprehensive documentation covers the entire Database Creator system, providing engineers with a complete understanding of its architecture, capabilities, and usage patterns for building custom database applications.
