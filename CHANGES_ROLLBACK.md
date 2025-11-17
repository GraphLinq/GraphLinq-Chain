# Implémentation de la commande `rollback` pour Geth

## Résumé

Cette modification ajoute une nouvelle commande `rollback` à geth qui permet de revenir en arrière dans la blockchain en supprimant un ou plusieurs blocs récents.

## Modifications apportées

### 1. Fichiers modifiés

#### `cmd/geth/chaincmd.go`

**Ajouts :**
- Nouvelle commande CLI `rollbackCommand` (lignes 167-187)
- Nouvelle fonction `rollback()` (lignes 515-562)

**Fonctionnalités :**
- Permet de spécifier le nombre de blocs à revenir en arrière (par défaut : 1)
- Validation des entrées (nombre positif, ne pas dépasser le genesis)
- Utilisation de la méthode `SetHead()` de la blockchain
- Messages de log et affichage utilisateur

#### `cmd/geth/main.go`

**Modification :**
- Ajout de `rollbackCommand` dans la liste des commandes (ligne 220)

### 2. Fichiers modifiés pour metadata.json

#### `core/blockchain.go`

**Ajout de la méthode publique `WriteBlockMetadata()` :**
- Permet aux packages externes d'appeler la mise à jour du metadata.json
- Wrapper public de la méthode privée `writeBlockMetadata()`

```go
// WriteBlockMetadata is a public wrapper for writeBlockMetadata
// It allows external packages to update the metadata.json file
func (bc *BlockChain) WriteBlockMetadata(block *types.Block) {
    bc.writeBlockMetadata(block)
}
```

### 3. Documentation créée

#### `ROLLBACK_COMMAND.md` (Français)
Documentation complète en français comprenant :
- Description détaillée
- Exemples d'utilisation
- Cas d'utilisation courants
- Avertissements et précautions
- Guide de dépannage
- Détails techniques
- **Information sur la mise à jour du metadata.json**

#### `ROLLBACK_METADATA_UPDATE.md`
Documentation spécifique sur la mise à jour du metadata.json :
- Problème résolu
- Solution implémentée
- Exemples avant/après
- Vérification

#### `docs/ROLLBACK_USAGE.md` (Anglais)
Guide de référence rapide en anglais avec :
- Syntaxe de base
- Exemples pratiques
- Gestion des erreurs
- Références croisées

### 3. Script de test

#### `test_rollback.sh`
Script de test automatisé qui :
- Crée une blockchain de test
- Génère des blocs
- Teste le rollback d'un bloc
- Teste le rollback de plusieurs blocs
- Vérifie la gestion des erreurs

## Utilisation

### Syntaxe de base

```bash
geth rollback [nombre_de_blocs]
```

### Exemples

```bash
# Revenir d'un bloc en arrière
geth rollback

# Revenir de 5 blocs en arrière
geth rollback 5

# Avec un datadir personnalisé
geth rollback --datadir /path/to/data 3
```

## Fonctionnalités

### Validations implémentées

1. **Vérification du bloc genesis**
   - Empêche le rollback du bloc 0
   
2. **Validation du nombre de blocs**
   - Doit être > 0
   - Ne peut pas dépasser le nombre de blocs existants
   
3. **Vérification de l'état de la blockchain**
   - Détecte si la blockchain est vide
   - Affiche le bloc actuel avant rollback

### Mise à jour automatique du metadata.json

⭐ **NOUVEAU** : Lors d'un rollback, le fichier `metadata.json` est automatiquement mis à jour !

**Avantages :**
- ✅ Le fichier `metadata.json` reste synchronisé avec l'état réel de la blockchain
- ✅ Les scripts de monitoring fonctionnent correctement après un rollback
- ✅ Pas besoin de redémarrer geth pour mettre à jour les métadonnées
- ✅ Message de confirmation affiché dans la console

**Implémentation :**
```go
// Après le rollback
newBlock := chain.CurrentBlock()
chain.WriteBlockMetadata(newBlock)  // Mise à jour du metadata.json
```

**Vérification :**
```bash
geth rollback 3
cat ~/.ethereum/metadata.json | jq '.blockNumber'
```

### Gestion des erreurs

- Messages d'erreur clairs et informatifs
- Vérification de tous les paramètres avant exécution
- Utilisation de `utils.Fatalf()` pour les erreurs critiques

### Logging

- Log WARNING lors du début du rollback
- Log INFO après succès
- Affichage console avec les numéros de blocs avant/après

## Architecture technique

### Flux d'exécution

```
1. Parse arguments CLI
2. Initialize node config
3. Open blockchain database
4. Get current block number
5. Validate rollback parameters
6. Call blockchain.SetHead(target)
7. Verify new block number
8. Display results
```

### Méthodes utilisées

- `utils.MakeChain()` : Création de l'instance blockchain
- `chain.CurrentBlock()` : Obtenir le bloc actuel
- `chain.SetHead(target)` : Effectuer le rollback
- `rawdb.*` : Lectures de la base de données

### Dépendances

- `github.com/ethereum/go-ethereum/core` : Blockchain core
- `github.com/ethereum/go-ethereum/core/rawdb` : Raw database access
- `github.com/ethereum/go-ethereum/cmd/utils` : CLI utilities
- `github.com/urfave/cli/v2` : CLI framework

## Tests

### Compilation

```bash
go build -o build/bin/geth ./cmd/geth
```

### Vérification de la commande

```bash
./build/bin/geth help | grep rollback
./build/bin/geth rollback --help
```

### Test automatisé

```bash
./test_rollback.sh
```

## Sécurité

### Précautions prises

1. **Validation stricte des paramètres**
   - Empêche les valeurs invalides
   - Protège contre le rollback au-delà du genesis

2. **Messages d'avertissement**
   - Documentation claire sur la nature destructive
   - Recommandation de sauvegarde

3. **Opérations atomiques**
   - Utilisation de `SetHead()` qui gère les transactions

### Recommandations d'utilisation

1. **Toujours sauvegarder** avant un rollback
2. **Arrêter geth** avant d'exécuter la commande
3. **Vérifier le numéro de bloc** avant et après
4. **Comprendre les implications** sur le réseau

## Cas d'utilisation

### 1. Développement et tests
- Revenir à un état antérieur pour tester
- Réinitialiser après des tests

### 2. Correction d'erreurs
- Supprimer un bloc corrompu
- Récupérer d'une synchronisation défaillante

### 3. Maintenance
- Réorganisation de la chaîne
- Migration de données

## Améliorations futures possibles

1. **Mode interactif**
   - Confirmation avant rollback
   - Affichage des blocs qui seront supprimés

2. **Mode dry-run**
   - Simulation sans modification
   - Estimation de l'impact

3. **Sauvegarde automatique**
   - Backup avant rollback
   - Option de restauration

4. **Support des ranges**
   - Rollback jusqu'à un hash spécifique
   - Rollback jusqu'à un timestamp

5. **Statistiques détaillées**
   - Nombre de transactions affectées
   - État des données supprimées

## Compatibilité

- ✅ Compatible avec toutes les versions de Geth basées sur cette structure
- ✅ Fonctionne avec tous les modes de sync (full, snap, light)
- ✅ Compatible avec les réseaux personnalisés
- ✅ Fonctionne avec ancient database

## Références

### Code source
- `core/blockchain.go` : Méthode `SetHead()`
- `core/headerchain.go` : Méthode `setHead()`
- `core/rawdb/` : Opérations sur la base de données

### Documentation go-ethereum
- Architecture de la blockchain
- Gestion de l'état
- Organisation des données

## Auteur et Date

- **Date** : Novembre 2025
- **Version de go-ethereum** : Fork basé sur go-ethereum
- **Fonctionnalité** : Nouvelle commande CLI `rollback`

## Licence

Ce code est sous la même licence que go-ethereum (GNU General Public License v3.0).

