---
layout: post
title: Memory Saftey: Double free errors
date: '2021-07-11 00:00:00'
---
 
In C++, you can call **delete** operator to free heap memory. 
Freeing a pointer twice leaves room for attackers to manipulate the linked list of free memory blocks. MITRE's [CWE](https://cwe.mitre.org/data/definitions/415.html) has more details around this.

In this article, we will see an example of a double-free error in C++, and then we will discuss how Rust handles this issue. 

Rust is a language with a high level of abstraction like C++, and it is designed to be memory safe.

## Double-free C++ example
```cpp
int main() {
    // allocate a string in heap memory
    string* heapString = new string("heap-allocated objects are deleted on 'delete heapString;'"); 
    delete heapString; // deallocate string from the heap memory
    // ... some complex body of code 
    delete heapString; // deallocate the string from the heap memory again
    return 0;
}
```
This code compiles without an issue. On runtime, you will get the following error.
```text
free(): double free detected in tcache 2
```
## Rust Memory management
Rust gives us the same level of control over memory like C++/C, but it works somewhat differently. It uses a concept called [ownership](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html) to manage memory. You don't call **free/delete** manually. Rather, the compiler implicitly injects the code to **drop** the memory based on the ownership rules.

The three Ownership rules are;

 - Each value in Rust has a variable that’s called its owner.
 -  There can only be one owner at a time.
 - When the owner goes out of scope, the value will be dropped.
 

 
```rust
use std::mem::drop; // equivalent to delete()

fn main() {
    // allocate a string in heap memory
    let heapString:String ="heap-allocated object".to_string(); 
    drop(heapString); // explictly call drop to deallocate string from the heap memory
    // ... some complex body of code 
    drop(heapString); //  explictly call drop to deallocate the string from the heap memory again
}
```
This code won't compile, we will get an error. Due to Rust’s ownership semantics, when we free a value, we relinquish ownership on it, which means subsequent attempts to use the value are no longer valid.
```text
error[E0382]: use of moved value: `heapString`
  --> DoubleFree.rs:7:10
   |
5  |     let heapString:String ="heap-allocated object".to_string();
   |         ---------- move occurs because `heapString` has type `std::string::String`, which does not implement the `Copy` trait
6  |     drop(heapString);
   |          ---------- value moved here
7  |     drop(heapString);
   |          ^^^^^^^^^^ value used here after move

error: aborting due to previous error

For more information about this error, try `rustc --explain E0382`.

```
Double free errors result in executions that may allow an attacker to execute arbitrary code. Rust compiler's borrow checker steps in to make sure the programmer is writing memory-safe code.









