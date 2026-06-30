# Exercise 08 - S.O.S

Encoding a string into Morse code using a lookup dictionary. The main things here are
building a mapping, validating input characters, and joining the output correctly.

- Dicts are the natural structure for character-to-code mappings
- `.join()` is the right way to build strings from lists — don't concatenate in a loop
- Spaces between words become `/` in Morse; spaces between characters within a word are just ` `

## The Morse Dictionary

Build a dict mapping each alphanumeric character to its Morse sequence:

```python
MORSE = {
    'A': '.-',   'B': '-...',  'C': '-.-.',  'D': '-..',
    'E': '.',    'F': '..-.',  'G': '--.',   'H': '....',
    'I': '..',   'J': '.---',  'K': '-.-',   'L': '.-..',
    'M': '--',   'N': '-.',    'O': '---',   'P': '.--.',
    'Q': '--.-', 'R': '.-.',   'S': '...',   'T': '-',
    'U': '..-',  'V': '...-',  'W': '.--',   'X': '-..-',
    'Y': '-.--', 'Z': '--..',
    '0': '-----', '1': '.----', '2': '..---', '3': '...--',
    '4': '....-', '5': '.....', '6': '-....', '7': '--...',
    '8': '---..', '9': '----.',
}
```

Reference: https://morsecode.world/international/morse2.html

## Input Validation

The exercise prints ERROR if any character isn't alphanumeric or a space. The `/`
character specifically causes ERROR (it's a Morse word separator in the output, not
a valid input character).

```python
text = ' '.join(sys.argv[1:]).upper()
for ch in text:
    if ch != ' ' and ch not in MORSE:
        print("ERROR")
        sys.exit()
```

## Encoding and Joining

The structure is: words separated by ` / `, characters within a word separated by ` `.

```python
words = text.split(' ')
encoded_words = [
    ' '.join(MORSE[ch] for ch in word)
    for word in words
]
print(' / '.join(encoded_words))
```

`text.split(' ')` splits on single spaces. Each word becomes a list of Morse codes
joined by spaces. The words are then joined by ` / `. This handles the structure
neatly without special-casing.

## Why .join() and Not String Concatenation

String concatenation in a loop creates a new string object every iteration —
O(n²) in total allocations. `.join()` computes the total length first and allocates
once. For short strings it doesn't matter, but it's the correct habit.

```python
# Don't do this
result = ""
for code in codes:
    result += code + " "

# Do this
result = ' '.join(codes)
```
