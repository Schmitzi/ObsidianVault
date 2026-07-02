An interactive guessing game introducing `random.randint` and the general pattern
for game loops: generate state, loop on input, handle bad input, check win/exit condition.

- `random.randint(a, b)` returns a random integer N where a ≤ N ≤ b (inclusive on both ends)
- Non-numeric input should be caught and reported, not crash the program
- Attempt counting happens regardless of whether the guess is valid

## Generating the Secret Number

```python
from random import randint

secret = randint(1, 99)
```

This needs to happen before the loop starts and never change during the game.

## The Game Loop

```python
attempts = 0

while True:
    guess = input("What's your guess between 1 and 99?\n>> ")

    if guess == "exit":
        print("Goodbye!")
        break

    try:
        n = int(guess)
    except ValueError:
        print("That's not a number.")
        continue   # don't increment attempts, go back to top of loop

    attempts += 1

    if n < secret:
        print("Too low!")
    elif n > secret:
        print("Too high!")
    else:
        # Win condition
        if secret == 42:
            print("The answer to the ultimate question of life, the universe and everything is 42.")
        if attempts == 1:
            print("Congratulations! You got it on your first try!")
        else:
            print(f"Congratulations, you've got it!\nYou won in {attempts} attempts!")
        break
```

## Gotchas

**Invalid input and attempt count**: The subject example shows "That's not a number"
doesn't increment the attempt counter — the user types `A` and still wins in 5
attempts (4 numeric guesses + 1 non-numeric). Use `continue` to skip the rest of
the loop body without counting.

**Inclusive randint**: `randint(1, 99)` can return 1 and can return 99. This is
different from Python's `range(1, 99)` which excludes the upper bound. For random
number generation, inclusive ranges are usually what you want.

**First try detection**: Check `attempts == 1` after incrementing. At the point of
a correct guess, `attempts` has already been incremented, so `attempts == 1` means
it was the first numeric guess.

**Douglas Adams easter egg**: Check `secret == 42` at win time, not at guess time.
The easter egg should trigger whether or not they guessed it on the first try.
