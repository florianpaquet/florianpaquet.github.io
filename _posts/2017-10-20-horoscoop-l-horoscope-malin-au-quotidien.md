---
layout: post
title:  "Horoscoop, l'horoscope malin au quotidien"
date:   2017-10-20 21:36:00 +0200
categories: projets
image: assets/images/elektronik-supersonik.jpg
image_description: Zlad!, inventeur du futur
---
Vous en avez assez de vivre dans l'expectative au quotidien, de voir des opportunités passer sous votre nez et d'aller au gré des rencontres plus ou moins bénéfiques pour vous ? [Horoscoop][horoscoop] est fait pour vous !
<!--more-->

# Partez désormais toujours du bon pied grâce à [Horoscoop][horoscoop].

Horoscoop est votre nouvel assistant quotidien et vous dicte votre horoscope. Oui, vous avez bien compris : Horoscoop lit votre horoscope pour vous.

Fini la corvée matinale de récolte de votre quotidien gratuit aux arrêts de tram bondés et le sourire forcé à ce jeune qui vous le tend. Car avouons-le, le seul intérêt de cette presse, c'est son avant-dernière page et son horoscope...

Quoi de mieux que de se faire annoncer une journée exécrable par une voix féminine neutre, limite robotique, avec un tact à toute épreuve :

> « Ne faites pas de diagnostic par le biais d'internet. Si vous n'avez pas le temps de voir un médecin, parlez-en au pharmacien. »

Prends ça dans tes dents Doctissimo.

# Les bienfaits d'un horoscope dès le réveil

Vous l'avez probablement expérimenté au moins une fois dans votre vie : le réveil radio FM à l'heure de l'horoscope. Souvenez-vous des jours annoncés comme compliqués et mettez-y en relation votre motivation à aller au travail...

À l'ère des assistants personnels (l'horoscope à la radio c'est désormais réservé aux _anciens_), Horoscoop s'intègre __parfaitement__ dans votre rythme quotidien et peut même vous conseiller de rester chez vous en cas de journée catastrophique.

Il ne vous restera alors plus qu'à en informer votre employeur en arguant que votre état de motivation pourrait être contagieux.

# Installation

```
sudo apt-get install -y libxml2-dev libxslt1-dev libttspico-utils
pip install horoscoop
```

# Utilisation

```
horoscoop [signe]
```

# Utilisation avancée

Ajoutez la commande précédente à votre `crontab` à l'horaire souhaité.

# Utilisation très avancée

Créer un script qui boucle sur l'ensemble des signes du Zodiaque afin de devenir __la__ référence horoscope de la machine à café.

# Références

[Horoscope.fr, horoscope de référence, fiable et reconnu](https://www.horoscope.fr/)

[horoscoop]: https://github.com/florianpaquet/horoscoop
