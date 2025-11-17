# Geth Rollback Command

## Quick Reference

The `rollback` command allows you to rewind the blockchain by removing recent blocks.

## Basic Usage

```bash
# Rollback by 1 block (default)
geth rollback

# Rollback by multiple blocks
geth rollback 5

# Rollback with custom datadir
geth rollback --datadir /path/to/data 3
```

## What it does

- Removes the specified number of blocks from the top of the blockchain
- Sets the chain head to the target block
- Deletes associated data (headers, bodies, receipts, state)

## ⚠️ Warning

This is a **destructive operation**. Always backup your data before using this command.

```bash
# Backup your data first
cp -r ~/.ethereum ~/.ethereum.backup

# Then rollback
geth rollback
```

## Examples

### Fix corrupted block

```bash
geth rollback
```

### Revert to a previous state

```bash
geth rollback 10
```

### Check current block before rollback

```bash
geth attach
> eth.blockNumber
> exit

geth rollback 5
```

## Error Handling

- **Cannot rollback genesis block**: Already at block 0
- **Cannot rollback X blocks**: Requested rollback exceeds chain length
- **No current block found**: Database is empty or corrupted

## After Rollback

When you restart geth, it will automatically resync with the network to download any missing blocks.

```bash
# After rollback
geth
```

## Technical Details

The command uses the underlying `SetHead` method from `core/blockchain.go` to safely rewind the blockchain state.

## See Also

- `geth dump [blockNum]` - Inspect state at a specific block
- `geth export <file> [first] [last]` - Export blocks to file
- `geth import <file>` - Import blocks from file

