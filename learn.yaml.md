# Learn yaml

## Writing string, number and boolean 
```yaml
---
# name string
name: Simran

# age number
age: 22

# working bool
working: True
```

## Force to be string
```yaml
---
# force 5.4 to be string 
height: !!str 5.4
```

## String Continuation 
```yaml
---
# it is same as writing: I am fine. I am healthy.
health: >
  I am fine.
  I am healthy.
```

## Multi Line String 
```yaml
---
hello_world_py: |
  def main():
    print("Hello World!")
```

# List
```yaml
---
tasks:
  - reception of consultancy
  - housewife
  - teaching
```

# Dictionary (Key-Value)
```yaml
---
address:
  municipality: Ithari
  ward: 08
  city: Ithari
  province: Koshi
  country: Nepal
```

# List of Objects
```yaml
---
friends:
  - name: Samir
    phone: 111
  - name: Deeya
    phone: 222
```

# List of Dictionaries
```yaml
---
family:
  parents:
    - name: Jane
      age: 50
    - name: John
      age: 52
  children:
    - name: Jimmy
      age: 22
    - name: Jenny
      age: 20
```  

# Anchor and Alias (resuse values)
```yaml
---
book_genre: &book_genre # anchor with &
  genre: self-help

books: 
  - name: Think and Grow Rich
    author: Napolean Hill
    <<: *book_genre # resuse of alias with << and *
  - name: Psychology of Money
    author: Morgan Housel
    <<: *book_genre # resuse of alias with << and *
```

