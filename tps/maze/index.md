---
title: Labyrinthes
author: 
- "[Sébastien Boisgérault](mailto:Sebastien.Boisgerault@minesparis.psl.eu), MINES Paris -- PSL"
license: "[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)"
date: auto
---

!["Maze" par [Mitchell Luo](https://unsplash.com/photos/z1c9juteR5c) sur [Unsplash](https://unsplash.com/)](images/mitchell-luo-z1c9juteR5c-unsplash.jpg)

Labyrinthes
--------------------------------------------------------------------------------

Nous nous intéressons aux labyrinthes définis au sein d'une grille 30x30.
Nous les représenterons en Python par des ensembles des paires d'entiers 
compris entre 0 et 29, paires qui désignent les coordonnées des cellules 
vides (traversables) du labyrinthe. 
La paire `(0,0)` correspond au coin en haut à gauche de la grille.


![Un labyrinthe généré (pseudo-)aléatoirement avec 25% de murs 
(cellules vides en blanc, murs en noir).](images/random-maze.jpg)

### Bibliothèque de labyrinthes

  - Examinez les fichiers du dossier 📁 [mazes](https://github.com/boisgera/python-fr/tree/master/tps/maze/mazes).

  - Chargez le texte des ces fichiers.
  
  - Reconstituez les labyrinthes associés.

<details>
<summary>
**Solution**
</summary>

Par exemple, pour obtenir le labyrinthe du fichier `"random-maze.py"`:

```python
filename = "mazes/random-maze.py"
file = open(filename, mode="r", encoding="utf-8")
random_maze_repr = file.read()
file.close()
random_maze = eval(random_maze_repr)
```

</details>

### Visualisation

Développez avec pygame une fonction `display_maze` de visualisation de labyrinthe.


<details>
<summary>
**Solution**
</summary>

```python
# Pygame
import pygame as pg


# Constants
WIDTH, HEIGHT = 30, 30
CELL_SIZE = 20
FPS = 1
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

def draw_background(screen):
    screen.fill(BLACK)

def draw_walls(screen, maze):
    h = CELL_SIZE
    for x, y in maze:
        pg.draw.rect(screen, WHITE, (x * h, y * h, h, h))

def display_maze(maze):
    pg.init()
    pg.display.set_caption("Labyrinthes")
    width_height = (WIDTH * CELL_SIZE, HEIGHT * CELL_SIZE)
    screen = pg.display.set_mode(width_height)
    clock = pg.time.Clock()
    while True:
        events = pg.event.get()
        if any(event.type == pg.QUIT for event in events):
            break
        draw_background(screen)
        draw_walls(screen, maze)
        pg.display.update()
        clock.tick(FPS)
    pg.quit()
```

</details>


### Créez vos propres labyrinthes

... puis ajoutez-les à la biblothèque existante !

<details>
<summary>
**Solution**
</summary>

Une fois votre labyrinthe `my_maze` conçu selon vos préférences

```pycon
>>> my_maze_repr = repr(my_maze)
>>> file = open("mazes/my-maze.py", "w", encoding="utf-8")
>>> file.write(my_maze_repr)
>>> file.close()
```

</details>


Graphes et chemins
--------------------------------------------------------------------------------


🏷️ Un **graphe** (orienté et pondéré) est un triplet composé :

  - d'un ensemble de **sommets** ou **noeuds** (🇺🇸 **vertices**),

  - d'un ensemble d'**arêtes** (orientées) ou **arcs** (🇺🇸 **edges**). 
    Une arête orientée est une paire composée d'un sommet
    source et d'un sommet cible.

  - d'une collection associant à chaque arête une valeur numérique appelée **poids** (🇺🇸 **weight**).

🏷️ Un **chemin** d'un graphe est une suite de sommets du graphe tels que 
chaque élément de la suite et son successeur forment une arête du graphe.


### Labyrinthes et graphes

On souhaite associer à un labyrinthe un graphe dont

  - les sommets sont les cellules vides du labyrinthe,

  - les arêtes représentent les déplacements admissibles d'une cellule à 
    une cellule voisine (les deux cellules sont vides et partagent un coté). 

  - le poids de chaque arête est 1 ; il représente le "coût" du déplacement
    d'une cellule à une cellule voisine.

Quelle structure de données Python utiliserait-t'on naturellement
pour représenter ces graphes ? 
⚠️ On ne cherche pas ici la structure la plus compacte ou performante 
mais à traduire aussi litéralement que possible la description mathématique 
du graphe.

Implémentez une fonction `maze_to_graph` qui construit le graphe associé 
à un labyrinthe.

<details>
<summary>
**Solution**
</summary>

Il semble naturel de représenter 
les sommets comme un ensemble de paires d'entiers, les arêtes comme un ensemble
de paires de sommets et les poids comme un dictionnaire ayant
comme clés les sommets et comme valeur unique l'entier 1.

```python
def maze_to_graph(maze):
    vertices = set(maze)
    edges = set()
    weights = {}
    for vertex in vertices:
        x, y = vertex
        for (dx, dy) in [(-1, 0), (0, -1), (1, 0), (0, 1)]:
            neighbor = (x + dx, y + dy)
            if neighbor in vertices:
                edge = (vertex, neighbor)
                edges.add(edge)
                weights[edge] = 1
    return (vertices, edges, weights)
  ```

</details>

### Cellules atteignables

Implémentez une fonction `reachable_cells` qui renvoie l'ensemble
des cellules d'un labyrinthe `maze` qui sont atteignables depuis 
la cellule `source`.

Etendez la fonction `display_maze` pour différencier graphiquement 
un ensemble de cellules. Puis utilisez-là pour représenter l'ensemble
des cellules atteignables depuis le coin en haut à gauche du labyrinthe
`"random"`.

![Cellules atteignables depuis le coin en haut à gauche (en vert). 
Un groupe de cellules vides sont inatteignables (en blanc) dans le coin en bas à gauche.](images/reachable-cells.jpg)


<details>
<summary>
**Solution**
</summary>

```python
def reachable_cells(maze, source):
    vertices, edges, _ = maze_to_graph(maze)
    todo = {source}
    done = set()
    while todo:
        current = todo.pop()
        neighbors = {
            v for v in vertices 
            if (current, v) in edges
        }
        for n in neighbors:
            if n not in done:
                todo.add(n)
        done.add(current)
    return done
```

```python
LIGHT_GREEN = (128, 255, 128)

def draw_cells(screen):
    h = CELL_SIZE
    for x, y in cells:
        pg.draw.rect(screen, LIGHT_GREEN, (x * h, y * h, h, h))

def display_maze(maze, cells=None):
    pg.init()
    pg.display.set_caption("Labyrinthes")
    width_height = (WIDTH * CELL_SIZE, HEIGHT * CELL_SIZE)
    screen = pg.display.set_mode(width_height)
    clock = pg.time.Clock()
    while True:
        events = pg.event.get()
        if any(event.type == pg.QUIT for event in events):
            break
        draw_background(screen)
        draw_walls(screen, maze)
        if cells is not None:
            draw_cells(screen)
        pg.display.update()
        clock.tick(FPS)
    pg.quit()
```

```python
cells = reachable_cells(random_maze, source=(0, 0))
display_maze(maze, cells=cells)
```

</details>


### Labyrinthes et chemins 

Implémentez une fonction `path_from` qui prend comme arguments :

  - `maze`: un labyrinthe 30x30,

  - `source`: une cellule source ,
  
et renvoie 

  - `path`: un dictionnaire ayant pour clés des cellules et 
    pour valeurs des chemins. Le chemin `path[target]` doit joindre 
    `source` et `target` si `target` est atteignable depuis `source` ; 
    dans le cas contraire, `target` ne doit pas être une clé du dictionnaire.

Utilisez cette fonction pour trouvez un chemin joignant les coins en haut à
gauche et en bas à droite du labyrinthe "random" et représenter graphiquement
le résulat.


![Un chemin joignant les coins en haut à gauche et en bas à droite.](images/path.jpg)

<details>
<summary>
**Solution**
</summary>

Une solution possible consiste à définir :

```python
def path_from(maze, source):
    vertices, edges, _ = maze_to_graph(maze)
    todo = set()
    done = set()
    path = {}
    if source in maze:
       todo.add(source)
       path[source] = [source]
    while todo:
        current = todo.pop()
        neighbors = {
            v for v in vertices 
            if (current, v) in edges
        }
        for n in neighbors:
            if n not in done and n not in todo:
                path[n] = path[current] + [n]
                todo.add(n)
        done.add(current)
    return path
```

puis à étendre notre fonction `display_maze` pour qu'elle prenne en charge
(optionnellement) l'affichage d'un chemin :

```python
PINK = (255, 128, 128)

def draw_path(screen, path):
    h = CELL_SIZE
    for x, y in path:
        pg.draw.rect(screen, PINK, (x * h, y * h, h, h))

def display_maze(maze, cells=None, path=None):
    pg.init()
    pg.display.set_caption("Labyrinthes")
    width_height = (WIDTH * CELL_SIZE, HEIGHT * CELL_SIZE)
    screen = pg.display.set_mode(width_height)
    clock = pg.time.Clock()
    while True:
        events = pg.event.get()
        if any(event.type == pg.QUIT for event in events):
            break
        draw_background(screen)
        draw_walls(screen, maze)
        if cells is not None:
            draw_cells(screen)
        if path is not None:
            draw_path(screen, path)
        pg.display.update()
        clock.tick(FPS)
    pg.quit()
```

On exploite ensuite ces fonctions de la façon suivante:

```python
TOP_LEFT = (0, 0)
BOTTOM_RIGHT = (WIDTH - 1, HEIGHT - 1)
target_to_path = path_from(maze, TOP_LEFT)
path = target_to_path[BOTTOM_RIGHT]
display_maze(maze, path=path)
```
</details>

Etendre à nouveau la fonction `display_maze` pour qu'elle accepte comme
argument supplémentaire le dictionnaire produit par `path_from`  et 
affiche chaque cellule atteignable dans une couleur dépendant 
de la longueur du chemin qui y mène.

Exploiter cette fonctionnalité avec le labyrinthe `"random"` avec comme
source le coin en haut à gauche.


![La carte des longueurs des chemins issus du coin en haut à gauche (violet
pour de petits nombres, jaune pour de grands nombres).](images/map.jpg)


🗝️ On pourra utiliser la fonction `colormap` suivante qui associe aux nombres
flottants entre `0.0` et `1.0` une couleur exploitable avec pygame.

```python
import matplotlib.cm  # matplotlib colormaps

COLORMAP = matplotlib.cm.viridis

def colormap(x):
    x = float(x)
    rgba = COLORMAP(x)
    rgb = rgba[0:3]
    RGB = [min(int(256 * c), 255) for c in rgb]
    return RGB
```

```python
assert colormap(0.0) == [68, 1, 84]     # purple
assert colormap(0.5) == [32, 145, 140]  # turquoise
assert colormap(1.0) == [254, 231, 36]  # yellow
```

<details>
<summary>**Solution**</summary>

```python
def draw_map(screen, map):
    h = CELL_SIZE
    v_max = max(v for v in map.values())
    for (x, y), v in map.items():
        pg.draw.rect(
            screen,
            colormap(float(v / v_max)),
            (x * h, y * h, h, h),
        )

def display_maze(maze, cells=None, path=None, map=None):
    pg.init()
    pg.display.set_caption("Labyrinthes")
    width_height = (WIDTH * CELL_SIZE, HEIGHT * CELL_SIZE)
    screen = pg.display.set_mode(width_height)
    clock = pg.time.Clock()
    while True:
        events = pg.event.get()
        if any(event.type == pg.QUIT for event in events):
            break
        draw_background(screen)
        draw_maze(screen, maze)
        if cells is not None:
            draw_cells(screen, cells)
        if map is not None:
            draw_map(screen, map)
        if path is not None:
            draw_path(screen, path)
        pg.display.update()
        clock.tick(FPS)
    pg.quit()
```
</details>

### Chemin optimal 

Implémentez une fonction `shortest_path_from(maze, origin)` qui renvoie un 
dictionnaire dont les clés sont les cellules atteignables depuis l'origine
et les valeurs un des chemins associés le plus courts (nécessitant le moins
de déplacements) qui joignent la source et la cible.

Vous pourrez tester votre résultat graphiquement en invoquant `display_maze`
comme à la question précédente.

![La carte des longueurs des plus courts chemins chemins issus du coin en haut à gauche (violet
pour de petits nombres, jaune pour de grands nombres).](images/optimal-map.jpg)

```python
def shortest_path_from(graph, source, target): 
    vertices, edges, weight = graph
    distance, paths = {}, {}
    todo = {source}
    distance[source] = 0
    paths[source] = [source]
    while todo:
        current = todo.pop()
        if current == target:
            return paths[current]
        neighbors = {
            v for v in vertices 
            if (current, v) in edges
        }
        for n in neighbors:
            d = distance[current] + weight[(current, n)]
            if d < distance.get(n, math.inf):
                distance[n] = d
                paths[n] = paths[current] + [n]
                todo.add(n)
    return None
```

<!--

Performance
--------------------------------------------------------------------------------

Plusieurs stratégies permettent d'améliorer les performances de la recherche
des plus courts chemins, un point qui devient critique quand la taille des
labyrinthes augmente ; notamment le choix de structures de données plus 
efficaces, et choix d'algorithmes plus efficaces.

### Mesure de la performance

Dans tous les cas, pour mesurer les (éventuels) progrès réalisés,
nous pourrons afficher le temps passé à déterminer les chemins optimaux ;
par exemple :

``` python
start = time.time()
paths = shortest_paths(maze, origin)
stop = time.time()
print(f"elapsed time (secs): {stop - start}")
```

Pour obtenir une image plus précise de ce qui se passe, et savoir dans quelle 
partie du code le temps est passé, on pourra utiliser le projet

  - 🐍 <https://github.com/pyutils/line_profiler>

### Structure de données

La structure de données choisie initialement pour représenter les graphes
n'est pas nécessairement la mieux choisie. Déterminez dans votre algorithme
quelles sont les opérations les plus fréquemment utilisées ; adaptez 
votre représentation des graphes en conséquence et mesure le résultat.

-->

## Pour aller plus loin

### Algorithmes

Améliorez ensuite l'algorithme lui-même. On pourra notamment étudier le
classique : 

  - 🎓 <https://fr.wikipedia.org/wiki/Algorithme_de_Dijkstra>

