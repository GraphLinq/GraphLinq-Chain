# Mise à jour du metadata.json lors du rollback

## Problème résolu

Précédemment, lors d'un rollback de la blockchain avec la commande `geth rollback`, le fichier `metadata.json` n'était pas mis à jour et conservait les informations de l'ancien bloc (avant le rollback).

## Solution implémentée

### 1. Ajout d'une méthode publique dans `core/blockchain.go`

```go
// WriteBlockMetadata is a public wrapper for writeBlockMetadata
// It allows external packages to update the metadata.json file
func (bc *BlockChain) WriteBlockMetadata(block *types.Block) {
    bc.writeBlockMetadata(block)
}
```

Cette méthode publique permet aux packages externes (comme `cmd/geth`) d'appeler la fonction de mise à jour du metadata.json.

### 2. Mise à jour de la fonction rollback dans `cmd/geth/chaincmd.go`

La fonction `rollback` a été modifiée pour appeler `WriteBlockMetadata` après le `SetHead` :

```go
newBlock := chain.CurrentBlock()

// Update metadata.json file with the new head block
chain.WriteBlockMetadata(newBlock)

log.Info("Rollback successful", "current_block", newBlock.NumberU64(), "current_hash", newBlock.Hash())
fmt.Printf("Successfully rolled back from block #%d to block #%d (%d blocks)\n", currentNumber, newBlock.NumberU64(), blocksToRollback)
fmt.Printf("metadata.json updated with new head block #%d\n", newBlock.NumberU64())
```

## Comportement après le rollback

Après un rollback, le fichier `metadata.json` sera automatiquement mis à jour avec les informations du nouveau bloc courant.

### Exemple

**Avant le rollback** (bloc #1000) :
```json
{
  "blockNumber": 1000,
  "blockHash": "0xabc123...",
  "parentHash": "0xdef456...",
  "timestamp": 1699876543,
  "gasUsed": 12345678,
  "gasLimit": 30000000,
  "difficulty": "12345678901234567890",
  "miner": "0x1234567890abcdef1234567890abcdef12345678",
  "txCount": 150
}
```

**Commande** :
```bash
geth rollback 5
```

**Après le rollback** (bloc #995) :
```json
{
  "blockNumber": 995,
  "blockHash": "0xghi789...",
  "parentHash": "0xjkl012...",
  "timestamp": 1699876000,
  "gasUsed": 11234567,
  "gasLimit": 30000000,
  "difficulty": "12345678901234567890",
  "miner": "0xabcdef1234567890abcdef1234567890abcdef12",
  "txCount": 145
}
```

**Sortie de la commande** :
```
Successfully rolled back from block #1000 to block #995 (5 blocks)
metadata.json updated with new head block #995
```

## Vérification

Pour vérifier que le metadata.json a bien été mis à jour après un rollback :

```bash
# Effectuer un rollback
geth rollback 3

# Vérifier le contenu du metadata.json
cat ~/.ethereum/metadata.json | jq '.blockNumber'

# Ou voir toutes les infos
cat ~/.ethereum/metadata.json | jq '.'
```

## Modifications techniques

### Fichiers modifiés

1. **`core/blockchain.go`**
   - Ajout de la méthode publique `WriteBlockMetadata()` (ligne ~908-912)

2. **`cmd/geth/chaincmd.go`**
   - Ajout de l'appel à `chain.WriteBlockMetadata(newBlock)` après le rollback (ligne ~560-561)
   - Ajout d'un message de confirmation (ligne ~565)

## Avantages

- ✅ Le fichier `metadata.json` reste toujours synchronisé avec l'état réel de la blockchain
- ✅ Les scripts de monitoring peuvent faire confiance aux données du metadata.json même après un rollback
- ✅ Pas besoin de redémarrer geth pour que le metadata.json soit mis à jour
- ✅ Message de confirmation dans la sortie de la commande

## Compatibilité

Cette modification est **rétrocompatible** :
- La méthode `WriteBlockMetadata()` est un simple wrapper public
- La méthode privée `writeBlockMetadata()` n'a pas changé
- Aucun impact sur le comportement normal de la blockchain
- Aucun impact sur les performances

## Tests

Pour tester la fonctionnalité :

```bash
# 1. Vérifier le bloc actuel
cat ~/.ethereum/metadata.json | jq '.blockNumber'

# 2. Effectuer un rollback
geth rollback 2

# 3. Vérifier que le metadata.json a été mis à jour
cat ~/.ethereum/metadata.json | jq '.blockNumber'
# Devrait afficher blockNumber - 2
```

## Conclusion

Le fichier `metadata.json` est maintenant correctement mis à jour lors d'un rollback, garantissant que les informations affichées correspondent toujours à l'état réel de la blockchain.

