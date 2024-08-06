`jq` is a lightweight and flexible command-line JSON processor.

# Reading JSON Data
```
cat data.json | jq .
```
OR
```
jq . data.json
```

# Simple Filtering
```
echo '{"name": "John", "age": 30}' | jq '.name'
```

# Nested Fields
```
echo '{"user": {"name": "John", "age": 30}}' | jq '.user.name'
```

# Filtering Arrays
```
echo '[{"name": "John", "age": 30}, {"name": "Jane", "age": 25}]' | jq '.[] | select(.age > 28)'
```

# Mapping Values
```
echo '[{"name": "John", "age": 30}, {"name": "Jane", "age": 25}]' | jq 'map(.age)'
```

# Combining Filters
```
echo '{"user": {"name": "John", "age": 30}, "location": "USA"}' | jq '.user | .name'
```

# Modifying JSON
```
echo '{"name": "John"}' | jq '. + {age: 30}'
```

# Compact the JSON
```
echo '{"name": "John", "age": 30}' | jq -c .
```

# Creating JSON Data
```
jq -n --arg name "John" --argjson age 30 '{"name": $name, "age": $age}'
```