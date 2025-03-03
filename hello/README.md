## Documentation

##### **Section: .data**
This section is used to define initialized data or constants.

- `msg db "Hello, World !", 0x0A`: Defines a message string "Hello, World !" followed by a newline character (`0x0A`).
- `len equ $ - msg`: Calculates the length of the `msg` string. `$` is the current address, so `$ - msg` gives the length of the string.

##### **Section: .text**
This section contains the code (instructions) that gets executed.

###### **Directive: global _start**
This tells the assembler that `_start` is the entry point of the program.

###### **Label: _start**
This label marks the beginning of the program execution.

###### **First Block: Write Message to Standard Output**
```asm
	mov eax, 4      ; syscall number for sys_write
	mov ebx, 1      ; file descriptor 1 (standard output)
	mov ecx, msg    ; pointer to the message to write
	mov edx, len    ; length of the message
	int 0x80        ; make the system call
```
- `mov eax, 4`: Load the syscall number for `sys_write` into register `eax`.
- `mov ebx, 1`: Load the file descriptor `1` (standard output) into register `ebx`.
- `mov ecx, msg`: Load the address of the message to write into register `ecx`.
- `mov edx, len`: Load the length of the message into register `edx`.
- `int 0x80`: Trigger the interrupt `0x80` to make the system call, which writes the message to the standard output.

###### **Second Block: Exit Program**
```asm
	mov eax, 1      ; syscall number for sys_exit
	mov ebx, 0      ; exit status 0 (success)
	int 0x80        ; make the system call
```
- `mov eax, 1`: Load the syscall number for `sys_exit` into register `eax`.
- `mov ebx, 0`: Load the exit status `0` (success) into register `ebx`.
- `int 0x80`: Trigger the interrupt `0x80` to make the system call, which terminates the program.

This code writes the message "Hello, World !" to the standard output and then exits the program successfully.

---

### Why does the code has to follow this particular order ?

The order of instructions in your code is crucial to correctly performing the `sys_write` system call. Each instruction sets up the necessary registers with the correct values before the interrupt (`int 0x80`) is triggered. Here's a detailed explanation:

1. **System Call Number (Setting up `eax`):**
   ```asm
   mov eax, 4      ; syscall number for sys_write
   ```
   The `eax` register needs to hold the system call number for the `sys_write` function, which is `4`. This tells the operating system which system call you want to perform. If you do not set `eax` correctly, the system won't know you want to use `sys_write`.

2. **File Descriptor (Setting up `ebx`):**
   ```asm
   mov ebx, 1      ; file descriptor 1 (standard output)
   ```
   The `ebx` register needs to hold the file descriptor. `1` represents standard output (usually the terminal). By setting `ebx` to `1`, you're specifying where the output should be sent. If you don't set this value, the system won't know where to write the data.

3. **Message Pointer (Setting up `ecx`):**
   ```asm
   mov ecx, msg    ; pointer to the message to write
   ```
   The `ecx` register needs to hold the memory address (pointer) of the data to be written. In this case, it's the address of the `msg` string. If `ecx` does not point to the correct data, the system won't know what to write.

4. **Message Length (Setting up `edx`):**
   ```asm
   mov edx, len    ; length of the message
   ```
   The `edx` register needs to hold the length of the data to be written. This ensures the system knows how many bytes to write. If `edx` is not set correctly, the system call might write the wrong amount of data.

5. **Trigger the System Call (using `int 0x80`):**
   ```asm
   int 0x80        ; make the system call
   ```
   The `int 0x80` instruction triggers the interrupt to switch control to the kernel, which then performs the `sys_write` system call using the values in `eax`, `ebx`, `ecx`, and `edx`.

In summary, each step sets up a critical parameter for the `sys_write` system call. The values in `eax`, `ebx`, `ecx`, and `edx` must be correctly assigned before the `int 0x80` instruction is executed. Any deviation or reordering could lead to incorrect behavior or failure of the system call.


---


In x86 assembly programming, the specific registers used (`eax`, `ebx`, `ecx`, `edx`) have predefined roles in the context of system calls. The Linux operating system expects certain values to be in specific registers for each system call to work correctly. Here's why you must follow this order:

1. **`eax` Register (System Call Number):**
   - `eax` must contain the syscall number, which tells the operating system which system call to execute. For `sys_write`, this number is `4`.
   - Without setting `eax` first, the OS won't know that you want to execute the `sys_write` function.

2. **`ebx` Register (File Descriptor):**
   - `ebx` must contain the file descriptor to which the data will be written. For standard output, this is `1`.
   - If `ebx` is not set properly, the data might be directed to an unintended file or output stream.

3. **`ecx` Register (Pointer to Data):**
   - `ecx` must point to the address of the data to be written.
   - Setting `ecx` to the wrong address would cause the wrong data to be written.

4. **`edx` Register (Length of Data):**
   - `edx` must contain the length of the data to be written.
   - If `edx` is not set to the correct length, the OS might write more or less data than intended.

The order in which you set these registers matters because each register needs to be loaded with the correct value before the interrupt (`int 0x80`) is called. Changing the order might result in undefined behavior or a failed system call because the operating system relies on the values being in the correct registers at the moment the interrupt is triggered.

So, while you can technically change the order of the instructions, you must ensure that all registers are correctly set before the `int 0x80` instruction. The following is an alternative order that would still work:

```asm
	mov ecx, msg    ; pointer to the message to write
	mov ebx, 1      ; file descriptor 1 (standard output)
	mov edx, len    ; length of the message
	mov eax, 4      ; syscall number for sys_write
	int 0x80        ; make the system call
```

As long as `eax`, `ebx`, `ecx`, and `edx` are set before `int 0x80`, the system call should work correctly. This flexibility allows you to reorder the instructions to some extent, provided the final state of the registers is correct just before invoking the interrupt.

If you need further assistance, feel free to ask! 😊
