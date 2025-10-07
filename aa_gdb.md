# 🧠 GDB Cheat Sheet

## Running

```bash
# gdb <program> [core dump]
```

Start GDB (with optional core dump).

```bash
# gdb --args <program> <args…>
```

Start GDB and pass arguments.

```bash
# gdb --pid <pid>
```

Start GDB and attach to process.

```bash
set args <args...>
```

Set arguments to pass to the program to be debugged.

**run** — Run the program to be debugged.
**kill** — Kill the running program.

---

## Breakpoints

```bash
break <where>
```

Set a new breakpoint.

```bash
delete <breakpoint#>
```

Remove a breakpoint.

```bash
clear
```

Delete all breakpoints.

```bash
enable <breakpoint#>
disable <breakpoint#>
```

Enable or disable breakpoints.

---

## Watchpoints

```bash
watch <where>
delete/enable/disable <watchpoint#>
```

Like breakpoints.

`<where>` can be:

* `function_name` — Break/watch the named function.
* `line_number` — Break/watch the line number in the current source file.
* `file:line_number` — Break/watch the line number in the named source file.

---

## Conditions

```bash
break/watch <where> if <condition>
```

Break/watch at the given location if the condition is met.
Conditions may be any C expression evaluating to true or false.

```bash
condition <breakpoint#> <condition>
```

Set or change the condition of an existing break- or watchpoint.

---

## Examining the Stack

```bash
backtrace
where
```

Show call stack.

```bash
backtrace full
where full
```

Show call stack and local variables in each frame.

```bash
frame <frame#>
```

Select the stack frame to operate on.

---

## Stepping

```bash
step
```

Next instruction (source line), diving into functions.

```bash
next
```

Next instruction (source line), without diving into functions.

```bash
finish
```

Continue until the current function returns.

```bash
continue
```

Continue normal execution.

---

## Variables and Memory

```bash
print/format <what>
```

Print variable, memory location, or register.

```bash
display/format <what>
```

Like `print`, but repeat after each step.

```bash
undisplay <display#>
enable display <display#>
disable display <display#>
```

Manage automatic display of variables.

```bash
x/nfu <address>
```

Print memory.

* `n`: number of units (default 1)
* `f`: format character
* `u`: unit type (`b` byte, `h` half-word, `w` word, `g` giant word)

### Formats

| Format | Meaning                          |
| ------ | -------------------------------- |
| `a`    | Pointer                          |
| `c`    | Char (integer read as character) |
| `d`    | Signed decimal                   |
| `f`    | Floating point                   |
| `o`    | Octal                            |
| `s`    | C string                         |
| `t`    | Binary                           |
| `u`    | Unsigned decimal                 |
| `x`    | Hexadecimal                      |

### What You Can Print

* Any C expression (including function calls, with proper casts)
* `file_name::variable_name`
* `function::variable_name`
* `{type}address`
* `$register` — e.g., `$esp`, `$ebp`, `$eip`

---

## Threads

```bash
thread <thread#>
```

Choose thread to operate on.

---

## Manipulating the Program

```bash
set var <variable>=<value>
```

Change the content of a variable.

```bash
return <expression>
```

Force the current function to return immediately with a value.

---

## Sources

```bash
directory <directory>
```

Add directory to the source search list.

```bash
list
list <filename>:<function>
list <filename>:<line_number>
list <first>,<last>
```

Show source context.

```bash
set listsize <count>
```

Set how many lines to show in `list`.

---

## Signals

```bash
handle <signal> <options>
```

Set how to handle signals.

Options:

* `(no)print`
* `(no)stop`
* `(no)pass`

---

## Informations

```bash
disassemble [<where>]
```

Disassemble the current function or location.

```bash
info args
info breakpoints
info display
info locals
info sharedlibrary
info signals
info threads
```

Print various kinds of debugging information.

```bash
show directories
show listsize
```

Show settings.

```bash
whatis <variable_name>
```

Print type of named variable.

---

© 2007 Marc Haisenko — [marc@darkdust.net](mailto:marc@darkdust.net)
