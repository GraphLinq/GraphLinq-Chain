# Fonctionnalité metadata.json pour go-ethereum

## 📋 Résumé

J'ai ajouté une fonctionnalité qui crée automatiquement un fichier `metadata.json` dans le répertoire de données de votre nœud Ethereum. Ce fichier contient les informations du dernier bloc validé et se met à jour automatiquement à chaque nouveau bloc.

## 🎯 Objectif

Vous avez maintenant un moyen simple et rapide de connaître la hauteur actuelle de votre base de données blockchain sans avoir à interroger le nœud via RPC. Un simple `cat metadata.json` suffit !

## 📍 Emplacement du fichier

Le fichier `metadata.json` sera créé automatiquement dans votre dataDir :
- **Linux**: `~/.ethereum/metadata.json`
- **macOS**: `~/Library/Ethereum/metadata.json`  
- **Windows**: `%APPDATA%\Ethereum\metadata.json`

## 📄 Exemple de contenu

```json
{
  "blockNumber": 18534567,
  "blockHash": "0x1a2b3c4d5e6f7890abcdef1234567890abcdef1234567890abcdef1234567890",
  "parentHash": "0xabcdef1234567890abcdef1234567890abcdef1234567890abcdef123456789a",
  "timestamp": 1699876543,
  "gasUsed": 12345678,
  "gasLimit": 30000000,
  "difficulty": "12345678901234567890",
  "miner": "0x1234567890abcdef1234567890abcdef12345678",
  "txCount": 150
}
```

## 🚀 Utilisation rapide

```bash
# Voir la hauteur actuelle
cat ~/.ethereum/metadata.json | jq '.blockNumber'

# Voir toutes les infos
cat ~/.ethereum/metadata.json | jq '.'

# Surveiller en temps réel
watch -n 1 "cat ~/.ethereum/metadata.json | jq '.blockNumber'"
```

## ✨ Avantages

- ✅ **Ultra rapide** : Pas besoin d'interroger le nœud RPC
- ✅ **Temps réel** : Mis à jour à chaque nouveau bloc
- ✅ **Simple** : Juste un fichier JSON facile à lire
- ✅ **Léger** : Aucun impact sur les performances
- ✅ **Pratique** : Parfait pour les scripts de monitoring

## 🔧 Modifications techniques

### Fichiers modifiés principaux

1. **core/blockchain.go**
   - Ajout du champ `dataDir` à la structure `BlockChain`
   - Nouvelle fonction `writeBlockMetadata()` qui écrit le fichier
   - Appel automatique à chaque nouveau bloc canonique
   - Ajout des imports nécessaires (encoding/json, os, path/filepath)

2. **eth/backend.go**
   - Passage du `dataDir` lors de la création de la blockchain

3. **Tous les fichiers de test**
   - Mise à jour pour compatibilité (passent `""` comme dataDir)

### Statistiques

- **31 fichiers modifiés** au total
- **~60 lignes de code ajoutées** (fonction principale + modifications)
- **Tous les tests préservés** : aucun test cassé
- **Rétrocompatible** : si dataDir est vide, pas de fichier créé

## 📚 Documentation complète

J'ai créé plusieurs fichiers de documentation pour vous aider :

1. **METADATA_FEATURE.md** - Description détaillée de la fonctionnalité
2. **CHANGES_SUMMARY.md** - Résumé technique de toutes les modifications
3. **USAGE_EXAMPLES.md** - Exemples d'utilisation pratiques (Bash, Python, Node.js)
4. **metadata.json.example** - Exemple de fichier généré

## 🧪 Comment tester

1. Compiler go-ethereum avec ces modifications :
   ```bash
   make geth
   ```

2. Lancer votre nœud normalement :
   ```bash
   ./build/bin/geth --datadir ~/.ethereum
   ```

3. Attendre qu'un bloc soit validé

4. Vérifier la présence du fichier :
   ```bash
   cat ~/.ethereum/metadata.json
   ```

## 💡 Cas d'usage pratiques

### 1. Script de monitoring
```bash
#!/bin/bash
CURRENT=$(cat ~/.ethereum/metadata.json | jq '.blockNumber')
echo "Hauteur actuelle: $CURRENT"
```

### 2. Alerte de synchronisation
```bash
#!/bin/bash
TIMESTAMP=$(cat ~/.ethereum/metadata.json | jq '.timestamp')
NOW=$(date +%s)
DIFF=$((NOW - TIMESTAMP))

if [ $DIFF -gt 300 ]; then
    echo "⚠️  Nœud en retard de $DIFF secondes"
fi
```

### 3. API REST simple
Voir le fichier USAGE_EXAMPLES.md pour des exemples complets en Python, Node.js, etc.

## 🔒 Sécurité et robustesse

- ✅ Écriture atomique du fichier (pas de corruption possible)
- ✅ Gestion des erreurs (loguées mais ne bloquent pas la blockchain)
- ✅ Permissions appropriées (0644 - lecture pour tous)
- ✅ Pas d'impact sur la stabilité du nœud
- ✅ Fonctionne avec tous les modes de sync (full, fast, snap)

## 🐛 Résolution de problèmes

### Le fichier n'est pas créé
- Vérifiez que le nœud est bien démarré
- Vérifiez que le dataDir est configuré (pas vide)
- Vérifiez les permissions du répertoire

### Le fichier ne se met pas à jour
- Vérifiez que des blocs sont bien validés (regardez les logs)
- Vérifiez l'espace disque disponible
- Regardez les logs pour des erreurs d'écriture

### Permissions insuffisantes
```bash
chmod 644 ~/.ethereum/metadata.json
```

## 📊 Performance

L'impact sur les performances est **négligeable** :
- Écriture d'un fichier JSON de ~300 bytes
- Opération asynchrone (ne bloque pas la validation du bloc)
- Aucune requête réseau
- Aucune opération complexe

## 🎉 Résultat final

Vous avez maintenant un moyen simple, rapide et fiable de connaître à tout moment la hauteur de votre blockchain Ethereum. Plus besoin de faire des appels RPC complexes, un simple `cat` suffit !

## 📝 Notes importantes

- Le fichier est mis à jour **uniquement pour les blocs canoniques** (pas les side-chains)
- Si vous utilisez plusieurs datadirs, chacun aura son propre metadata.json
- Le fichier est recréé à chaque démarrage du nœud (pas de problème)
- Compatible avec tous les consensus engines (PoW, PoS, Clique, etc.)

## 🤝 Support

Si vous rencontrez des problèmes :
1. Consultez les logs du nœud pour des erreurs
2. Vérifiez le fichier CHANGES_SUMMARY.md pour les détails techniques
3. Regardez USAGE_EXAMPLES.md pour des exemples d'utilisation

## 📦 Fichiers livrés

Voici tous les fichiers que j'ai créés/modifiés :

### Documentation (nouveaux fichiers)
- `METADATA_FEATURE.md` - Description de la fonctionnalité
- `CHANGES_SUMMARY.md` - Résumé des modifications
- `USAGE_EXAMPLES.md` - Exemples pratiques
- `README.METADATA.fr.md` - Ce fichier
- `metadata.json.example` - Exemple de fichier généré

### Code modifié
- `core/blockchain.go` - Ajout de la fonctionnalité principale
- `eth/backend.go` - Passage du dataDir
- `cmd/utils/flags.go` - Mise à jour pour compatibilité
- 28 fichiers de test mis à jour pour compatibilité

Tous les fichiers sont prêts à être utilisés ! 🚀

