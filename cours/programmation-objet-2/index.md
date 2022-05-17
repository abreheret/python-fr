---
title: Programmation Orientée Objet 2
author: 
- "[Sébastien Boisgérault](mailto:Sebastien.Boisgerault@minesparis.psl.eu), MINES Paris, Université PSL"
license: "[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)"
date: auto
---

# TODO

  - Polymorphism

  - Latent ("duck") typing

  - Subtyping, inheritance

  - Inheritance / composition / delegation

  - Examples in stdlib (duck typing, hierarchy, subtyping, etc.) :

    - file-like (filesystem, url, string buffer)

    - iterable

    - readline (?)

    - codecs (?)

    - datetime, zoneinfo

    - pickle/copy

    - random

    - tk

    - doctest



# Introduction 🚧 TODO 🚧

# Typage implicite

## Etude de cas

La fonction `copy_file` lit le contenu d'un objet fichier et l'écrit dans un autre.

```python
def copy_file(input, output):
    data = input.read()
    output.write(data)
```

Créons un (tout petit) fichier binaire `image.png` sur notre disque dur

```python
>>> with open("image.png", mode="bw") as image_file:
...     image_file.write(b'\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR\x00\x00\x01\x00\x00\x00\x01\x00\x01\x03\x00\x00\x00f\xbc:%\x00\x00\x00\x03PLTE\xb5\xd0\xd0c\x04\x16\xea\x00\x00\x00\x1fIDATh\x81\xed\xc1\x01\r\x00\x00\x00\xc2\xa0\xf7Om\x0e7\xa0\x00\x00\x00\x00\x00\x00\x00\x00\xbe\r!\x00\x00\x01\x9a`\xe1\xd5\x00\x00\x00\x00IEND\xaeB`\x82')
...
```

puis exploitons `copy_file` pour en créer un copie nommée `image-copy.png`.

```python
>>> input = open("image.png", mode="br")
>>> output = open("image-copy.png", mode="bw")
>>> copy_file(input, output)
``` 

Tout se passe comme prévu ! Néanmoins, on aurait pu faire l'économie de la 
création du fichier initial et créer un objet similaire à un fichier,
mais qui stocke son contenu en mémoire plutôt que sur notre disque dur.


```python
>>> import io
>>> buffer = io.BytesIO(b'\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR\x00\x00\x01\x00\x00\x00\x01\x00\x01\x03\x00\x00\x00f\xbc:%\x00\x00\x00\x03PLTE\xb5\xd0\xd0c\x04\x16\xea\x00\x00\x00\x1fIDATh\x81\xed\xc1\x01\r\x00\x00\x00\xc2\xa0\xf7Om\x0e7\xa0\x00\x00\x00\x00\x00\x00\x00\x00\xbe\r!\x00\x00\x01\x9a`\xe1\xd5\x00\x00\x00\x00IEND\xaeB`\x82')
>>> buffer.seek(0)
```

(L'appel `buffer.seek(0)` repositionne le curseur de lecture/écriture au 
début du fichier.)

On peut alors copier son contenu de la même façon que précédemment

```python
>>> input = buffer
>>> output = open("image-copy.png", mode="bw")
>>> copy_file(input, output)
``` 

En fait l'image originale est une tuile bleue-gris utilisée par
le projet de cartographie OpenStreetMap 
(cf. ["The smallest 256x256 single-color PNG file, and where you've seen it"](https://www.mjt.me.uk/posts/smallest-png/)).

Elle est disponible en ligne à l'adresse <https://www.mjt.me.uk/assets/images/smallest-png/openstreetmap.png>. On aurait donc pu créer un objet similaire à un fichier 
mais qui sait ouvrir des ressources Web plutôt que de recopier à la main son 
contenu.

```python
>>> from urllib.request import urlopen
>>> url = "https://www.mjt.me.uk/assets/images/smallest-png/openstreetmap.png"
>>> input = urlopen(url)
```

A nouveau, la copie entre ce fichier distant et sa copie locale s'effectue 
comme précédemment.

```python
>>> output = open("image-copy.png", mode="bw")
>>> copy_file(input, output)
```

## Protocoles

Ce qui compte dans les trois cas d'usage précédents, ça n'est pas que l'objet
`input` soit un vrai fichier, mais qu'il se comporte comme tel. Ici, très
précisément la fonction `copy_file` a besoin d'un objet `input` qui :

  - à une méthode `read`,

  - qui s'invoque sans argument,

  - et renvoie un objet de type `bytes`.

C'est tout ce que la fonction `copy_file` exige de son argument `input` pour
que ça marche : qu'il soit suffisamment similaire à un "vrai" fichier. 
On ne demande pas à ce qu'il soit d'un type particulier, par exemple 
qu'il valide un test du type `isinstance(input, File)`.

Ce concept moins exigeant de typage, c'est ce qu'en Python on appelle le 
**typage canard** (🇺🇸 **duck typing**) d'après la citation 
attribuée à James Whitcomb Riley

> When I see a bird that walks like a duck and swims like a duck and 
> quacks like a duck, I call that bird a duck. 🦆

(Si je vois un oiseau qui vole comme un canard, nage comme un canard et 
cancane comme un canard, alors j'appelle cet oiseau un canard.)

A noter que pour le moment, les contraintes que doit satisfaire l'argument
`input` de la fonction `copy_file` est uniquement un contrat (moral) entre
le concepteur de la fonction et son utilisateur : tant que l'utilisateur
respecte le contrat, tout se passera comme prévu. On parle parfois de 
**protocole** (🇺🇸 **protocole**) pour faire référence à ce contrat
(ou de **concept** ou encore d'**interface** implicite).

A ce stade, l'interpréteur Python n'est **pas** informé de ce contrat et
ne fait rien de particulier pour assurer que l'engagement mutuel soit
respecté. Il conviendra donc au développeur de la fonction de documenter
ce protocole et à son utilisateur de lire et de le respecter.

## Protocoles standards

## Vérification statique

Il existe des outils qui permettent de formaliser (partiellement) 
les contrats sur lesquels reposent vos programmes, par exemple [mypy](http://mypy-lang.org/).

En contrepartie du travail qui consistera à décrire les protocoles, 
vous disposerez d'un outil qui vous informe de certains violations 
des contrats lors de l'écriture du code, et non bien plus tard, lors de 
son exécution.

Par exemple, on peut formaliser les deux protocoles associés aux arguments de
notre fonction `copy_file` 

```python
from typing import Protocol

class Readable(Protocol):
    def read(self) -> bytes:
        pass

class Writable(Protocol):
    def write(self, data: bytes):
        pass
```

puis annoter le type des arguments de la fonction pour indiquer quel protocole
doit être respecté.

```python
def copy_file(input: Readable, output: Writable):
    data = input.read()
    output.write(data)
```

Si l'on utilise le code client

```python
from urllib.request import urlopen
url = "https://www.mjt.me.uk/assets/images/smallest-png/openstreetmap.png"
input = urlopen(url)
output = open("image-copy.png", mode="bw")
copy_file(input, output)
```

Mypy nous affirmera que de son point de vue, tout va bien

```bash
$ mypy main.py 
Success: no issues found in 1 source file
```

Par contre si l'on se trompe en fournissant par exemple comme second
argument de la fonction `copy_file` un nom de fichier plutôt qu'un 
objet fichier

```python
from urllib.request import urlopen
url = "https://www.mjt.me.uk/assets/images/smallest-png/openstreetmap.png"
input = urlopen(url)
output = open("image-copy.png", mode="bw")
copy_file(input, "image-copy.png")
```

alors mypy nous en informera.

```bash
$ mypy main.py 
main.py: error: Argument 2 to "copy_file" has incompatible type "str"; expected "Writable"
Found 1 error in 1 file (checked 1 source file)
```

# Héritage

What example / base ? Abstract ? Point2D/Point3D? LocalFile/URLFile/InMemoryFile?
(issue with read / write? Do read only?)

TODO:

  - extend: add attributes, methods, special methods

  - replace : override attributes, method

  - blend: call / use parent methods

(explain late dispatch)


# Use case

doctest? With pass for thumbs up in return : pass on thumbs up directive.


<!--
```python
>>> import random
>>> random.seed(0)
>>> random.random()
0.8444218515250481
>>> random.random()
0.7579544029403025
>>> random.gauss(mu=0.0, sigma=1.0)
-0.6797144480784211
>>> random.gauss(mu=0.0, sigma=1.0)
0.3705035674606598
```

```python
>>> random.seed(0)
>>> random.random()
0.8444218515250481
>>> random.random()
0.7579544029403025
>>> random.gauss(mu=0.0, sigma=1.0)
-0.6797144480784211
>>> random.gauss(mu=0.0, sigma=1.0)
0.3705035674606598
```

```python
>>> random.getstate()
(3, (1372342863, 3221959423, 4180954279, 3990540705, 1021773023, 2223090966, 3843864895, 597749037, 4038628808, 2184695301, 2599863370, 1285463076, 3619013288, 3445797418, 662770754, 586125062, 1346770502, 3658746180, 3438111236, 309182431, 3796115977, 3396521978, 406330169, 2387780391, 2469455462, 2266287431, 1699932795, 4049251296, 2880579946, 2212080061, 3732465507, 2085677040, 3780473213, 2666506866, 2546029306, 3062944794, 3831291871, 2699197260, 1538606110, 2628704661, 2899336459, 3451611859, 99931860, 1959678400, 1104806618, 222110430, 2129932650, 956476906, 1123944865, 4148113835, 3692694084, 1144181241, 329767184, 3790223790, 2243139411, 3664375038, 1649616822, 2085955322, 878670707, 427637247, 1097949229, 3413060003, 2868874638, 714350532, 2486509003, 3288554389, 1763249054, 3065280929, 3617934738, 630740700, 76720155, 3508325981, 3932676767, 1329309665, 1100166609, 3150737391, 3713849116, 1027154122, 621059281, 2649419324, 1724153688, 787407208, 2006574677, 2947315466, 1182636319, 3824689780, 1212132099, 4044342334, 315023297, 3553864982, 1157225958, 1432164329, 1160723985, 371272026, 927525248, 1908778666, 1373080387, 1470677278, 801074131, 3914200339, 3381407135, 2878632719, 3924891213, 4265632409, 2538399467, 2281243182, 3528786809, 875255179, 900526794, 3848503840, 1800107094, 99435770, 2252333719, 459600575, 3980788272, 642015534, 3185107417, 3358361764, 109426005, 1526990449, 2897953574, 735827974, 292571356, 656909865, 1811241718, 3852326868, 935626136, 576421325, 428560518, 2760997449, 2373789786, 2483545676, 990053668, 4260816364, 2400672108, 931204375, 1656208850, 1752543171, 3393345042, 2975883932, 3535909646, 441369518, 1875275912, 3788298971, 1604151293, 3265285628, 3389517324, 1129673657, 158637840, 3401368526, 3511603269, 1365694929, 2076135726, 144518747, 1738447909, 3315335210, 696514991, 2903756646, 1692635312, 2492835022, 793066282, 3701291598, 1682146840, 2833764772, 1874922241, 1109375682, 2157966518, 2136578897, 2614256798, 2861779371, 2698885417, 160504417, 554219202, 1430333145, 716754647, 529846619, 3482487583, 829954169, 2479439816, 1934256524, 140815401, 1125214395, 1642506558, 1276896160, 3983009834, 1007760484, 3281113000, 139221400, 361504304, 3713284634, 2928612637, 935745191, 4151018351, 3848439956, 3104510935, 615014456, 2939062496, 1306813042, 2930774231, 1126465550, 39679357, 4157006879, 3487860998, 4211840723, 4199318320, 3081958281, 1231549084, 114041891, 1456998828, 4027354301, 779738931, 2010884098, 3007041435, 1711812332, 1173077090, 2341061349, 2446045442, 3860139934, 3135005041, 1937407922, 4130038091, 4004503321, 3899652703, 1806610961, 3173634158, 3201508203, 1971746461, 2278546448, 2657844960, 2678611962, 2241832425, 1883633391, 3181942897, 2792225916, 1475682425, 1169520747, 176579167, 3554068333, 4262273005, 2352440083, 1253626969, 572672438, 1524846517, 1147813165, 3942527944, 3167059431, 1688983706, 463078297, 3955475903, 894345047, 3894065346, 1892851780, 1869067318, 3416414633, 2408185539, 2584178019, 1536598083, 1738798172, 212705107, 3295820959, 2164521947, 1063447507, 4125226161, 1611652661, 2186239661, 43480999, 3233893377, 1157189060, 3865800845, 2580347146, 3522458126, 3833272017, 2577087932, 3480966526, 3731431004, 1721158498, 1011307417, 3952810466, 2262537757, 642429441, 971181236, 362223105, 1255663603, 146673027, 2069157663, 1273202481, 120747504, 3207942888, 3721730895, 3643503245, 3816306763, 3137048487, 2860458877, 1819115946, 230113293, 3614309992, 916806549, 923982787, 2460427649, 467621168, 934475584, 1085334794, 3075553476, 2827600963, 614120395, 763452512, 3876411276, 42356791, 149721240, 3427418988, 3168682449, 3386540377, 3369227246, 3430314618, 585185811, 3994790629, 1295417371, 3112055284, 3311144040, 1059910922, 3540388763, 455829500, 3061532685, 2182265378, 607235460, 3248562707, 3580560719, 3868768753, 740125315, 4068402071, 4048707696, 3779755980, 2863645708, 3854638905, 3401978190, 2702849333, 2769927328, 1519450755, 84928381, 1224162609, 1520368433, 480115764, 37675693, 657908362, 3808994824, 1171659743, 2699029399, 2015754554, 2753802761, 2050370709, 1273170742, 2140991184, 4266961092, 1786701666, 3880429141, 759425704, 45926880, 836647469, 1573125967, 1163571132, 203220878, 861333837, 4060383179, 2628819199, 3536629941, 2457334432, 1867577917, 3536511998, 3697335810, 3133380389, 1404482804, 881246958, 4135024728, 534752881, 595543441, 33441746, 4269857928, 1314335199, 2505095316, 1578860014, 1875057095, 509159986, 1944155236, 70374715, 2945217752, 4041521547, 2966052890, 145655727, 714593746, 1002319513, 519269594, 2926922359, 944143625, 3916879502, 2378371484, 912744193, 3910705684, 2276700770, 4142358138, 3964824208, 2780717615, 4081508385, 236938235, 3840831868, 647425534, 2210040950, 2279462069, 168097603, 1461160443, 1580267216, 2324105764, 2690677560, 1690421599, 3577506112, 738462960, 2274273062, 2504137643, 3265350434, 1366106822, 2974016022, 1988817242, 797755685, 2403760280, 641540828, 3788701957, 2779856144, 2104598935, 3745352634, 1577129078, 1373804927, 3903547821, 2872229328, 3056383831, 3072291562, 2275982065, 1522674624, 3582423212, 3669935104, 3633938225, 629521463, 1238893987, 3075431842, 813637316, 3510510638, 2917340917, 2768681237, 1942077161, 3758057471, 1933876081, 2244433730, 1480577031, 1351970442, 396252158, 4175666695, 1357894203, 234330270, 3181220453, 1331961817, 784329267, 779668731, 2328048439, 1103091157, 3004928118, 1162468270, 3007103493, 2725514842, 274062491, 3149489948, 2987599126, 3292515534, 282168320, 959822509, 2291575880, 3673412248, 162691783, 1230667779, 2485346968, 2008336037, 3320353719, 3822671307, 3089220377, 3503681079, 1302975933, 3327766732, 3674135615, 1073069922, 2772807178, 3196662328, 1713328812, 1475744247, 1685385181, 1643753643, 3794162811, 2190568646, 2689827007, 3126829736, 2761526147, 2511266684, 1782816864, 2926893516, 3512983057, 2661970973, 2689095779, 1199154380, 382426272, 643849114, 807267160, 1518924385, 1695080083, 2934795830, 4181065090, 3342589537, 2406648073, 1186040699, 2721983068, 2970957157, 3556085447, 3070386458, 3046088386, 1804573955, 2003863284, 4166932229, 2182095175, 2134701977, 911628328, 2867572258, 1276531023, 518250163, 2617826095, 21746525, 2080648289, 1970004495, 2621101460, 3726800574, 155341573, 2577500508, 1723761283, 3633471938, 222521000, 2717308343, 4013182446, 2258756213, 771120786, 815922213, 3754576141, 2998487214, 3115409480, 3002705922, 1074978656, 3006531599, 823178279, 416424292, 43442668, 1726938224, 3934805529, 161873912, 2803348707, 1043589181, 102006005, 1730373835, 1210399249, 2434080783, 2191863758, 3057527658, 3614195070, 3229819907, 334710481, 2664502177, 1325604787, 3988671798, 2576445889, 2959706600, 2017397749, 1080624230, 903812120, 1214058725, 1799750730, 3417289034, 807297012, 1437878312, 2645448696, 2089144691, 1348214076, 2839011040, 648098687, 4294223352, 3774352159, 968459174, 396311303, 1631056285, 3504308190, 1279074970, 1878080429, 2991953531, 3355603528, 1614988643, 388314152, 964861631, 3836107078, 126268069, 1350108355, 1730114235, 926577527, 430314525, 756676482, 2364788864, 3573266150, 2877606665, 2554592887, 2297285047, 4155839037, 1342607531, 1587367306, 1366061151, 1625602653, 1519619151, 150082289, 2420444017, 2103580590, 1875045396, 3514756929, 3527524385, 3119378202, 418789356, 8), None)
```

```python
>>> state = random.getstate()
```

```python
>>> random.gauss(mu=0.0, sigma=1.0)
-1.016348894188071
>>> random.gauss(mu=0.0, sigma=1.0)
-0.07212002278507135
```

```python
>>> random.setstate(state)
>>> random.gauss(mu=0.0, sigma=1.0)
-1.016348894188071
>>> random.gauss(mu=0.0, sigma=1.0)
-0.07212002278507135
```

### Interface orientée objet

```python
>>> import random
>>> r = random._inst
>>> type(r)
<class 'random.Random'>
>>> r.seed(0)
>>> r.random()
0.8444218515250481
>>> r.random()
0.7579544029403025
>>> r.gauss(mu=0.0, sigma=1.0)
-0.6797144480784211
>>> r.gauss(mu=0.0, sigma=1.0)
0.3705035674606598
```

> Class Random can also be subclassed if you want to use a different basic generator of your own devising: in that case, override the random(), seed(), getstate(), and setstate() methods.


```python
import math

class IHaveNoIdeaWhatIAmDoingRandom(random.Random):
    def __init__(self, seed=None):
        self.seed(seed)
    def seed(self, number=None):
        if number is None:
            number = 0.0
        state = abs(number) % 1.0
        self.setstate(state)
    def getstate(self):
        return self._state
    def setstate(self, state):
        self._state = state
    def random(self):
        value = self.getstate()
        self.setstate((value + math.pi) % 1.0)
        return value
```
-->