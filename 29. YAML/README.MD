# YAML

A superset of JSON. A data serialisation language, to present data as text.  

Has 2 styles, flow and block, flow is more like JSON. Use block.  

## Building blocks
- Sequences (lists)
- Mapping (key value pairs)
- Scalars (e.g., strings)

## Indentation
Indentation is spaces (default 2), never tabs.  

## Sequences
Sequences are arrays or lists.

### Block style
Indicated by a dash.
```
- first time
- second item
- third item
```

### Flow style
Separated by commas.
```
[first time, second item, third item]
```

## Mappings
Key value pair, indicated by a colon. Can also contain lists assigned to a key.  

### Block style
```
key: value
list:
  - first item
  - second item
  - third item
flowlist: [just, a, list]

# Key can have spaces wrapped in quotes
"some key": value

# Number
version: 1.2

```

### Flow style
```
{ key: value, another: value, flowlist: [just, a, list] }
```

## Scalars
Basic types, numbers, string, booleans. Duplicate keys are not allowed.  
For strings, we can use spaces without quotes, or add quotes of preferred.  
Can only escape something if using double quotes.  

### Block style
```
tool: yaml
awesome: true
version: 1.2
nothing: null
duplicateKeys: not allowed

escape: "she said \"escaping can only be done in double quotes\""
alternativeEscape: 'use single quotes to "escape also"'

octal: 012 # Starts with a zero, so e.g. a phone number would have to be wrapped in quotes.
hexadecimal: 0xABC

boolean: true
nothing: null
```

## Comments
Comment lines start with a hashtag
```
# this is a comment

#Between quotes is not a comment
"# somekey": value

# Separate YAML documents are separated by 3 quotes

---
document: one
---
document: two
```


## Multilines
You can wrap lines in YAML using the chevron symbol.
```
multi-line: >
  Some test
  and more on another line

  the white space above introduces a newline \n in the compiled yaml
```
Other symbols:  
> multiline but removes newlines (without blank line)  
>- Keep newlines but trim at the end  
>+ Keep newlines but don't trim at the end  
| Keep all newlines and whitespace  
|- remove trailing lines  
|+ Keep trailing lines  

## Nested Sequences
List of lists / sequences of sequences.  
```
- 
  - a
  - b
  - c
-
  - d
  - e
```

## Nested mappings
An object containing objects, or maps.  
```
person:
  name: David
  age: 0
  address:
    streetname: Some street
    number: 0
    city: Some city
```

## Combining sequences and mappings
```
persons:
  - name: David
    age: 0
    address:
      streetname: Some street
      number: 0
      city: Some city
  - name: Eve
    age: 0
    address:
      streetname: Some street
      number: 0
      city: Some city
```

## Dates
```
timestamp: 2024-02-25T15:05.10+:00:00
simple_date: 2024-02-25

# Using the !!str means it wil be interpretted as a string not a date
not_a_date: !!str 2024-02-25
```

## Repeated Nodes
Use & to declare value to repeat, and * to repeat it.  
```
key: &repeated Hi there
new_key: *repeated

person: &person
  name: David
  age: 0
  address:
    streetname: Some street
    number: 0
    city: Some city
another_person: *person

# To override some value in the repeated node, use <<

yet_another_person:
  <<: *person
  name: Rob
```

## Tags
```
no_date: !!str 2023-01-01
no_number: !!str 20
unnecessary: !!int 20
phone: !!str 07111111123 # Use the !!str to prevent the number being interpretted as an octal
```
