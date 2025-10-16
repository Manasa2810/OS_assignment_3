# OS_assignment_3

##  Question 1 – Print Page Table Entries

###  Aim

To add a new system call that prints the page table entries (PTEs) for the **first 10** and **last 10** pages of a process.

---

###  Files Modified

**1. `sysproc.c`**

* Added a new system call `sys_pgpte()` that takes a virtual address and returns its **page table entry**.
* Used `getpte()` from `vm.c` to find the entry in the current process’s page directory.

```c
int sys_pgpte(void) {
  uint va;
  pde_t *pgdir = proc->pgdir;
  if (argint(0, (int*)&va) < 0)
    return -1;
  pte_t *pte = getpte(pgdir, (void*)va);
  if (pte == 0)
    return 0;
  return *pte;  // return raw PTE value
}
```

**2. `vm.c`**

* Added a helper function `getpte()` to walk the page table and return the pointer to the PTE for a given virtual address.
* This was used by the new syscall above.

**3. `syscall.h`**

* Added new syscall numbers:

  ```c
  #define SYS_pgpte   22
  #define SYS_kpt     23
  #define SYS_ugetpid 24
  ```

**4. `syscall.c`**

* Added the syscall function pointer to the `syscalls[]` table:

  ```c
  [SYS_pgpte]   sys_pgpte,
  [SYS_kpt]     sys_kpt,
  [SYS_ugetpid] sys_ugetpid,
  ```

**5. `user.h` / `usys.S`**

* Added user-level stubs for `pgpte`, `kpt`, and `ugetpid`.

  ```c
  int pgpte(void *va);
  ```

**6. `test.c`**

* modified test program that calls `pgpte()` for the first and last 10 pages using the helper functions `print_pt()` and `print_pte()`.
* The program prints each virtual address, PTE, physical address, and access permission (AP bits).

---

We modified the test.c to the version of our os.
Output Example
VA 0x00000000 -> PA 0x00101000 | PTE=0x00101007 | Flags: P W U
VA 0x00001000 -> PA 0x00102000 | PTE=0x00102007 | Flags: P W U
...
VA 0xBFFFF000 -> PA 0x002A4000 | PTE=0x002A4005 | Flags: P U

Explanation of Bits
Bit	Symbol	Meaning
0	P	Page Present in Memory
1	W	Writable
2	U	User Accessible
3–11	—	OS/Reserved bits
12–31	Physical Page Frame Address	Base physical address of the page
Interpretation

The first 10 pages belong to the program’s code, data, and stack.

The last 10 pages usually include kernel space mappings (often only P bit set, no U access).

PTE=0 means the virtual page is not mapped.

