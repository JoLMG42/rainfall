# **Bonus 03**

On decompile code:

```
int main(unsigned int v4, struct_0 *a0)
{
    void* v0;  // [bp-0x94], Other Possible Types: char
    char v1;  // [bp-0x53]
    char v2;  // [bp-0x52]
    void* v3;  // [bp-0x10]
    void* v5;  // edi
    unsigned int v6;  // ecx
    unsigned int v7;  // d

    v3 = fopen("/home/user/end/.pass", "r");
    v5 = &v0;
    for (v6 = 33; v6; v5 += v7 * 4)
    {
        v6 -= 1;
        v0 = 0;
    }
    if (!v3)
    {
        return -1;
    }
    else if (v4 == 2)
    {
        fread(&v0, 1, 66, v3);
        v1 = 0;
        (&v0)[atoi(a0->field_4)] = 0;
        fread(&v2, 1, 65, v3);
        fclose(v3);
        if (strcmp(&v0, a0->field_4))
        {
            puts(&v2);
            return 0;
        }
        execl("/bin/sh", "sh");
        return 0;
    }
    else
    {
        return -1;
    }
}
```

On remarque tout desuite 2 choses, l'ouverture du .pass de l'user du dessus et l'execution d'un shell.
On peut aussi voir que le programme prends en parametre un argument.
On peut remarquer sur la `ligne 31 du code`, le placement a `v0` (contenue du fichier .pass apres le fread) de `atoi(av[1])` `v0[atoi(av[1])]` un charactere de fin de chaine `'\0'`.
Le point important suivant est la `comparaison` entre `v0` et notre `argument` qui doivent etre egaux pour l'execution de notre shell sinon on sort du programme.
Or, nous ne connaissons pas le contenue du fichier .pass de l'user end, donc nous allons plutot essayer de "supprimer" le contenue de `v0` en placant un `'\0'` de le debut de la chaine.
Pour faire cela, il faut que le retour de `atoi` soit egal a `0`, or on sait que `atoi renvoie 0`, si on lui envoie une `lettre` par exemple; sauf que si on lui envoie une lettre en argument,
notre chaine v0 ne sera par egale a notre parametre passe en argument.
Pour que notre atoi renvoie 0 et que notre chaine vale `'\0'`, il faut passe en argument du main une chaine vide `""` qui vaudra donc seulement `'\0'`.
Donc maintenant la comparaison retourne bien 0 et cela execute notre shell.

![image](https://github.com/Seriots/rainfall/assets/94530285/39cf12e6-cee8-4230-be0d-07427728b250)
