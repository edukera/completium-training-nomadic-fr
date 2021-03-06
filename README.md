# Formation Archetype

Bonjour, bienvenue dans la session de formation au langage [Archetype](https://archetype-lang.org/) :

* [Introduction](#introduction)
* [Base de la syntaxe](#bases-de-la-syntaxe)
* [Collection d'assets](#collection-dassets)
* [Machine à état](#machine-à-état)
* [Evolution de contrat](#evolution-de-contrat)
* [Vérification formelle](#vérification-formelle)
* [Application décentralisée](#application-décentralisée)

# Introduction

> Présentation : [vidéo](https://www.youtube.com/watch?v=Zj1m-BouSkc)

# Bases de la syntaxe


> Présentation : [vidéo](https://www.youtube.com/watch?v=vcsMwDImTH8)

> Exercices : [vidéo](https://www.youtube.com/watch?v=OAQz2xMTq28)

## Exercice 1

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

```completium-cli call hello --with '"Hello Archetype World"'```

> le contrat n’ayant qu’un point d’entrée, il n’est pas nécessaire de spécifier le nom du point d’entrée. L’option --entry est utilisée pour spécifier le nom du point d’entrée.

## Exercice 2

Une obligation zéro-coupon (zero coupon bond) est la plus simple des obligations entre un émetteur (issuer) et un souscripteur (holder).

La différence entre le prix d'émission (*original value*) et le prix de rachat (*face value*) est régie par un coefficient (*face rate*) tel que:

```face value = face rate * original value```

Le contrat [zcb.arl](./contracts/zcb.arl) implante une obligation à zéro-coupon :
le point d’entrée `subscribe` permet au souscripteur déclaré de transférer les fonds à l’émetteur
le point d’entrée `redeem` permet à l'émetteur de racheter l’obligation

Parmis les 4 propriétés suivantes de l'obligation zéro-coupon, 2 ne sont pas vérifiées par zcb.arl; les trouver et les corriger :
1. la date de maturité (`redemption`) est calculée comme la date de souscription plus 365 jours
2. le rachat de l’obligation ne peut se faire qu’après la date de maturité
3. la valeur de rachat (`facevalue`) est le prix d’émission (`originalvalue`) multiplié par le coefficient (`facerate`)
4. le solde du contrat est toujours 0 XTZ

# Collection d'assets

> Présentation : [vidéo](https://www.youtube.com/watch?v=tI2b55fFajg)

> Exercices : [vidéo](https://www.youtube.com/watch?v=zOVulHJzysA)

> Documentation sur les [assets](https://docs.archetype-lang.org/archetype-language/data-model)

Un token non fongible (non fungible token, NFT) est un token unique que l’on peut transférer d’un propriétaire à un autre.

Le contrat [nft.arl](./contracts/nft.arl) implante un NFT en s’inspirant de la norme du FA 2 de Tezos pour les tokens non fongibles.

Cette norme prévoit que l’on puisse déléguer la capacité de transfert à un tiers. Les informations de délégation (allowance) sont stockées dans une collection d’asset. Le point d’entrée permettant d’autoriser un tiers à transférer son token est allow.

Le tiers délégataire est typiquement un autre contrat qui met en œuvre un processus de vente, comme des enchères par exemple.

Le point d’entrée transfer effectue le transfert de propriété du token en lisant la table des autorisations dans le cas où l'appelant n’est pas le propriétaire du token.

## Exercice 1

Écrire le point d’entrée `mint` du contrat de token non fongible nft.arl pour ajouter un asset de token au ledger.
Ce point d'entrée prend en argument le nécessaire pour créer le token et ne peut être appelé que par l'adresse `admin`.

## Exercice 2

Ajouter des données au token :
* `uri` (adresse IPFS de l’asset digital)
* `creator` (adresse du compte tezos du créateur du NFT)
* `royalties` (pourcentage reversé au créateur à chaque vente)
* `nbtransfers` (nombre de transferts du NFT)

Écrire les points d’entrée :
* `increase` qui augmente de 10% le pourcentage reversé au créateur des tokens ayant été échangés plus de 10 fois
* `rm` qui supprime du ledger les tokens ayant été échangés plus de 20 fois

# Machine à état

> Présentation : [vidéo](https://www.youtube.com/watch?v=Rp67NCopj0s)

> Exercices : [vidéo](https://www.youtube.com/watch?v=aTKxdkTUVU0)

> Documentation sur les [machines à état](https://docs.archetype-lang.org/archetype-language/state-machine)

## Exercice 1

Écrire une version du contrat zcb.arl avec un état à 4 valeurs:
* `Created`
* `Subscribed`
* `Redeemed`
* `Defaulted`

Transformer le point d’entrée `subscribe` en transition de `Created` vers `Subscribed`.

Transformer le point d’entrée `redeem` en transition de `Subscribed` vers `Redeemed`.

Ajouter une transition `default` appelée par `holder`, qui passe de `Subscribed` vers `Defaulted` si le rachat de l'obligation ne s'est pas passé dans les 5 jours suivant la date de maturité :

 * Définir une variable du Storage nommé `payback` de type `duration` initialisée à 5 jours.
 * Ajouter à la transition `default` la condition (d'exécution) que la date d'appel est au délà de 5 jours après la date de maturité.

## Exercice 2

Etablir le point d’entrée `sign` qui doit être appelé par le souscripteur ET l'émetteur pour que le contrat passe de l’état `Created` à `Subscribed`.

Ce point d’entrée mémorise dans deux variables booléennes du Storage si le souscripteur et l’émetteur l’ont appelé de sorte que, lorsque les deux adresses l'ont appelé, `sign` appelle alors la transition `subscribe`.

La transition `subscribe` ne peut être appelée que par le contrat lui-même.

> Un contrat peut s’appeler lui-même avec l’instruction transfer:
> `transfer 0tz to entry self.subscribe();`

> L’adresse du contrat est `selfaddress`.

# Evolution de contrat

> Présentation : [vidéo](https://www.youtube.com/watch?v=qhH-iFRNs_A)

> Exercices : [vidéo](https://www.youtube.com/watch?v=lvDZh9vbUhA)

> Documentation sur les [appels inter-contrats](https://docs.archetype-lang.org/archetype-language/transfers)

Mettre en place le principe de modularité d’un processus de vente aux enchères d’un token NFT avec les contrats `nft.arl` et [auction.arl](./contracts/auction.arl)

Le contrat `auction.arl` fournit le mécanisme de vente aux enchères d'un token:
* `upforsale` est appelé par le propriétaire du token pour le mettre aux enchères
* `bid` est appelé pour faire une offre
* `claim` est appelé pour transférer la propriété du token au gagnant de l’enchère

## Exercice 1

Ajouter un getter nommé `getOwner` au contrat NFT qui renvoie le propriétaire du token dont l'identifiant est passé en argument.

Modifier le contrat d’enchère `auction.arl` de façon à ce qu’il interagisse avec le contrat NFT pour :
* Vérifier que le token échangé appartienne à celui qui démarre les enchères :
  * Créer un point d'entré nommé `checkowner` qui prend en argument une adresse.
  * Ce point d'entré doit échouer si l'appelant n'est pas l'addresse du contrat `nft` ET si l'adresse en argument n'est pas l'adresse du déclencheur initiale de l'operation (constante `source` en archetype).
  * Dans le point d'entrée `upforsale`, ajouter un appel au getter `getOwner` du contrat `nft` avec comme callback le point d'entrée `checkowner`.
* Autoriser le contrat d’enchère à effectuer le transfert :
  * Dans le point d'entrée `upforsale`, ajouter un appel au point d'entré `approve` du contrat `nft`.
  * Les arguments de `approve` sont l'adresse de ce contrat (`selfaddress`) et l'identifiant du token (`tokenid`) pour approver ce contrat à transférer ce token.
* Transférer le token au gagnant de l’enchère :
  * Dans le point d'entrée `claim`, ajouter un appel au point d'entré `%transfer` au contrat `nft` dans l'entrée `claim`, après le transfert de fonds à l'ex-propriétaire.
  * Les arguments de `%transfer` sont l'adresse de l'ex-propriétaire,l'adresse du gagnant de l'enchère et l'identifiant du token (`tokenid`).

## Exercice 2

L'objectif est d'observer dans l'indexer [Better Call Dev](https://better-call.dev/) les sous-transactions générées par les appels à `upforsale` et `claim`.

Dans ce qui suit, `admin` désigne le compte courant.

Prérequis:
* Créer un nouvel utilisateur nommé `buyer` avec un nouveau faucet.json

Editer le fichier [exec_auction.sh](./contracts/exec_auction.sh) et remplacer `<YOUR_ALIAS>` avec l'alias de votre compte établi dans l'[exercice 1](#exercice-1) de la partie *Bases de la syntaxe* :

```sh
#completium-cli import faucet faucet_buyer.json as buyer

admin_alias=<YOUR_ALIAS>
buyer_alias=buyer

completium-cli set account $admin_alias
admin=`completium-cli show address $admin_alias`
completium-cli deploy nft.arl --init $admin --force
completium-cli call nft --entry mint --with "(24, $admin, \"ipfs://...\", $admin, 15, 100)" --force
nft=`completium-cli show address nft`
completium-cli deploy auction.arl --init "($admin, 24, $nft)" --force
completium-cli call auction --entry upforsale --with 10tz --force
completium-cli set account $buyer_alias
completium-cli call auction --entry bid --amount 12tz --force
# wait 3 minutes
sleep 180
completium-cli call auction --entry claim --force
```

Ce script consiste à :

* déployer le contrat `nft` avec le getter en l'initialisant avec l'adresse courante `admin` (`completium-cli show account`).
* créer le token ayant pour identifiant `24` et appartenant à l'adresse `admin` dans le contrat `nft` en appelant l'entrée `mint`.
* déployer le contract `auction` mettant aux enchères le token `24`.
* appeler l'entrée `upforsale` avec un prix de vente de `10tz`.
* changer de compte courant et selectionner `buyer`.
* enchérir à `12tz` en appelant l'entrée `bid` (constater qu'une valeur strictement inferieur à 10tz fait échouer l'appel)
* appeler `claim` une fois l'enchère finie et constater le changement effectif de propriétaire du token.

Exécuter le script avec la commande :

`sh ./contracts/exec_auction.sh`


# Vérification formelle

> Présentation : [vidéo](https://www.youtube.com/watch?v=59JJk-7gJxM)

## Exercice 1

Lister et écrire en **langage naturel** les postconditions du point d’entrée `%transfer` de nft.arl.

```
entry %transfer (%from : address, %to : address, tid : nat) {
  require {
    r0: ledger[tid].owner = %from
  }
  effect {
    if caller <> %from then begin
      dorequire (allowance.contains((%from, caller)), "CALLER NOT ALLOWED");
      allowance.remove(((%from, caller)))
    end;
    ledger[tid].owner := %to;
  }
}
```

## Exercice 2 (optionnel)

Transcrire les postconditions en langage de spécification formelle Archetype.

# Application décentralisée

> Présentation : [vidéo](https://www.youtube.com/watch?v=fyuB6CKmg4w)

Effectuer le didacticiel [First Dapp](https://completium.com/docs/dapp-first) du site Completium.
