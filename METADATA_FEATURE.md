# Fonctionnalité metadata.json

## Description

Cette fonctionnalité ajoute un fichier `metadata.json` qui contient les informations du dernier bloc validé dans la blockchain. Ce fichier est automatiquement mis à jour à chaque nouveau bloc canonique ajouté à la chaîne.

## Emplacement du fichier

Le fichier `metadata.json` est créé dans le répertoire de données de votre nœud Ethereum (dataDir), typiquement :
- Linux: `~/.ethereum/metadata.json`
- macOS: `~/Library/Ethereum/metadata.json`
- Windows: `%APPDATA%\Ethereum\metadata.json`

## Format du fichier

```json
{
  "blockNumber": 1234567,
  "blockHash": "0x1234567890abcdef...",
  "parentHash": "0xabcdef1234567890...",
  "timestamp": 1699876543,
  "gasUsed": 12345678,
  "gasLimit": 30000000,
  "difficulty": "12345678901234567890",
  "miner": "0x1234567890abcdef1234567890abcdef12345678",
  "txCount": 150
}
```

## Champs

- **blockNumber**: Numéro du bloc (hauteur de la blockchain)
- **blockHash**: Hash du bloc actuel
- **parentHash**: Hash du bloc parent
- **timestamp**: Timestamp Unix du bloc
- **gasUsed**: Quantité de gas utilisée dans le bloc
- **gasLimit**: Limite de gas du bloc
- **difficulty**: Difficulté du bloc (sous forme de chaîne)
- **miner**: Adresse du mineur qui a validé le bloc
- **txCount**: Nombre de transactions dans le bloc

## Utilisation

Ce fichier est particulièrement utile pour :
- Surveiller rapidement la hauteur actuelle de la blockchain sans interroger le nœud
- Scripts de monitoring qui ont besoin de connaître l'état de synchronisation
- Applications externes qui ont besoin d'informations sur le dernier bloc validé
- Vérification rapide de l'état du nœud

## Modifications apportées

Les modifications suivantes ont été apportées au code source de go-ethereum :

### 1. core/blockchain.go
- Ajout du champ `dataDir` à la structure `BlockChain`
- Modification de la signature de `NewBlockChain` pour accepter le paramètre `dataDir`
- Ajout de la fonction `writeBlockMetadata` qui écrit le fichier metadata.json
- Modification de `writeHeadBlock` pour appeler `writeBlockMetadata` après chaque mise à jour du bloc de tête

### 2. eth/backend.go
- Mise à jour de l'appel à `core.NewBlockChain` pour passer `stack.Config().DataDir`

### 3. Fichiers de test
Tous les fichiers de test ont été mis à jour pour passer un `dataDir` vide `""` lors de l'appel à `NewBlockChain`.

## Exemple d'utilisation

```bash
# Lire la hauteur actuelle de la blockchain
cat ~/.ethereum/metadata.json | jq '.blockNumber'

# Vérifier le dernier bloc
cat ~/.ethereum/metadata.json | jq '.'
```

## Notes

- Le fichier est mis à jour uniquement pour les blocs canoniques (non pour les blocs de side-chain)
- Si le dataDir n'est pas configuré, le fichier ne sera pas créé
- Le fichier est écrit de manière atomique pour éviter les corruptions
- Les erreurs d'écriture sont loguées mais ne bloquent pas le fonctionnement de la blockchain

