---
name: jq-json-reader
description: Read and analyze JSON files using jq for efficient parsing and filtering. Use this skill when reading JSON files (*.json), exploring JSON structure, extracting specific fields, or filtering JSON data. Provides intelligent jq query generation for common JSON operations.
---

# jq JSON Reader

## Overview

Use jq to efficiently read, parse, filter, and transform JSON files. This skill provides intelligent jq query generation for common JSON operations, avoiding loading entire large JSON files into context.

## When to Use This Skill

Use this skill when:
- Reading any JSON file (`.json`, `.jsonl`, etc.)
- Exploring the structure of a JSON file
- Extracting specific fields or values from JSON
- Filtering JSON arrays based on conditions
- Transforming JSON data
- Working with large JSON files where loading the entire file is inefficient

## Core Workflow

### 1. Initial Structure Analysis

When first encountering a JSON file, analyze its structure:

```bash
# Check if file is an array or object at root level
jq 'type' file.json

# Get top-level keys (for objects)
jq 'keys' file.json

# Get array length (for arrays)
jq 'length' file.json

# Preview structure with first few items
jq '.[0:3]' file.json        # First 3 items of array
jq 'to_entries | .[0:5]' file.json  # First 5 key-value pairs of object
```

### 2. Common Query Patterns

**Extract specific field:**
```bash
jq '.fieldName' file.json
jq '.nested.field' file.json
jq '.array[0].field' file.json
```

**Get all values of a field from array:**
```bash
jq '.[].name' file.json
jq '[.[].name]' file.json  # Wrap in array
```

**Filter array items:**
```bash
jq '.[] | select(.status == "active")' file.json
jq '[.[] | select(.count > 10)]' file.json
jq '.[] | select(.name | contains("test"))' file.json
```

**Transform data:**
```bash
jq '.[] | {id, name}' file.json  # Select specific fields
jq '.[] | {userId: .id, userName: .name}' file.json  # Rename fields
jq 'map({id, name})' file.json  # Same as above for arrays
```

**Count and aggregate:**
```bash
jq 'length' file.json  # Count items
jq '[.[].status] | unique' file.json  # Unique values
jq 'group_by(.type) | map({type: .[0].type, count: length})' file.json
```

### 3. Working with Large Files

For large JSON files, use streaming and limiting:

```bash
# Preview first N items
jq '.[0:10]' large.json

# Stream process (memory efficient)
jq -c '.[]' large.json | head -20

# Count without loading all
jq -c '.[]' large.json | wc -l

# Find specific item without loading all
jq '.[] | select(.id == "target")' large.json | head -1
```

### 4. Output Formatting

**Compact output:**
```bash
jq -c '.' file.json
```

**Pretty print:**
```bash
jq '.' file.json
```

**Raw strings (no quotes):**
```bash
jq -r '.name' file.json
```

**Tab-separated output:**
```bash
jq -r '.[] | [.id, .name] | @tsv' file.json
```

**CSV output:**
```bash
jq -r '.[] | [.id, .name] | @csv' file.json
```

### 5. Error Handling

**Handle missing fields:**
```bash
jq '.field // "default"' file.json
jq '.field? // empty' file.json
```

**Check if field exists:**
```bash
jq 'has("fieldName")' file.json
jq '.[] | select(has("optionalField"))' file.json
```

## Quick Reference

| Operation | jq Command |
|-----------|------------|
| Get type | `jq 'type'` |
| Get keys | `jq 'keys'` |
| Get length | `jq 'length'` |
| First N items | `jq '.[0:N]'` |
| Last N items | `jq '.[-N:]'` |
| Specific index | `jq '.[N]'` |
| All values of key | `jq '.[].key'` |
| Filter by condition | `jq '.[] \| select(.field == "value")'` |
| Unique values | `jq '[.[].field] \| unique'` |
| Sort by field | `jq 'sort_by(.field)'` |
| Group by field | `jq 'group_by(.field)'` |
| Map/transform | `jq 'map({newKey: .oldKey})'` |
| Flatten nested | `jq 'flatten'` |
| Combine objects | `jq 'add'` |

## Common Use Cases

### Exploring Unknown JSON Structure

```bash
# Step 1: What type is the root?
jq 'type' file.json

# Step 2: What are the keys/structure?
jq 'keys' file.json  # for object
jq '.[0] | keys' file.json  # for array of objects

# Step 3: Preview data
jq '.' file.json | head -50
```

### Finding Specific Data

```bash
# Find by ID
jq '.[] | select(.id == 123)' file.json

# Find by partial match
jq '.[] | select(.name | test("pattern"))' file.json

# Find by multiple conditions
jq '.[] | select(.status == "active" and .count > 0)' file.json
```

### Extracting Reports

```bash
# List all unique statuses with counts
jq 'group_by(.status) | map({status: .[0].status, count: length})' file.json

# Sum numeric field
jq '[.[].amount] | add' file.json

# Get min/max
jq '[.[].score] | max' file.json
```

### Modifying JSON (for piping, not in-place)

```bash
# Add field to all items
jq '.[] += {newField: "value"}' file.json

# Update field value
jq '(.[] | select(.id == 1)).name = "newName"' file.json

# Remove field
jq 'del(.[].unwantedField)' file.json
```

## Best Practices

1. **Always check structure first** before writing complex queries
2. **Use `.[0:N]` for previews** instead of loading entire large files
3. **Use `-c` for streaming** when processing line by line
4. **Use `-r` for raw output** when values will be used in scripts
5. **Use `//` for defaults** to handle missing fields gracefully
6. **Quote filter expressions** to avoid shell interpretation issues

## Integration with Other Tools

```bash
# Combine with grep for text search
jq -c '.[]' file.json | grep "pattern"

# Pipe to other commands
jq -r '.[].url' file.json | xargs curl

# Use with watch for live monitoring
watch "jq '.status' file.json"
```
