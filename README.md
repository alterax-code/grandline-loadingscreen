# Écran de chargement

La page que voient les joueurs pendant la connexion : visuels, musique, conseils,
et une barre qui suit **le vrai téléchargement du contenu serveur**.

Source : la page de Lucas (`Grandline RP - Page de chargement.html`), déballée en
fichiers séparés et branchée sur les rappels du moteur.

---

## ⚠️ Elle doit être HÉBERGÉE — un fichier local ne marchera pas

Garry's Mod ne charge l'écran que depuis une **URL http(s)**. Le dossier
`loadingscreen/` est la *source* ; il faut en publier le contenu quelque part,
puis pointer le serveur dessus :

```
sv_loadingurl "https://ton-domaine/loading/"
```

Cette ligne va dans `server.cfg` (non versionné : il contient le mot de passe
rcon). N'importe quel hébergement statique convient — un sous-domaine à toi,
GitHub Pages, un bucket. L'`index.html` doit être à la racine de l'URL, avec le
dossier `assets/` à côté.

Le client ne l'affiche que si `cl_enable_loadingurl` est à 1 chez lui (défaut) et
que le serveur a plus d'un slot.

---

## Ce que la barre montre vraiment

Le moteur appelle cinq fonctions JS sur la page pendant la connexion. Elles sont
implémentées en fin d'`index.html`, dans un bloc commenté :

| Fonction | Ce qu'elle apporte |
|---|---|
| `GameDetails(...)` | nom du serveur, carte, nombre de places |
| `SetFilesTotal(n)` | nombre total de fichiers à télécharger |
| `SetFilesNeeded(n)` | combien il en reste |
| `DownloadingFile(nom)` | le fichier en cours |
| `SetStatusChanged(s)` | message d'état libre du moteur |
| `AllDownloadsComplete()` | tout est là, la carte va charger |

**Leurs noms sont imposés par le moteur.** Les renommer les rend muettes, sans
aucune erreur visible — la barre retomberait simplement sur son animation de
démo, et on croirait qu'elle marche.

### Pourquoi elle plafonne à 92 %

Ces rappels ne couvrent que le **téléchargement**. Ensuite la carte se charge, et
le moteur ne dit plus rien : il n'existe aucun moyen de suivre cette phase.

Une barre qui atteint 100 % puis reste plantée vingt secondes se lit comme un
blocage. Elle monte donc jusqu'à 92 % pendant le téléchargement, et ne va au bout
qu'une fois `AllDownloadsComplete` reçu — le temps restant se lit alors comme un
travail en cours, ce qu'il est.

### Elle converge, elle ne saute pas

Les paliers de téléchargement arrivent par à-coups. La barre **tend** vers la
valeur réelle au lieu de s'y téléporter : un bond de 12 % à 60 % en une image se
lit comme un bug. Et elle ne recule jamais.

### Hors GMod, l'animation de démo reprend

Ouverte dans un navigateur, la page anime la barre au hasard, comme la version
d'origine — sinon elle paraîtrait figée. Dès que le moteur parle, le vrai relais
prend la main.

---

## ⚠️ Le poids : 12,7 Mo, à réduire

C'est ce que **chaque joueur télécharge avant de voir l'écran**. Deux fichiers
font 88 % du total :

| Fichier | Poids | Remarque |
|---|---|---|
| `assets/ost01.mp3` | 6,02 Mo | la musique |
| `assets/img01.png` | 5,13 Mo | 2752×1536, le fond |
| les 28 autres | 1,6 Mo | rien à y faire |

Deux gestes suffiraient à passer sous les 3 Mo :

- **Le fond en JPEG qualité 85** au lieu du PNG : un visuel photographique de
  cette taille n'a aucun besoin du sans-perte. Compter ~400 Ko, soit **−4,7 Mo**.
- **La musique en 96 kbps mono** (ou une boucle plus courte) : c'est une ambiance
  de fond, pas une écoute. Compter ~1,5 Mo, soit **−4,5 Mo**.

Je ne l'ai pas fait moi-même : ré-encoder tes visuels et ta musique est un choix
qui t'appartient, et la différence de rendu se juge à l'œil et à l'oreille.

---

## ⚠️ La musique : à tester en jeu

La page lance la musique dès l'ouverture, et réessaie au premier clic. Les
navigateurs modernes bloquent souvent la lecture automatique tant qu'il n'y a pas
eu d'interaction — **le moteur de rendu de GMod peut faire pareil**.

Si le son ne part pas, ça se verra tout de suite au premier essai. C'est le seul
point de cette intégration que je n'ai pas pu vérifier sans lancer le jeu.

---

## Ce qui a été retiré

L'ancien `index.html` (écran sombre avec une barre dorée) est remplacé. Ses
quatre fichiers — `bebasneue.ttf`, `onepiece.ttf`, `grandline_bg.jpg`,
`grandline_bg_alt.jpg` — ne sont plus référencés par personne et ont été
supprimés. Tout reste dans l'historique git si tu veux y revenir.
