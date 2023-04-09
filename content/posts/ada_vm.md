---
title: "Write your own Virtual Machine in 62 lines of Ada"
summary: This tutorial guides you to create a simple VM using Ada
author: "M.V.Harish kumar"
date: 2023-04-09T12:21:33+05:30
tags: ["VM", "Ada", "SUBLEQ"]
draft: false
---

# What is Virtual Machine(VM)

For those who never had an Idea about it, here is a short excrept from wikipedia

> In computing, a "virtual machine" (VM) is the virtualization or emulation of a computer system.  Virtual 
> machines are based on computer architectures and provide the functionality of a physical computer.

Thus, a Virtual Machine is basically a software which emulates the functionality of a computer system. It 
basically translates the instructions of another computer architecture to run on the actual host. This was 
very helpful in many ways:

- They help in running softwares which are only **native to a specific processor** (eg: running x86 on ARM).
- Helps in making **old softwares usable** (eg: running old DOS games on a Windows 10/Android).
- Helps in **Easy Backup** and **Isolation** which are very helpful during software testing.

## Why Ada?

Ada is a Programming language created for making **reliable**, **efficient** and safe for use in 
**Mission critical systems**. It is created for use in Defense and Aerospace technology. But now, it's usage 
is getting low due to the advent of programming language such as Python, Java, etc...

Also, I made this VM using Ada to give it a try. Why not Ada? being such a type-safe language, it has all the 
capabilities to create a virtual machine

# The SUBLEQ VM

Virtual Machine softwares, basically take a program written in assembly instruction of another language and 
interprets it to run on the actual architecture. So, to implement a VM, we need to choose a architecture 
amongst the following:

- **CISC**: Complex Instruction Set Computer, they have a large fleet of instruction to implement(eg: Intel's x86).
- **RISC**: Reduced Instruction Set Computer, have less no. of instructions to implement(eg: ARM, IBM's PowerPC).
- **OISC**: One Instruction Set computer, has only one instruction to implement.

The OISC architecture is also known as URISC(Ultimate RISC) architecture. This is still a theoretical 
concept, but implemented virtually. Such a instruction used in real-life would definitely create highly 
powerful, fast systems.

There are many instructions which support this architecture. One such instruction is SUBLEQ(**SU**btract and 
**B**ranch if **L**ess than or **EQ**ual to zero). It corresponds to this C program:

```c
Mem[dest] = Mem[dest] - Mem[src]
if (Mem[dest] <= 0) goto branch
```

Where `src`, `dest` and `branch` are arguments of the SUBLEQ instruction. Also, we need to note that:
- If `src = -1`, then the program should input a character and save it in `Mem[dest]`.
- If `dest = -1`, then the program should output a character stored in `Mem[src]`.

This architecture is choosen to build a simple, but fast VM which you can make it within 15 minutes(includes 
logical understanding of program).

# Prerequisites

To make this VM, you just need a Ada compiler(I used [GNAT Ada]((https://www.gnu.org/software/gnat)) 
compiler) and basic knowledge about Ada programming langauge.

# Setup

Just `cd` in your project folder and create a directory for the project by running the following commands

```bash
$ mkdir memoria
$ cd memoria
```

In the above commands, we create a directory to store our VM(I'm calling it Memoria) and make it as our 
current directory. Now create a file named `vm.adb` and open it using your favourite editor.

# Package Imports and Declarations

Let us start the program by importing necessary packages.

```ada
with Ada.Text_IO; use Ada.Text_IO;
with Ada.Command_Line; use Ada.Command_Line;
with Ada.Integer_Text_IO;

procedure VM is
    package IntIO renames Ada.Integer_Text_IO;

    type Code_Arr is array (0..255) of Integer;

    Program     : File_Type;
    Num, Mindex : Integer := 0;
    Memory      : Code_Arr := (others => 0);
```

The first 3 lines of this code is simple. It just imports `Ada.Text_IO`, `Ada.Integer_Text_IO` for IO 
operations and `Ada.Command_Line` for accessing command-line arguments. Also, we rename `Ada.Integer_Text_IO` 
into `IntIO` to create distinction between `Ada.Text_IO`'s functions.

Then we declare a type `Code_Arr` which is a simple array and it has 256 locations(i.e., 32 byte). The 
variable `Program`, `Num` and `Mindex` are created to load our program from file to program memory(i.e., into 
`Memory` variable)

# Making the VM

The SUBLEQ VM is internally created as a Procedure:

```ada
procedure Memoria (code: in out Code_Arr) is
        IP, NextIP : Integer := 0;
        function Src return Integer is (code(IP));
        function Dest return Integer is (code(IP + 1));
        function Branch return Integer is (code(IP + 2));
```

We create a procedure for our VM and declare the following:

- Variables `IP` to store the pointer(i.e., location) of current instruction in memory and `NextIP` to store the upcoming instruction pointer.
- Functions `Src`, `Dest` and `Branch` corresponds to the 3 instructions of SUBLEQ

```ada
begin
        while IP >= 0 loop
            NextIP := NextIP + 3;
```

As we begin our function, we create a while loop which runs until `IP` is a whole number(which means to exit 
the program we need to change `IP` to `-1`). Also, we increment the `NextIP` variable by `3`.

```ada
            if Src = -1 then
                declare
                    Ch: Character;
                begin
                    Get(Ch);
                    code(Dest) := Character'Pos(Ch);
                end;
```

The above code handles input(i.e., If src = -1 case), it inputs a cheracter and store it in variable `Ch` 
temporarily and sets memory location pointed by `Dest` to ascii value of `Ch`.

```ada
            elsif Dest = -1 then
                Put(Character'Val(code(Src)));
```

This code is self-explanatory, It outputs the character for its ascii value stored in memory pointed by `Src`.

```ada
            else
                code(Dest) := code(Dest) - code(Src);
                if code(Dest) <= 0 then
                    NextIP := Branch;
                end if;
            end if;
            IP := NextIP;
        end loop;
        exception
            when others => Put_Line("Memoria Program Execution Error");
    end Memoria;
```

Here, we check for our actual condition which we need to implement the SUBLEQ instruction. If it is true, we 
change the value of `NextIP` to the `Branch` pointer and finally assign it to the current `IP`. Also, we 
include a `exception` to catch up error thrown while running any specific SUBLEQ program.

By the end of this procedure, you've a functional SUBLEQ VM(but it needs some interface with real os).

# Reading stored Programs

Now, we're ready with a VM. But we need to access the program files to execute it. Thus, we need to read the 
program file and load it into our memory.

```ada
begin
    if Argument_Count < 1 then
        Put_Line("Memoria VM needs atleast one CMDline argument");
        Put_Line("Usage: ./memoria <fname>");
    else
        Open(Program, In_File, Argument(1));

        while not End_Of_File(Program) loop
            IntIO.Get(Program, Num);
            Memory(Mindex) := Num;
            Mindex := Mindex + 1;
        end loop;

        Close(Program);
        Memoria(Memory);
    end if;
end VM;
```

This is our main function(which gets executed when running the executable). In the first couple of lines, we 
include the code to handle if argument haven't supplied by the user. Then, we do the following:

- `Open` the file provided by the user at the first argument(`Argument(1)`) with `In_File`(read) functionality.
- Read the file until EOF and read every integer seperated by whitespace and store it temporarily in `Num`.
- Load each `Num` into Memory pointed by `Mindex`.
- Increment the memory index pointed by `Mindex` by 1.
- `Close` the `Program` file and pass the memory to our `Memoria` function to execute it.

Now we have a fully functional VM which can execute programs stored in file.

# Running "Hello, world!"

The first program to test in our VM is to print `"Hello, world!"` on to the screen. For that I've a premade [hello_world program](https://github.com/harishtpj/Memoria/blob/master/hello.txt). Just download it, and pass the file as a commandline argument to the program.

To run the VM, execute the following commands

```bash
$ gnatmake vm.adb -o memoria
$ ./memoria hello.txt
```

This command would execute the `"Hello, world!"` program which is stored in `hello.txt` file.

# Mission accomplished

Yay! now you've made a VM in just 62 lines of Ada. You can check out the source of this project [here](https://github.com/harishtpj/Memoria). This VM can execute more complex commands using multiple SUBLEQ instructions. If you’re interested in this topic and want to expand more, there is a lot of resources out there on the internet. I've also personally wrote a [More complex VM](https://github.com/harishtpj/RAM-VM) and also a [CHIP-8](https://github.com/harishtpj/Chip8-CPU) emulator.

If you've liked this article, [star](https://github.com/harishtpj/Memoria) my memoria github repository. If you need to suggest any changes, feel free to create an [Issue](https://github.com/harishtpj/Memoria/issues) at my repo.

Thanks for Reading!
