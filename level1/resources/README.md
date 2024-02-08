# **Level 1**

Pour le `level1` nous avons un executable qui fais juste appelle a gets et il y a aussi une fonction pas appeler qui lance un `system("/bin/sh")`

```
void run(void)

{
  fwrite("Good... Wait what?\n",1,0x13,stdout);
  system("/bin/sh");
  return;
}



void main(void)

{
  char local_50 [76];
  
  gets(local_50);
  return;
}
```

> Avec scp et un decompiler on recupere le code decompiler de l'executable
 
Le but ici va etre de faire un buffer overflow grace a gets pour modifier le register `eip`, celui qui designe la ou est situee la prochaine instruction

Pour ce faire la theorie n'est pas tres complique, voici un magnifique schema explicatif



> ### NEXT : [Level 2](/level2/resources/README.md)
