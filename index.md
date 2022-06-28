class: center, middle
        
# `hypothesis` & Property-Based Testing

---

# Software testing

---

## A software tester walks into a bar...

Runs into a bar.

Crawls into a bar.

Dances into a bar.

Flies into a bar.

Jumps into a bar.

And orders:

- a beer.
- 2 beers.
- 0 beers.
- 99999999 beers.
- a lizard in a beer glass.
- -1 beer.
- "qwertyuiop" beers.

Testing complete.

---

## A real customer walks into the bar...

... and asks where the bathroom is.

--

The bar goes up in flames.

---

# Testing methods

---

## example test

```python
def login(password):
    return password == "valid"

def test_login():
    assert login("valid")

```

---

# Property-based Testing

---

# `hypothesis`

---
## What is `hypothesis`

https://hypothesis.works/



---
## how does it work?

---

# `hypothesis` in Starfish (examples)

---

## hypothesis in CI

* how `hypothesis` caches examples in CI?

---

## naming strategies

```python
from hypothesis import given
from hypothesis.strategies import one_of, text, just

invalid_passwords = text().filter(lambda x: x != TEST_VALID_PASSWORD)

@given(one_of(invalid_passwords, just("-:")))
def test_login_failed_with_wrong_password(password):
    assert login(password) is False
```
