# Générateur de template pour les épisodes de podcastscience

# Description

`ps-gen` est un générateur de template pour les épisodes de podcastscience.

## Dépendances

Ce générateur dépend des outils suivant :

 * pdftk : pour la génération de la couverture
 * ImageMagick (convert) : pour la conversion d'images
 * pandoc : pour la génération des différents formats de fichiers de sortie


## Fonctionnement

Afin de générer un ensemble complet, les éléments suivants sont nécessaires :

 * document.md : fichier contenant le dossier
 * image_couverture.png : image qui sera affichée sur la couverture
 * image_couverture.md : éléments d'information sur l'image de couverture (crédit, auteur, etc)
 * teaser.md : teaser de l'épisode
 * quote.md : quote de l'épisode
 * README.md : fichier de readme expliquant quel est l'épisode
 * titre.md : fichier décrivant les méta-informations associées au dossier à créer

### Vérification des dépendances

Avant toute génération de documents pour un épisode, il est nécessaire de s'assurer que l'ensemble des dépendances nécessaires seront satisfaites.

Vous pouvez effectuer cette opération en utilisant la commande suivante : 

    make verify

### Génération des éléments d'un épisode

Pour générer les fichiers de base, il est nécessaire d'exécuter la commande suivante : 
    
    make generate

Cette commande génèrera pour vous les éléments nécessaires à la bonne génération des fichiers ci-dessus.

Cette commande vérifiera aussi que l'ensemble des dépendances pour la génération des documents est bien présent.

### Génération des fichiers de sortie

Trois formats sont actuellement disponibles comme sortie :

 * pdf
 * html
 * epub

Ces trois cibles sont donc disponibles pour la compilation :

    make pdf # pour la génération de pdf
    make html # pour la génération d'html
    make epub # pour la génération d'epub
    make all # pour tout générer

### Nettoyage des fichiers d'output

Pour nettoyer les fichiers d'output, des cibles correspondantes existent :

    make clean_pdf # pour supprimer les fichiers correspondant à la sortie pdf
    make clean_html # même chose pour la sortie html
    make clean_epub # même chose pour la sortie epub
    make clean # pour l'ensemble des sorties
