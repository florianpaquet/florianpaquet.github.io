---
layout: post
title: "Django REST framework, JWT et FranceConnect"
date: 2017-10-27 23:00:00 +0200
categories: django
image: assets/images/france-connect.jpg
---
Je me suis récemment penché sur l'interfaçage de Django, Django REST framework, des JSON Web Tokens et de FranceConnect. Découvrez ici comment faire !
<!--more-->

# Python Social Auth

Python Social Auth permet de mettre en place des mécanismes d'authentification à l'aide de services tiers (Google, Facebook, GitHub, etc.).

Sa force réside dans sa notion de `pipeline`, permettant de définir un ensemble d'actions à effectuer à l'authentification d'un utilisateur, par exemple (liste non exhaustive) :

- Stocker l'avatar de l'utilisateur
- Générer un nom d'utilisateur unique
- Associer un utilisateur existant

[La liste des backends supportés](http://python-social-auth.readthedocs.io/en/latest/backends/index.html) est déjà bien fournie et permet de mettre en place les services les plus courants.

[Voir ici pour installer et configurer Python Social Auth avec Django](http://python-social-auth.readthedocs.io/en/latest/configuration/django.html).

# djangorestframework-jwt

Les JSON Web Tokens (JWT) pour Django REST framework. Je passe rapidement sur ce package car si vous êtes ici, c'est que vous connaissez déjà le principe !

[Voir ici pour installer et configurer djangorestframework-jwt](http://getblimp.github.io/django-rest-framework-jwt/).

# rest-social-auth

C'est grâce à ce package que Django REST framework, Python Social Auth et JWT communiquent ensemble.

Il propose un point d'API prenant en entrée le code d'autorisation fourni par le service tiers et une URL de callback optionnelle.

C'est donc au client (front, mobile, etc.) de se charger d'initialiser l'autorisation avec le service tiers. Une fois le code en main, il effectue une requête sur le point d'API dédié et Python Social Auth prend le relai à partir de l'obtention du token d'accès.

![](https://assets.digitalocean.com/articles/oauth/abstract_flow.png){: .centered }

[Voir ici pour installer et configurer rest-social-auth](https://github.com/st4lk/django-rest-social-auth). Activez les routes JWT pour notre cas.

# FranceConnect

Il faut maintenant créer le backend de Python Social Auth pour s'authentifier à l'aide de FranceConnect. Sans plus attendre, voici le code fonctionnel :

```python
from urllib.parse import urlencode
from social_core.backends.base import BaseAuth
from social_core.exceptions import AuthFailed


class FranceConnectAuth(BaseAuth):
    """
    France Connect OpenID authentication backend
    """
    name = 'france-connect'
    BASE_URL = 'https://fcp.integ01.dev-franceconnect.fr'
    AUTHORIZATION_URL = '/api/v1/authorize'
    ACCESS_TOKEN_URL = '/api/v1/token'
    USER_INFO_URL = '/api/v1/userinfo'

    def get_url(self, url=""):
        return self.setting('BASE_URL', self.BASE_URL) + url

    def auth_url(self):
        args = {
            'response_type': 'code',
            'client_id': self.setting('CLIENT_ID'),
            'state': 'test',
            'nonce': 'test',
            'scope': self.setting('SCOPE', 'openid identite_pivot'),
            'redirect_uri': self.redirect_uri
        }
        args.update(self.auth_extra_arguments())

        return '{url}?{params}'.format(
            url=get_url(self.AUTHORIZATION_URL),
            params=urlencode(args))

    def auth_complete(self, *args, **kwargs):
        # Retrieve access code
        code = self.data.get('code')

        if not code:
            raise ValueError("No code returned")

        # Query for access token
        token_response = self.get_json(
            get_url(self.ACCESS_TOKEN_URL),
            data={
                'grant_type': 'authorization_code',
                'redirect_uri': self.redirect_uri,
                'client_id': self.setting('CLIENT_ID'),
                'client_secret': self.setting('SECRET_KEY'),
                'code': code
            },
            method='POST')

        if token_response.get('status') == 'failure':
            raise AuthFailed(self)

        # Query for user details
        response = self.get_json(
            get_url(self.USER_INFO_URL),
            params={
                'schema': 'email'
            },
            headers={
                'Authorization': 'Bearer {}'.format(token_response['access_token'])
            })

        kwargs.update({'response': response, 'backend': self})
        return self.strategy.authenticate(*args, **kwargs)

    def get_user_details(self, response):
        return {
            'username': response['sub'],
            'first_name': response['given_name'],
            'last_name': response['family_name'],
            'email': response['email']
        }
```

Ce backend reprend la logique du workflow présenté avant, en intégrant les variables nécessaires à FranceConnect.

La méthode `auth_url` ne sert pas vraiment dans notre cas car cette étape est gérée par le client (front, mobile, etc.).

Voici la liste des paramètres configurables dans le `settings.py` :

- `SOCIAL_AUTH_FRANCE_CONNECT_BASE_URL` : l'URL de base du service (https://fcp.integ01.dev-franceconnect.fr par défaut)
- `SOCIAL_AUTH_FRANCE_CONNECT_SCOPE` : le scope à utiliser pour la récupération des informations de l'utilisateur. [Voir la documentation officielle][fc-doc] pour plus d'informations.
- `SOCIAL_AUTH_FRANCE_CONNECT_CLIENT_ID` : le client ID fourni par FranceConnect
- `SOCIAL_AUTH_FRANCE_CONNECT_SECRET_KEY` : la clé secrète fournie par FranceConnect

Ajoutez ce backend dans la liste des `AUTHENTICATION_BACKENDS` de Django.

# It's a kind of magic

![](http://www.magicienh.fr/wp-content/uploads/2017/02/Garcimore-prestidigitateur.jpg){: .centered}

Toujours en utilisant la [documentation de FranceConnect][fc-doc], utilisez votre client pour émettre la première requête d'autorisation vers FranceConnect. Il s'agit de l'URL construite avec la méthode `auth_url` du backend précedemment créé.

**C'est ici qu'il faut être rigoureux !**

Le `redirect_uri` que vous allez transmettre à FranceConnect doit rediriger vers votre client qui se chargera de récupérer le `code` d'autorisation.

Lorsque vous allez émettre la requête à l'API pour authentifier l'utilisateur, **vous devez impérativement renseigner la même `redirect_uri`**, sinon l'authentification échoue. Pour ce faire, construisez le corps de la requête comme suit :

```json
POST /api/social/jwt/france-connect
{
    "code": "secret-c0de",
    "redirect_uri": "Même `redirect_uri` que pour l'autorisation"
}
```

Et, magie : si le workflow se déroule sans accroc, vous obtenez un JSON Web Token utilisable dans votre application pour vous authentifier !

[fc-doc]: https://partenaires.franceconnect.gouv.fr/fournisseur-service
