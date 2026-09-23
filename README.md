<h3 align="center">
  <em>Because ft_putnbr() and ft_putstr() aren’t enough</em>
</h3>

## ⚠️ Disclaimer

- **Full Portfolio:** This repository focuses on this specific project. You can find my entire
42 curriculum 👉 [here](https://github.com/martinnsanzz/42-Curriculum).
- **Subject Rules:** I strictly follow the rules regarding 42 subjects; I cannot share the PDFs,
but I explain the concepts in this README.
- **Archive State:** The code is preserved exactly as it was during evaluation (graded state).
I do not update it, so you can see my progress and mistakes from that time.
- **Academic Integrity:** I encourage you to try the project yourself first. Use this repo only
as a reference, not for copy-pasting. Be patient, you will succeed.

## 📂 Description
The goal of this project is to re-implement the function `printf` from the C standard library
by building a static library called (`libftprintf.a`).

This project teaches a new concept called **Variadic Functions**, functions that handle and
indefinite number of argument using `<stdarg.h>`. This function handles a variadic format string and outputs to stdout formatted text, returning the total number of characters written.

**📦 Library Integration:**
We are allowed to use our `libft` in this project. However, it must be compiled using its own
`Makefile`. This challenges us to understand how to **link external libraries** and manage
**nested compilations** (calling a Makefile from another).

## 🔷 Usage

### Clone repository
To use this project you first need to clone the repository in your directory.

```bash
git clone git@github.com:martinnsanzz/ft_printf.git
```

### Run make
To compile this projet you need to run `make` in the terminal. This will first compile `libft`
from the `libft/` subdirectory and then build `libftprintf.a` at the root of the repository using `cc -Wall -Werror -Wextra` and `ar` to bundle multiple object files into a single archive.

```bash
make
```

### How to use the libray
To use the library in your project you must include the header on top of the file.

```c
#include "libftprintf.h"
```

Once you use the library within your file you need to link the library with the source file
during compilation file as follow:

The requirements for this project are as follows:
- Compiler: cc with flags -Wall -Wextra -Werror
- Archiver: ar (use of libtool is forbidden)
- Standard: C89/C99 compatible (no restrict qualifier in prototypes)

```bash
cc -Wall -Wextra -Werror your_main.c libftprintf.a && ./a.out
```

To utilize the library in you're project you will need to link them together using
the following comand.

This implementation supports the following standard conversions:

| Specifier | Description |
| :---: | :--- |
| `%c` | Prints a single character. |
| `%s` | Prints a string (as defined by the common C convention). |
| `%p` | The `void *` pointer argument has to be printed in hexadecimal format. |
| `%d` | Prints a decimal number (base 10). |
| `%i` | Prints an integer in base 10. |
| `%u` | Prints an unsigned decimal (base 10) number. |
| `%x` | Prints a number in hexadecimal (base 16) lowercase format. |
| `%X` | Prints a number in hexadecimal (base 16) uppercase format. |
| `%%` | Prints a percent sign. |

**Return value:**
- Total characters written on success.
- `-1` if `format` is NULL.
- Partial count if an invalid specifier is encountered (loop breaks early).

## 📖 Resources

For the creation of this project the following resources were used:
- [GeeksforGeeks - Variadic Functions in c](https://www.geeksforgeeks.org/c/variadic-functions-in-c/)
- [cppreference - Variadic Functions](https://cppreference.com/w/c/variadic.html)
- [GeeksforGeeks - 32-bit and 64-bit operating systems](https://www.geeksforgeeks.org/operating-systems/32-bit-vs-64-bit-operating-systems/)
- Peer to peer - For trouble shooting or testing different approaches

**AI usage:**

Claude by Anthropic was used in this project for the following:
- **As a teacher:** Explaining variadic function mechanics, `va_list` internals, 
and low-level C behavior.
- **Rewriting comments:** Improving clarity and precision of function-level documentation.
- **Testing:** Discussing edge cases and verifying expected behavior against 
the standard `printf()`.

AI was NOT used to write or generate any code. All function bodies were written by myself.

## 🤖 Algorithm & Data Structure

## Core loop
`ft_printf` loops across the format string one character at a time. Normal character 
are directly printed to stdout via `ft_printf_char`. When a `%` is found, the next character 
is validated by `check_format` against the included format specifiers.
- If invalid, an error is written to stdeer and the loop breaks immediatly, returning 
a partial count.
- If valid, `ft_handle_spec` calls the appropriate handler and the format pointer is moved 
past the specifier.
This method was chosen for clarity and extensibility. Each specifier goes to exactly one 
handler and adding a new one just requires a new handler and a new branch in `ft_handle_spec`
and `check_format`.

## Length tracking via pointer
All the handler functions are of type void, no handler returns a value. Instead, all handlers
receive `int *len` and increment it directly. This eliminates the need to sum return values
across the call chain and keeps the interface uniform across all handlers. `len` is the
single source of truth for the total character count.

## Base conversion (%x, %X, %p)
Hexadecimal output and pointer output delegate the conversion to `ft_putnbr_base`, which
takes any type of base string.
- The base gets validated via `check_base_error` (rejects bases shorter than 2, bases with
duplicate characters, and bases containing whitespace or `+`/`-`).
- Then `convert_base` gets called recursively. `convert_base` divides by the base size until
the number falls below the base, then unwinds the stack writing digits in most-significant-first order.

## Unsigned integer output (`%u`)
`ft_printf_u_int` applies the same recursive digit-writing strategy directly, without delegating
to `ft_putnbr_base`. Each recursive call writes one decimal digit via `write()` and increments
`len` by 1.

## Integer output (`%d`, `%i`)
`ft_printf_int` uses `ft_itoa` to measure the string length of the number (for updating `len`),
then frees the allocation and delegates actual output to `ft_putnbr_fd`.

# Null and edge case handling
- `ft_printf_char` casts `int` to `unsigned char` before output to avoid sign-extension. It
also handles `'\0'` explicitly via `write`, since string-based output functions stop at null.
- `ft_printf_str` prints `"(null)"` for a NULL string argument.
- `ft_printf_ptr` prints `"(nil)"` for a NULL pointer argument; non-NULL pointers are cast
to `uintptr_t` and printed with a `"0x"` prefix.