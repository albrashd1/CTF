#### Q: Can you figure out how this program works to get the flag?
First, they gave us two files, a source code, and a binary file. Let's download them to read the source code and test the executable file. 

Here is the source code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>


void print_segf_message(){
  printf("Segfault triggered! Exiting.\n");
  sleep(15);
  exit(SIGSEGV);
}

int win() {
  FILE *fptr;
  char c;

  printf("You won!\n");
  // Open file
  fptr = fopen("flag.txt", "r");
  if (fptr == NULL)
  {
      printf("Cannot open file.\n");
      exit(0);
  }

  // Read contents from file
  c = fgetc(fptr);
  while (c != EOF)
  {
      printf ("%c", c);
      c = fgetc(fptr);
  }

  printf("\n");
  fclose(fptr);
}

int main() {
  signal(SIGSEGV, print_segf_message);
  setvbuf(stdout, NULL, _IONBF, 0); // _IONBF = Unbuffered

  unsigned int val;
  printf("Enter the address in hex to jump to, excluding '0x': ");
  scanf("%x", &val);
  printf("You input 0x%x\n", val);
  void (*foo)(void) = (void (*)())val;
  foo();
}
```

There are three functions, first `print_segf_message()`, `win()` and `main()`. 
- `print_segf_message()` for printing the segmentation fault error
- `win` function is going to open a file and print what in that file, else if it will say the file is not available. 
- `main` function will ask user for an address and then it will go to that address. Note that it will call a `foo()` function which is being declared in line `void (*foo)(void)`, basically saying declare a foo function pointer that takes no parameter and return nothing (void). Now that equal `(void (*)())val`, which means initialize the function pointer foo with the `val` value. Declaration and initializing in this way called casting, it will convert the value of `val` to a function pointer and when we call `foo()` it will take us to the value of `val`. So, simply, anything we type in, the program will take us to that address. We want to jump to `win()` function. We can use GDB to het the address of `win()`. 
```bash
gef➤  info functions
All defined functions:

Non-debugging symbols:
0x0000000000401000  _init
0x00000000004010e0  putchar@plt
0x00000000004010f0  puts@plt
0x0000000000401100  fclose@plt
0x0000000000401110  printf@plt
0x0000000000401120  fgetc@plt
0x0000000000401130  signal@plt
0x0000000000401140  setvbuf@plt
0x0000000000401150  fopen@plt
0x0000000000401160  __isoc99_scanf@plt
0x0000000000401170  exit@plt
0x0000000000401180  sleep@plt
0x0000000000401190  _start
0x00000000004011c0  _dl_relocate_static_pie
0x00000000004011d0  deregister_tm_clones
0x0000000000401200  register_tm_clones
0x0000000000401240  __do_global_dtors_aux
0x0000000000401270  frame_dummy
0x0000000000401276  print_segf_message
0x000000000040129e  win
0x0000000000401334  main
0x00000000004013d0  __libc_csu_init
0x0000000000401440  __libc_csu_fini
0x0000000000401448  _fini
```

We can see that the `win()` is in 0x40129e. 

Giving this address without (0x) to the program will print the flag. 

![[picker.png]]
