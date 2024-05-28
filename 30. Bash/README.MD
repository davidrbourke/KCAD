# Bash

## Output to console
`echo "Hello"`  

## Variables
```
greet=HelloGreet

echo $greet

HelloGreet
```
 
## Maths
```
expr 3 + 6

9
```
 
## Expression into variable
```
result=$((3 + 7))

echo $result

10
```
 
## Read input
```
read a

(entered Hi)

echo $a

Hi
```
 
## Read input with prompt
```
read -p "Enter your age" age

(entered 5)

echo $age

5
```

## One line IF statement

- Need the semi-colons to execute each part of the statement on a one liner
- must have spaces around the square brackets, e.g. [ 3 -eq 3 ] will work, but not [3 -eq 3]

```
if [ 3 -eq 3 ]; then echo 'yes'; else echo 'no'; fi;

outputs: yes
```

## Using else-if (elif)
```
if [ 3 -eq 2 ]; then echo 'yes'; elif [ 3 -lt 4 ]; then echo 'found'; else echo 'no'; fi;

outputs: found
```

## Conditions
```
-eq equal

-ge greater or equal

-gt greater than

-le less or equal

-lt less than

-ne not equal
```

## For loops
```
for i in {1..5}; do echo $i; done;

Outputs:

1

2

3

4

5
```

## Loops through strings

```
for aStr in hi this is david; do echo $aStr; done;

Outputs:

hi

this

is

david
````

## While loop
```
i=1; while [[ $i -le 5 ]]; do echo $i; (( i += 1 )); done;

Outputs:

1

2

3

4

5
```

## File operations

### Write to a file >
```
echo "test" > filename.txt
```

### Append to file >>
```
echo "test 2" >> filename.txt
```
### Read a file  <
```
echo $(< f1.txt)
```
 
### Read a file into a variable
```
fromFile=$( < f1.txt )
```