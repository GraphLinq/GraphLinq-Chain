# Commande Rollback pour Geth

## Description

La commande `rollback` permet de revenir en arrière dans la blockchain en supprimant un ou plusieurs blocs récents. Cela repositionne la tête de la chaîne à un bloc antérieur.

## Utilisation

```bash
geth rollback [nombre_de_blocs]
```

### Paramètres

- `nombre_de_blocs` (optionnel) : Le nombre de blocs à revenir en arrière
  - Par défaut : 1 (un seul bloc)
  - Doit être un nombre entier positif
  - Ne peut pas dépasser le numéro du bloc actuel

## Exemples

### Revenir d'un bloc en arrière (par défaut)

```bash
geth rollback
```

Cette commande supprime le dernier bloc et positionne la tête de la chaîne sur le bloc précédent.

### Revenir de plusieurs blocs en arrière

```bash
geth rollback 5
```

Cette commande supprime les 5 derniers blocs et positionne la tête de la chaîne 5 blocs en arrière.

### Avec un répertoire de données personnalisé

```bash
geth rollback --datadir /path/to/data 3
```

Cette commande effectue un rollback de 3 blocs sur une blockchain située dans un répertoire personnalisé.

## Options disponibles

La commande `rollback` supporte les options suivantes :

- `--datadir` : Spécifie le répertoire de données de la blockchain
- `--datadir.ancient` : Spécifie le répertoire pour les données anciennes
- `--cache` : Mémoire allouée au cache interne (en Mo)
- `--syncmode` : Mode de synchronisation de la blockchain

## Cas d'utilisation

### 1. Correction d'un bloc corrompu

Si le dernier bloc de votre chaîne est corrompu, vous pouvez le supprimer :

```bash
geth rollback
```

### 2. Retour à un état antérieur

Pour revenir à un état spécifique de la blockchain :

```bash
# Si vous savez que le problème a commencé il y a 10 blocs
geth rollback 10
```

### 3. Resynchronisation

Après un rollback, si vous redémarrez geth normalement, il resynchronisera automatiquement avec le réseau pour obtenir les nouveaux blocs.

## Avertissements et Précautions

⚠️ **ATTENTION** : Cette opération est **destructive** et **irréversible**.

### Avant d'utiliser la commande :

1. **Sauvegardez vos données** : Faites une copie complète de votre répertoire de données
   ```bash
   cp -r ~/.ethereum ~/.ethereum.backup
   ```

2. **Arrêtez geth** : Assurez-vous que geth n'est pas en cours d'exécution

3. **Vérifiez le numéro du bloc actuel** :
   ```bash
   geth attach
   > eth.blockNumber
   ```

### Limitations :

- Vous ne pouvez pas revenir en arrière au-delà du bloc genesis (bloc 0)
- Les transactions et l'état associés aux blocs supprimés seront perdus
- Si d'autres nœuds ont ces blocs, ils seront retéléchargés lors de la resynchronisation

## Erreurs courantes

### "Cannot rollback genesis block"

Vous essayez de revenir en arrière alors que vous êtes déjà au bloc genesis.

### "Cannot rollback X blocks: current block is #Y"

Le nombre de blocs spécifié est supérieur au nombre de blocs dans votre chaîne.

### "No current block found"

La base de données de la blockchain est vide ou corrompue.

## Fonctionnement technique

La commande `rollback` utilise la méthode `SetHead` de la blockchain Ethereum, qui :

1. Lit le bloc actuel
2. Calcule le numéro du bloc cible (actuel - nombre_de_blocs)
3. Repositionne la tête de la chaîne à ce bloc
4. Supprime les données associées aux blocs ultérieurs (headers, corps, receipts, etc.)
5. Met à jour les caches et les métadonnées
6. **Met à jour le fichier `metadata.json`** avec les informations du nouveau bloc courant

### Mise à jour du metadata.json

Après un rollback, le fichier `metadata.json` est automatiquement mis à jour pour refléter le nouveau bloc courant. Cela garantit que :
- Les scripts de monitoring ont toujours des informations à jour
- Le numéro de bloc dans `metadata.json` correspond à l'état réel de la blockchain
- Aucun redémarrage de geth n'est nécessaire pour la mise à jour

Vous pouvez vérifier la mise à jour avec :
```bash
cat ~/.ethereum/metadata.json | jq '.blockNumber'
```

## Code source

Les modifications se trouvent dans :
- `cmd/geth/chaincmd.go` : Définition de la commande et implémentation
- `cmd/geth/main.go` : Enregistrement de la commande

La fonctionnalité sous-jacente utilise `core/blockchain.go:SetHead()`.

## Voir aussi

- `geth dump` : Pour inspecter l'état à un bloc donné
- `geth export` : Pour exporter une plage de blocs
- `geth import` : Pour importer des blocs depuis un fichier

## Support

Pour toute question ou problème, consultez :
- La documentation officielle de go-ethereum
- Les issues sur GitHub

