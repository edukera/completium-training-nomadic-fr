# Exercices Archetype

Bonjour, bienvenue dans la session d'exercices du langage [Archetype](https://archetype-lang.org/).

## Bases de la syntaxe

### Exercice 1

Écrire, déployer et appeler un premier Smart Contract en Archetype qui permette d’enregistrer une chaîne de caractères et de la modifier avec un point d’entrée qui prend la nouvelle valeur en argument.

Un contrat Archetype commence par le mot clé archetype suivi du nom logique du contrat. Par exemple dans ce cas:

```
archetype hello

...
```

Créer un fichier hello.arl dans VS Code en cliquant droit dans le panneau latéral droit et en sélectionnant “Nouveau Fichier”.

Déployer le contrat avec votre compte de test. Pour cela, importer le compte Faucet dans la CLI de Completium:

```completium-cli import faucet <FAUCET.json> as <ACCOUNT_ALIAS>```

Pour créer le faucet dans l’environnement Gitpod, créer un fichier faucet.json et y copier/coller le contenu du fichier Faucet.

Déclarer ce compte comme le compte courant :

```completium-cli switch account```

Sélectionner le compte que vous venez d’importer.

Déployer le contrat :

```completium-cli deploy hello.arl```

L’URL du contrat apparaît dans la trace de la commande de déploiement. Cliquer dessus pour observer le contrat dans l’indexer Better-Call-Dev.

Appeler le contrat pour changer la chaîne de caractères du contrat :

```completium-cli call hello --with "Hello Archetype World"```

> le contrat n’ayant qu’un point d’entrée, il n’est pas nécessaire de spécifier le nom du point d’entrée. L’option --entry est utilisée pour spécifier le nom du point d’entrée.

### Exercice 2

Une obligation à coupon zéro (zero coupon bond) est la plus simple des obligations entre un émetteur (issuer) et un souscripteur (holder).

La différence entre le prix d'émission (original value) et le prix de rachat (face value) est régie par un coefficient (face rate) tel que:

```face value = face rate * original value```

Le contrat [zcb.arl](./contracts/zcb.arl) implante une obligation à zéro coupon:
le point d’entrée subscribe permet au souscripteur déclaré de transférer les fonds à l’émetteur
le point d’entrée redeem permet à l'émetteur de racheter l’obligation

Parmis les éléments suivants d’une obligation à coupon zéro, 2 ne sont pas implantés par zcb.arl; les trouver et les corriger :
* la date de maturité est calculée comme la date de souscription plus 356 jours
* le rachat de l’obligation ne peut se faire qu’après la date de maturité
* la valeur de rachat (face value) est le prix d’émission multiplié par le coefficient
* le solde du contrat est toujours 0 XTZ




