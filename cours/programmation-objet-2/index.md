---
title: Programmation Orientée Objet 2
author: 
- "[Sébastien Boisgérault](mailto:Sebastien.Boisgerault@minesparis.psl.eu), MINES Paris, Université PSL"
license: "[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)"
date: auto
---

# 🚧 TODO 🚧

  - Polymorphism

  - ~~Latent/structural ("duck") typing~~

  - ~~Subtyping, inheritance~~ `issubclass`

  - Inheritance vs composition vs delegation


# Introduction 

🚧 TODO 🚧

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

🚧 TODO 🚧

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

Considérons à nouveau notre classe de nombres complexes "maison".

```python
class Complex:
    def __init__(self, real, imag):
        self.set_real(real)
        self.set_imag(imag)
    def get_real(self):
        return self._real
    def set_real(self, real):
        if isinstance(real, float):
            self._real = real
        else:
            raise TypeError(f"{real!r} is not a float")
    real = property(get_real, set_real)
    def get_imag(self):
        return self._imag
    def set_imag(self, imag):
        if isinstance(imag, float):
            self._imag = imag
        else:
            raise TypeError(f"{imag!r} is not a float")
    imag = property(get_imag, set_imag)
    def conjugate(self):
        return Complex(self._real, -self._imag)
    def __repr__(self):
        # ⚠️ weird output when self.imag < 0
        return f"({self._real}+{self._imag}j)"
    def __add__(self, other):
        return Complex(
            self._real + other._real, 
            self._imag + other._imag
        )
```

Nous allons essayer de nous doter d'une nouvelle classe de nombres complexes,
`Complex2` dont les instances auront un comportement qui nous convient mieux, 
sans modifier le code source de `Complex`, mais en exploitant ses fonctionnalités
au maximum.

Pour cela, nous allons dériver la class `Complex2` de la classe `Complex`.
Au minimum, cela signifie une déclaration de la forme

```python
class Complex2(Complex):
    pass
```

A ce stade, pour l'essentiel, pas de changement dans le comportement des
nombres complexes qui en sont les instances, car toutes les méthodes
de `Complex2` sont héritées de celles de `Complex` :

```python
>>> z = Complex2(0.5, 1.5)
>>> z
(0.5+1.5j)
>>> z.real
0.5
>>> z.real = -0.5
>>> z
(-0.5+1.5j)
>>> z.real = "Hello"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 11, in set_real
TypeError: 'Hello' is not a float
>>> w = Complex2.conjugate(z)
>>> w.real
0.5
>>> w.imag
-1.5
>>> z + z.conjugate()
(1+0j)
```

On a même

```python
>>> isinstance(z, Complex)
True
```

Seuls changements visibles, on a désormais

```python
>>> type(z) is Complex
False
>>> type(z) is Complex2
True
>>> isinstance(z, Complex2)
```

Ce qui motive au départ l'introduction d'une nouvelle classe de nombres
complexes, c'est que l'on a oublié d'implémenter la multiplication :

```python
>>> Complex(1.0, 0.0) * Complex(0.0, 1.0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for *: 'Complex' and 'Complex'
```

Réparons cet oubli en ajoutant une méthode `__mul__` à la classe `Complex2`

```python
class Complex2(Complex):
    def __mul__(self, other):
        r1, i1 = self.real, self.imag
        r2, i2 = other.real, other.imag
        real = r1*r2 - i1*i2
        imag = r1*i2 + r2*i1
        return Complex2(real, imag)
```

```python
>>> Complex2(1.0, 0.0) * Complex2(0.0, 1.0)
(0.0+1.0j)
```

C'est mieux ! Il y a en fait un subtil bug (voyez-vous lequel ?) mais nous
allons attendre un peu pour le corriger, nous serons bientôt mieux placés
pour corriger le problème.

En attendant, nous allons faire en sorte que notre constructeur soit un peu
plus polyvalent ; nous aimerions bien pouvoir construire un nouvel objet
complex à partir de tout objet qui possède des attributs numériques
`real` et `imag`, par exemple, un nombre complexe intégré, instance de
la class `complex`. Avec la classe `Complex`, cela ne marche pas :

```python
>>> Complex(0.5+1.5j)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: __init__() missing 1 required positional argument: 'imag'
```

et pas plus avec la classe `Complex2` :

```python
>>> Complex2(0.5+1.5j)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: __init__() missing 1 required positional argument: 'imag'
```

En effet, en l'absence de constructeur `__init__` qui lui soit propres,
les nouveaux complexes sont instanciées au moyen de la méthode `__init__`
héritée.

Mais on peut définir un nouveau constructeur `__init__` qui aura la 
priorité. Pour ce faire, on teste si le premier argument nommé
`real_or_complex` à des champs `real` et `imag`. Sinon c'est le
cas on l'interprête comme un nombre complexe ; dans le cas contraire un
utilise cet argument comme partie réelle et le second comme partie imaginaire.

```python
class Complex2(Complex):
    def __init__(self, real_or_complex, imag=None):
        try:
            real = real_or_complex.real
            imag = real_or_complex.imag
        except AttributeError:
            real = real_or_complex
            imag = imag
        self.real = real
        self.imag = imag
    def __mul__(self, other):
        r1, i1 = self.real, self.imag
        r2, i2 = other.real, other.imag
        real = r1*r2 - i1*i2
        imag = r1*i2 + r2*i1
        return Complex2(real, imag)
```

On notera que les deux dernières lignes du constructeurs sont un copié-collé
du code du constructeur parent. Autant faire appel directement à celui-ci !
On pourra au choix utiliser la syntaxe explicite 
`Complex.__init__(self, real, imag)` ou la construction `super()`
comme ci-dessous :


```python
class Complex2(Complex):
    def __init__(self, real_or_complex, imag=None):
        try:
            real = real_or_complex.real
            imag = real_or_complex.imag
        except AttributeError:
            real = real_or_complex
            imag = imag
        super().__init__(real, imag)
    def __mul__(self, other):
        r1, i1 = self.real, self.imag
        r2, i2 = other.real, other.imag
        real = r1*r2 - i1*i2
        imag = r1*i2 + r2*i1
        return Complex2(real, imag)
```

Désormais, le constructeur de `Complex2` accepte les arguments complexes :

```python
>>> Complex2(0.5+1.5j)
(0.5+1.5j)
>>> Complex2(Complex(0.5, 1.5))
(0.5+1.5j)
```

Il est temps de revenir à notre subtil bug. En héritant la méthode `__add__`
de la classe parent `Complex`, on va malheureusement toujours obtenir une
instance de `Complex` quand on additionne des instances de `Complex2`.

```python
>>> z = Complex2(0.5, 1.5)
>>> w = z + z
>>> type(w)
<class '__main__.Complex'>
```

Il est possible de corriger cela directement en réimplémentant `__add__`
dans la class dérivée

```python
class Complex2(Complex):
    def __init__(self, real_or_complex, imag=None):
        try:
            real = real_or_complex.real
            imag = real_or_complex.imag
        except AttributeError:
            real = real_or_complex
            imag = imag
        super().__init__(real, imag)
    def __add__(self, other):
        return Complex2(
            self.real + other.real, 
            self.imag + other.imag
        )
    def __mul__(self, other):
        r1, i1 = self.real, self.imag
        r2, i2 = other.real, other.imag
        real = r1*r2 - i1*i2
        imag = r1*i2 + r2*i1
        return Complex2(real, imag)
```

Ca marche, mais cela revient à perdre le bénéfice de ce qui a déjà été implémenté. 
On peut être plus subtil, appeler la méthode de la classe parente pour 
l'addition et corriger à posteriori le type du résultat, avec notre
constructeur flambant neuf :

```python
class Complex2(Complex):
    def __init__(self, real_or_complex, imag=None):
        try:
            real = real_or_complex.real
            imag = real_or_complex.imag
        except AttributeError:
            real = real_or_complex
            imag = imag
        super().__init__(real, imag)
    def __add__(self, other):
        # ℹ️ sum = Complex.__add__(self, other) would also work.
        sum = super().__add__(other) 
        return Complex2(sum)
    def __mul__(self, other):
        r1, i1 = self.real, self.imag
        r2, i2 = other.real, other.imag
        real = r1*r2 - i1*i2
        imag = r1*i2 + r2*i1
        return Complex2(real, imag)
```

Et désormais la somme se comporte comme prévu

```python
>>> z = Complex2(0.5, 1.5)
>>> w = z + z
>>> type(w)
<class '__main__.Complex2'>
```

Au passage, remarquons que si une génération future de développeur doit 
reprendre notre travail et introduire une classe `Complex3` qui dérivera
de `Complex2`, ils vont être confrontés au même problème. Pour leur
faciliter la vie, on peut utiliser un code qui va adapter le type
de la valeur renvoyée au type de `self` et qui pourra donc être
héritée telle quelle dans `Complex3`.

```python
class Complex2(Complex):
    def __init__(self, real_or_complex, imag=None):
        try:
            real = real_or_complex.real
            imag = real_or_complex.imag
        except AttributeError:
            real = real_or_complex
            imag = imag
        super().__init__(real, imag)
    def __add__(self, other):
        ComplexType = type(self)
        sum = super().__add__(other)
        return ComplexType(sum)
    def __mul__(self, other):
        ComplexType = type(self)
        r1, i1 = self.real, self.imag
        r2, i2 = other.real, other.imag
        real = r1*r2 - i1*i2
        imag = r1*i2 + r2*i1
        return ComplexType(real, imag)
```



# La bibliothèque standard

## `pathlib`

🚧 TODO 🚧

## `random`

### Introduction

Le module de la bibliothèque Python standard `random` permet de générer des 
nombres pseudo-aléatoires.

```python
>>> import random
```

La fonction `random` du module va générer des nombres à virgule flottante 
uniformément distribués entre 0 et 1.

```python
>>> random.random()
0.17288416418484898
>>> random.random()
0.7270494197615684
>>> random.random()
0.22967289202282093
```

De multiples fonctions sont fournies pour générer des nombres pseudo-aléatoires
suivant des distributions de probabilité diverses. Par exemple, pour générer
des nombres distribués selon la gaussienne d'espérance $\mu = 0.0$ et 
d'écart-type $\sigma = 1.0$, on peut invoquer

```python
>>> random.gauss(mu=0.0, sigma=1.0)
0.7010040262172509
>>> random.gauss(mu=0.0, sigma=1.0)
0.11430668630347102
>>> random.gauss(mu=0.0, sigma=1.0)
-0.49389733826503307
```


### Interface orientée objet

L'étude du fichier source [random.py](https://github.com/python/cpython/blob/3.10/Lib/random.py)
nous informe que l'interface classique du module n'est qu'un fin vernis au-dessus
d'une architecture objet. Le module définit une classe `Random`, puis crée une
instance privé `_inst` dans ce module. Les "fonctions" du module `random`
comme `gauss` sont simplement des raccourcis vers les méthodes de cette instance

```python
>>> random.random
<built-in method random of Random object at 0x55a5a09ad260>
>>> random.gauss
<bound method Random.gauss of <random.Random object at 0x55a5a09ad260>>
>>> r = random._inst
>>> type(r)
<class 'random.Random'>
>>> r.random
<built-in method random of Random object at 0x55a5a09ad260>
>>> r.gauss
<bound method Random.gauss of <random.Random object at 0x55a5a09ad260>>
```

La méthode `random` utilisée par défaut génère des nombres entiers aléatoires 
compris entre $0$ et $2^{53} - 1$ (la probabilité de chaque entier étant
identique), puis divise le résultat par $2^{53}$. Inconvénient de cette
approche : `random()` renvoie une grandeur qui est toujours un multiple
de $2^{-53}$. Le nombre flottant $2^{-1074}$ par exemple, qui est le plus
petit nombre flottant strictement positif n'a aucune chance d'être produit.

```python
>>> r.random() * 2**53
4346481833061509.0
>>> r.random() * 2**53
6826402970501312.0
>>> r.random() * 2**53
5570978756682725.0
```

Si c'est un problème pour vous, il est possible de corriger ce comportement
comme le suggère la [documentation du module `random`](https://docs.python.org/fr/3/library/random.html#recipes) en définissant une classe dérivée de `Random` qui
surcharge la méthode `random`

```python
from math import ldexp

class AltRandom(random.Random):
    def random(self):
        mantissa = 0x10_0000_0000_0000 | self.getrandbits(52)
        exponent = -53
        x = 0
        while not x:
            x = self.getrandbits(32)
            exponent += x.bit_length() - 32
        return ldexp(mantissa, exponent)
```

L'usage est immédiat

```python
>>> r = AltRandom()
>>> r.random()
0.2768487552410033
>>> r.random()
0.08881389087065399
>>> r.random()
0.28173863914986846
```


Les valeurs produites par la méthode `random` ne sont plus nécessairement
des multiples de $2^{-53}$ (il y a néanmoins un peu plus d'une chance sur
deux que cela soit le cas).
```python
>>> r.random() * 2**53
6118147054761291.0
>>> r.random() * 2**53
1809975186779188.8
>>> r.random() * 2**53
6828617072759119.0
```

Les autres distributions de probabilités exploitant la méthode `random`
comme source de valeurs aléatoires, nous n'avons pas besoin de réimplémenter
quoi que ce soit d'autre pour bénéficier très largement de cette source
aléatoire améliorée.

```python
>>> r.gauss(mu=0.0, sigma=1.0)
-0.28865100238160024
>>> r.gauss(mu=0.0, sigma=1.0)
-0.5190938357947126
>>> r.gauss(mu=0.0, sigma=1.0)
1.0356452612439027

```

## `doctest`

🚧 TODO 🚧

doctest? With pass for thumbs up in return : pass on thumbs up directive.


