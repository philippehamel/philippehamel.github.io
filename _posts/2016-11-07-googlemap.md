---
layout: post
comments:  true
title:  "Localiser des points sur une carte avec R"
date: 2016-11-07
published: true
categories: ['Carte']
output:
  html_document:
    mathjax:  default
---

Afin de montrer comment utiliser **ggmap** pour localiser des points en utilisant R, je vais usiliser les données de localisation du Cicuit Électrique qui se trouve ici <https://lecircuitelectrique.com/trouver-une-borne>. J'ai inclus dans le fichier *data* un jeu de données préalablement nettoyé et dont les variable *string* ont été manipulées. Je ne montrerai pas comment manipuler des *string* aujourd'hui, ce sera le sujet d'un prochain tutoriel.

{% highlight r %}
library(RCurl)
{% endhighlight %}



{% highlight text %}
## Warning: package 'RCurl' was built under R version 3.2.4
{% endhighlight %}



{% highlight text %}
## Loading required package: methods
{% endhighlight %}



{% highlight text %}
## Loading required package: bitops
{% endhighlight %}



{% highlight r %}
x <- getURL("https://raw.githubusercontent.com/philippehamel/data-BlogueuR/master/Carte/BorneCE_16-09.csv")

Borne <- read.csv(text = x)

BorneQc <- Borne[Borne$Adresse.Province == "QC", ]
head(Borne)
{% endhighlight %}



{% highlight text %}
##          ID                                  ID.Parc
## 1   CEA-146           First Capital - Place Nelligan
## 2 CEA-10097                DGI - Complexe Desjardins
## 3   CEA-067 St-Laurent - Biblioth\x8fque du Bois\x8e
## 4   CEA-049         Hydro-Qu\x8ebec - CA Lebourgneuf
## 5   CEA-461    Shawinigan - Ar\x8ena Gilles Bourassa
## 6 CEA-10205                     RIO - Parc Olympique
##                                            Adresse.Complete   Type
## 1     1134 St-Ren\x8e Boulevard West, Gatineau, QC, J8T 6H1 Level2
## 2           1251 rue Jeanne-Mance, Montr\x8eal, QC, H2X 3Y2 Level2
## 3            2727 boul. Thimens, Saint-Laurent, QC, H4R 1T4 Level2
## 4            2625 boul. Lebourgneuf, Qu\x8ebec, QC, G2C 1P1 Level2
## 5                1705 117e rue, Shawinigan-Sud, QC, G9N 6V3 Level2
## 6 4141 Avenue Pierre-De Coubertin, Montr\x8eal, QC, H1V 3N7 Level2
##   Latitude Longitude                     Adresse.Rue  Adresse.Ville
## 1 45.48785 -75.70122  1134 St-Ren\x8e Boulevard West       Gatineau
## 2 45.50792 -73.56414           1251 rue Jeanne-Mance    Montr\x8eal
## 3 45.50482 -73.70434              2727 boul. Thimens  Saint-Laurent
## 4 46.82593 -71.31671          2625 boul. Lebourgneuf      Qu\x8ebec
## 5 46.51708 -72.75541                   1705 117e rue Shawinigan-Sud
## 6 45.55542 -73.55138 4141 Avenue Pierre-De Coubertin    Montr\x8eal
##   Adresse.Province Adresse.Postal
## 1               QC        J8T 6H1
## 2               QC        H2X 3Y2
## 3               QC        H4R 1T4
## 4               QC        G2C 1P1
## 5               QC        G9N 6V3
## 6               QC        H1V 3N7
{% endhighlight %}

Le *package* utilisé dans ce tutoriel est:


{% highlight r %}
require(ggmap)
{% endhighlight %}
Avant de commencer, il est important de comprendre le concept de **ggmap**. Lorsque **ggplot** construit des graphiques, il superpose des couches, ou *layers*. La fonction *get_map* va nous permettre d'aller chercher une carte dont l'axe *x* est la longitude et l'axe *y* est la latitude, puis de l'utiliser comme base pour superposer des couches contenant des points.

La première étape de la construction de la carte est le choix de la localisation. Il existe trois manière de sélectionner celle-ci.

Avec une adresse :

{% highlight r %}
Location <- "Université Laval"
{% endhighlight %}
Les coordonnées d'un lieu :

{% highlight r %}
Location <- c(lat = 47.558820, lon = -71.232386)
{% endhighlight %}
Où en donnant une série de coordoné qui serviront à faire un cadre :

{% highlight r %}
Location <- c(-80, 40, -62, 52)
{% endhighlight %}

Puis il faut créer un objet qui servira de base à la carte :


{% highlight r %}
map <- get_map(location = Location, source = "google", maptype = "roadmap", zoom = 11)
{% endhighlight %}
La fonction **get_map**, qui va chercher la carte dans google map, prend au minimum 4 arguments. L'argument **location** est le lieu que l'on veut avoir sur la carte. L'argument **zoom** est l'équivalent des + et - sur google map, il permet de s'éloigner ou se rapprocher du lieu qui à été sélectionné. L'arguement **source** est la source de provenance de la carte. Il il y a au moins 4 sources disponibles, et chacune a ses propres **maptype**.
La *cheat sheet* suivante les explique mieux que j'en serais capable dans cet article : <https://www.nceas.ucsb.edu/~frazier/RSpatialGuides/ggmap/ggmapCheatsheet.pdf>.

La dernière étape est d'y superposer les points à l'aide de **ggplot** :

{% highlight r %}
Borne_point <-   ggmap(map) +
    geom_point(data = BorneQc, 
               aes(x = Longitude, y = Latitude), 
               size = 1) +
    ggtitle("Répartition des bornes du Circuit Électrique au Québec") +
  theme_nothing(legend = T)
{% endhighlight %}
![plot of chunk unnamed-chunk-10](/figure/source/2016-11-07-googlemap/unnamed-chunk-10-1.png)

Les fonctionnalités disponibles avec **ggplot** sont utilisable ici. Par exemple, on peut changer la couleur des points selon des variables catégoriques. Le fichier de données donne une valeur différente pour chaque type de borne, recharge rapide ou 200 volt. On peut donc leur donner chacun une couleur différente avec l'arguement **color**


{% highlight r %}
Borne_point <-   ggmap(map) +
    geom_point(data = BorneQc, 
               aes(x = Longitude, y = Latitude, color = Type), 
               size = 1) +
    ggtitle("Répartition des bornes du Circuit Électrique au Québec") +
    scale_colour_manual(name = "", 
                        values = c("Level2" = "blue", "FastDC" = "red")) +
    theme_nothing(legend = T)
{% endhighlight %}
![plot of chunk unnamed-chunk-12](/figure/source/2016-11-07-googlemap/unnamed-chunk-12-1.png)

**Note importante** : pour des utilisations académiques ou qui nécessitent une diffusion publique, ggmap demande de se faire citer de la manière suivante :

{% highlight r %}
citation('ggmap')
{% endhighlight %}



{% highlight text %}
## 
## To cite ggmap in publications, please use:
## 
##   D. Kahle and H. Wickham. ggmap: Spatial Visualization with
##   ggplot2. The R Journal, 5(1), 144-161. URL
##   http://journal.r-project.org/archive/2013-1/kahle-wickham.pdf
## 
## A BibTeX entry for LaTeX users is
## 
##   @Article{,
##     author = {David Kahle and Hadley Wickham},
##     title = {ggmap: Spatial Visualization with ggplot2},
##     journal = {The R Journal},
##     year = {2013},
##     volume = {5},
##     number = {1},
##     pages = {144--161},
##     url = {http://journal.r-project.org/archive/2013-1/kahle-wickham.pdf},
##   }
{% endhighlight %}
