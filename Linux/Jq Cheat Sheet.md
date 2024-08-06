`jq` is a lightweight and flexible command-line JSON processor.

# Reading JSON Data
```bash
cat data.json | jq .
```
OR
```bash
jq . data.json
```

# Simple Filtering
```bash
echo '{"name": "John", "age": 30}' | jq '.name'
```

# Nested Fields
```bash
echo '{"user": {"name": "John", "age": 30}}' | jq '.user.name'
```

# Filtering Arrays
```bash
echo '[{"name": "John", "age": 30}, {"name": "Jane", "age": 25}]' | jq '.[] | select(.age > 28)'
```

# Mapping Values
```bash
echo '[{"name": "John", "age": 30}, {"name": "Jane", "age": 25}]' | jq 'map(.age)'
```

# Combining Filters
```bash
echo '{"user": {"name": "John", "age": 30}, "location": "USA"}' | jq '.user | .name'
```

# Modifying JSON
```bash
echo '{"name": "John"}' | jq '. + {age: 30}'
```

# Compact the JSON
```bash
echo '{"name": "John", "age": 30}' | jq -c .
```

# Creating JSON Data
```bash
jq -n --arg name "John" --argjson age 30 '{"name": $name, "age": $age}'
```

