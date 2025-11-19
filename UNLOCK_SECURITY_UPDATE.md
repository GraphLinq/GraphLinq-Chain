# Modifications de sécurité : Flag --unlock et méthodes RPC dangereuses

## Résumé des changements

Cette modification améliore considérablement la sécurité en :
1. Séparant clairement les méthodes de déverrouillage au démarrage et via RPC
2. Désactivant complètement les méthodes RPC dangereuses qui permettent la signature côté serveur

## Changements principaux

### 1. Flag `--unlock` au démarrage (✅ Autorisé)

Le flag `--unlock` fonctionne maintenant **toujours** au démarrage du nœud, indépendamment de la configuration RPC (HTTP/WebSocket).

**Exemples d'utilisation :**
```bash
# Démarrage avec RPC HTTP activé - FONCTIONNE
./geth --http --http.addr "127.0.0.1" --unlock "0xVotreAdresse"

# Démarrage avec RPC sur interface publique - FONCTIONNE
./geth --http --http.addr "0.0.0.0" --unlock "0xVotreAdresse"

# Démarrage sans RPC - FONCTIONNE
./geth --unlock "0xVotreAdresse"
```

### 2. Méthode RPC `personal.unlockAccount` (❌ Désactivée)

La méthode RPC `personal.unlockAccount` est maintenant **complètement désactivée** pour des raisons de sécurité, peu importe la configuration.

**Avant :**
- Pouvait être activée avec `--allow-insecure-unlock`
- Permettait de déverrouiller des comptes après le démarrage via HTTP/WebSocket
- Risque de sécurité : mots de passe transmis via le réseau

**Après :**
- ❌ Toujours bloquée via RPC, même avec `--allow-insecure-unlock`
- Retourne l'erreur : `"account unlock via RPC is disabled for security - use --unlock flag at startup instead"`
- ✅ Les comptes doivent être déverrouillés au démarrage uniquement

### 3. Flag `--allow-insecure-unlock` (⚠️ Deprecated)

Le flag `--allow-insecure-unlock` n'a maintenant plus d'effet sur la méthode `personal.unlockAccount` car celle-ci est complètement désactivée.

### 4. Méthodes RPC de signature côté serveur (❌ DÉSACTIVÉES)

Les méthodes RPC suivantes sont maintenant **complètement désactivées** pour des raisons de sécurité :

- **`eth_sign`** : Signature de messages arbitraires avec un compte déverrouillé
- **`eth_signTransaction`** : Signature de transactions avec un compte déverrouillé
- **`eth_sendTransaction`** : Signature et envoi de transactions avec un compte déverrouillé

**Pourquoi ces méthodes sont dangereuses :**
- Elles nécessitent que les clés privées soient stockées et déverrouillées sur le serveur
- `eth_sign` peut être exploité pour signer des transactions déguisées en messages
- Elles exposent un risque majeur en cas de compromission du serveur
- Elles violent les meilleures pratiques de sécurité Web3

**Erreurs retournées :**
```
"eth_sign has been disabled for security reasons - use client-side signing (e.g., MetaMask, hardware wallets) instead"
"eth_signTransaction has been disabled for security reasons - use client-side signing (e.g., MetaMask, hardware wallets) instead"
"eth_sendTransaction has been disabled for security reasons - use eth_sendRawTransaction with client-side signing (e.g., MetaMask, hardware wallets) instead"
```

**Méthodes alternatives sécurisées :**
- ✅ `eth_sendRawTransaction` : Envoi de transactions signées côté client
- ✅ Client-side signing avec MetaMask, Ledger, Trezor, WalletConnect
- ✅ Signature locale via console IPC si nécessaire

## Fichiers modifiés

1. **`cmd/geth/main.go`** (fonction `unlockAccounts`)
   - Suppression de la vérification restrictive au démarrage
   - Les comptes peuvent toujours être déverrouillés au démarrage

2. **`internal/ethapi/api.go`**
   - **Méthode `PersonalAccountAPI.UnlockAccount`** : Blocage complet via RPC
   - **Méthode `TransactionAPI.Sign` (eth_sign)** : Complètement désactivée
   - **Méthode `TransactionAPI.SignTransaction` (eth_signTransaction)** : Complètement désactivée
   - **Méthode `TransactionAPI.SendTransaction` (eth_sendTransaction)** : Complètement désactivée
   - Messages d'erreur informatifs avec alternatives sécurisées

3. **`cmd/utils/flags.go`** (définition du flag)
   - Mise à jour de la documentation du flag `--allow-insecure-unlock`
   - Indication que `personal.unlockAccount` est désactivée

4. **`node/config.go`**
   - Aucune modification fonctionnelle (nettoyage du code)

## Avantages de sécurité

1. **Protection des clés privées** : Les clés ne peuvent plus être utilisées pour signer via RPC
2. **Protection contre eth_sign phishing** : Impossible d'exploiter `eth_sign` pour signer des transactions déguisées
3. **Défense en profondeur** : Même si un compte est déverrouillé, il ne peut pas signer via RPC
4. **Conformité aux meilleures pratiques** : Force l'utilisation de la signature côté client
5. **Réduction drastique de la surface d'attaque** : Pas de signature côté serveur = pas de risque
6. **Workflow moderne et sécurisé** : Encourage l'utilisation de wallets modernes (MetaMask, hardware wallets)

## Migration

### 1. Migration de personal.unlockAccount

**Ancien workflow :**
```javascript
// Via JavaScript console ou RPC
personal.unlockAccount("0xAddress", "password", 3600)
```

**Nouveau workflow :**
```bash
# Au démarrage du nœud (uniquement pour cas d'usage spécifiques)
./geth --unlock "0xAddress" --password /chemin/vers/password.txt
```

### 2. Migration de eth_sign, eth_signTransaction, eth_sendTransaction

**❌ Ancien workflow (INSÉCURE - ne fonctionne plus) :**
```javascript
// Signature côté serveur - DANGEREUX
const signature = await web3.eth.sign(message, account)
const hash = await web3.eth.sendTransaction({from, to, value})
```

**✅ Nouveau workflow (SÉCURISÉ - Recommandé) :**

#### Option A : Utiliser MetaMask ou un wallet moderne (Production)
```javascript
// Demander à l'utilisateur de signer avec son wallet
const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' })
const signature = await window.ethereum.request({
  method: 'personal_sign',
  params: [message, accounts[0]]
})

// Envoyer une transaction
const txHash = await window.ethereum.request({
  method: 'eth_sendTransaction',
  params: [{from: accounts[0], to: to, value: value}]
})
```

#### Option B : Signature côté client avec bibliothèque (Backend)
```javascript
const ethers = require('ethers')

// Signer côté client avec une clé privée
const wallet = new ethers.Wallet(privateKey, provider)
const signature = await wallet.signMessage(message)

// Envoyer une transaction signée
const tx = await wallet.sendTransaction({to, value})
```

#### Option C : Utiliser eth_sendRawTransaction (Backend)
```javascript
const ethers = require('ethers')

// 1. Construire la transaction
const tx = {
  to: to,
  value: value,
  gasLimit: 21000,
  gasPrice: await provider.getGasPrice(),
  nonce: await provider.getTransactionCount(address)
}

// 2. Signer localement (JAMAIS sur le serveur geth)
const wallet = new ethers.Wallet(privateKey)
const signedTx = await wallet.signTransaction(tx)

// 3. Envoyer la transaction signée via RPC
const txHash = await provider.sendTransaction(signedTx)
```

#### Option D : Hardware Wallet (Maximum de sécurité)
```javascript
// Utiliser Ledger/Trezor pour la signature
import TransportWebUSB from '@ledgerhq/hw-transport-webusb'
import Eth from '@ledgerhq/hw-app-eth'

const transport = await TransportWebUSB.create()
const eth = new Eth(transport)
const signature = await eth.signPersonalMessage(path, message)
```

## Compatibilité

**✅ Continue de fonctionner :**
- Le flag `--unlock` au démarrage
- Les scripts de démarrage existants
- `eth_sendRawTransaction` (transactions pré-signées)
- `eth_call`, `eth_estimateGas` et autres méthodes de lecture
- Tous les RPC qui ne nécessitent pas de signature

**❌ Ne fonctionne plus (migration requise) :**
- `personal.unlockAccount` via RPC
- `eth_sign` (remplacer par signature client-side)
- `eth_signTransaction` (remplacer par signature client-side)
- `eth_sendTransaction` (remplacer par `eth_sendRawTransaction` + signature client-side)

**⚠️ Conservé pour compatibilité (sans effet) :**
- Le flag `--allow-insecure-unlock`

## Notes techniques

### Méthodes désactivées

Les méthodes suivantes retournent immédiatement une erreur sans traitement :
- `personal.unlockAccount` : Bloquée via tout RPC
- `eth_sign` : Complètement désactivée
- `eth_signTransaction` : Complètement désactivée
- `eth_sendTransaction` : Complètement désactivée

### Pourquoi ces changements sont nécessaires

**Problèmes de sécurité avec eth_sign :**
- `eth_sign` signe des données arbitraires, ce qui peut être exploité
- Un attaquant peut présenter une transaction comme un message innocent
- Une fois signé, cela peut être rejoué comme une transaction valide
- C'est l'une des attaques de phishing les plus courantes dans Web3

**Problèmes de sécurité avec eth_sendTransaction :**
- Nécessite que les clés privées soient stockées sur le serveur
- Les clés doivent être déverrouillées, augmentant le risque d'exposition
- En cas de compromission du serveur, toutes les clés sont à risque
- Viole le principe "not your keys, not your coins"

**Architecture sécurisée moderne :**
```
❌ Ancien : Client → Serveur geth (signe) → Blockchain
✅ Nouveau : Client (signe) → Serveur geth → Blockchain
```

### Impact sur les applications

**Applications Web (dApps) :**
- ✅ Doivent déjà utiliser MetaMask/WalletConnect → Aucun impact
- ❌ Si elles utilisaient eth_sendTransaction → Migration requise

**Scripts Backend :**
- ✅ Peuvent utiliser des bibliothèques comme ethers.js pour signer localement
- ✅ Envoyer via `eth_sendRawTransaction`
- ❌ Plus de signature côté geth

**Outils de développement :**
- ✅ Utiliser un wallet de développement local (ex: Hardhat, Foundry)
- ✅ Console IPC geth continue de fonctionner pour les tests locaux

