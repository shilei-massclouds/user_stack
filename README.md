# user_stack

User-space stack management module.

This module provides functionality for managing user-space stacks with proper
alignment handling and common stack operations.

## Examples

```rust
#![no_std]
#![no_main]

#[macro_use]
extern crate axlog2;

use core::panic::PanicInfo;
use core::ptr::null;
use axhal::arch::TASK_SIZE;
use user_stack::UserStack;
use axtype::is_aligned;

#[no_mangle]
pub extern "Rust" fn runtime_main(_cpu_id: usize, _dtb_pa: usize) {
    user_stack::init();
    axlog2::init("debug");
    info!("[rt_user_stack]: ...");

    const BUF_SIZE: usize = 256;
    let buffer = [0u8; BUF_SIZE];

    let mut stack =
        UserStack::new(TASK_SIZE, buffer.as_ptr() as usize + BUF_SIZE);
    stack.push(&[null::<u64>()]);

    let random_str: &[usize; 2] =
        &[3703830112808742751usize, 7081108068768079778usize];
    stack.push(random_str.as_slice());

    stack.push_str("hello");
    stack.push_str("world");

    stack.push(&[null::<u8>(), null::<u8>()]);

    if !is_aligned(stack.get_sp(), 16) {
        stack.push(&[null::<u8>()]);
    }

    // pointers to envs
    stack.push(&[null::<u8>()]);
    stack.push(&[null::<u8>()]);
    // pointers to argv
    stack.push(&[null::<u8>()]);
    // argc
    stack.push(&[100]);

    let sp = stack.get_sp();

    info!("[rt_user_stack]: sp {:#X}", sp);
    info!("[rt_user_stack]: ok!");
    axhal::misc::terminate();
}

#[panic_handler]
pub fn panic(info: &PanicInfo) -> ! {
    arch_boot::panic(info)
}
```

## Functions

### `init`

```rust
pub fn init()
```

Initializes the user stack subsystem

## Struct

### `UserStack`

```rust
pub struct UserStack { /* private fields */ }
```

Represents a user-space stack with automatic alignment management

#### Implementations

```rust
pub fn new(base: usize, ptr: usize) -> Self
```

Creates a new user stack instance

```rust
pub fn get_sp(&self) -> usize
```

Returns the current stack pointer position

```rust
pub fn push<T: Copy>(&mut self, data: &[T])
```

Pushes an array of data onto the stack

```rust
pub fn push_str(&mut self, str: &str) -> usize
```

Pushes a string onto the stack, adding a null terminator
