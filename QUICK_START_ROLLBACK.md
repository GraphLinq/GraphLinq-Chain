# 🚀 Démarrage Rapide - Commande Rollback

## Installation

La commande est déjà intégrée dans geth. Il suffit de compiler :

```bash
cd /Users/jeremyguyet/project/glq/go-ethereum
go build -o build/bin/geth ./cmd/geth
```

## Utilisation de base

### 1. Sauvegarder vos données (IMPORTANT !)

```bash
# Arrêter geth s'il tourne
killall geth

# Sauvegarder votre datadir
cp -r ~/.ethereum ~/.ethereum.backup
```

### 2. Vérifier le bloc actuel

```bash
# Méthode 1 : Via geth console
geth attach
> eth.blockNumber
> exit

# Méthode 2 : Via les métadonnées de la DB
geth db metadata
```

### 3. Effectuer le rollback

```bash
# Rollback d'un seul bloc
geth rollback

# Rollback de 5 blocs
geth rollback 5

# Avec un datadir personnalisé
geth rollback --datadir /custom/path 3
```

### 4. Vérifier le résultat

```bash
# Méthode 1 : Via geth console
geth attach
> eth.blockNumber  // Devrait être réduit du nombre de blocs rollback
> exit

# Méthode 2 : Via metadata.json (plus rapide)
cat ~/.ethereum/metadata.json | jq '.blockNumber'
```

**Note** : Le fichier `metadata.json` est automatiquement mis à jour après le rollback !

## Exemples pratiques

### Scénario 1 : Supprimer le dernier bloc corrompu

```bash
# 1. Arrêter geth
killall geth

# 2. Sauvegarder
cp -r ~/.ethereum ~/.ethereum.backup

# 3. Rollback
geth rollback

# 4. Redémarrer geth (il se resynchronisera)
geth
```

### Scénario 2 : Revenir à un état antérieur pour tests

```bash
# Vous êtes au bloc 1000 et voulez revenir au bloc 995
geth rollback 5

# Faire vos tests...

# Ensuite, resynchroniser en redémarrant geth
geth
```

### Scénario 3 : Mode développement avec réseau privé

```bash
# Initialiser une blockchain de test
geth --datadir ./testnet init genesis.json

# Lancer et créer des blocs
geth --datadir ./testnet --mine console

# ... créer quelques blocs ...
# exit

# Rollback pour tests
geth --datadir ./testnet rollback 3
```

## Vérifications de sécurité

Avant de faire un rollback en production :

```bash
# 1. Vérifier l'espace disque disponible
df -h

# 2. Vérifier le numéro de bloc actuel
geth attach --exec "eth.blockNumber"

# 3. Vérifier qu'il y a assez de blocs
# Si vous êtes au bloc 10 et voulez rollback 15 blocs → ERREUR

# 4. Sauvegarder
tar -czf ethereum-backup-$(date +%Y%m%d).tar.gz ~/.ethereum/
```

## Résolution de problèmes

### Erreur : "Cannot rollback genesis block"

**Cause** : Vous êtes au bloc 0  
**Solution** : Aucun rollback possible depuis le genesis

### Erreur : "Cannot rollback X blocks: current block is #Y"

**Cause** : Vous demandez de rollback plus de blocs qu'il n'en existe  
**Solution** : Réduisez le nombre de blocs ou vérifiez le bloc actuel

### Erreur : "No current block found"

**Cause** : La base de données est vide ou corrompue  
**Solution** : Réinitialisez avec `geth init genesis.json`

### La blockchain ne se resynchronise pas après rollback

**Cause** : Geth n'est pas connecté au réseau  
**Solution** : 
```bash
# Vérifier les peers
geth attach --exec "admin.peers"

# Ajouter manuellement un peer si besoin
geth attach --exec 'admin.addPeer("enode://...")'
```

## Scripts utiles

### Backup automatique avant rollback

```bash
#!/bin/bash
# backup_and_rollback.sh

DATADIR=~/.ethereum
BACKUP_DIR=~/.ethereum.backups
BLOCKS=${1:-1}

# Créer le répertoire de backup
mkdir -p $BACKUP_DIR

# Backup avec timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
echo "Sauvegarde en cours..."
cp -r $DATADIR "$BACKUP_DIR/backup_$TIMESTAMP"

# Rollback
echo "Rollback de $BLOCKS bloc(s)..."
geth rollback $BLOCKS

echo "Terminé ! Backup dans : $BACKUP_DIR/backup_$TIMESTAMP"
```

### Vérifier le statut de la blockchain

```bash
#!/bin/bash
# check_status.sh

geth attach --exec "
  console.log('Bloc actuel:', eth.blockNumber);
  console.log('En sync:', eth.syncing);
  console.log('Peers:', admin.peers.length);
"
```

## Aide supplémentaire

```bash
# Aide complète
geth rollback --help

# Documentation
cat ROLLBACK_COMMAND.md

# Tests
./test_rollback.sh
```

## Support

- 📚 Documentation complète : `ROLLBACK_COMMAND.md`
- 🔧 Détails techniques : `CHANGES_ROLLBACK.md`
- 🌐 Référence EN : `docs/ROLLBACK_USAGE.md`
- 🧪 Script de test : `test_rollback.sh`

---

**⚠️ Rappel** : Toujours sauvegarder avant un rollback !

