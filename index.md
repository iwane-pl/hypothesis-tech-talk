class: center, middle

# `hypothesis` & Property-Based Testing

---

# Software testing

---

## A software tester walks into a bar...

--

- Runs into a bar.
- Crawls into a bar.
- Dances into a bar.
- Flies into a bar.
- Jumps into a bar.

--

And orders:

- a beer.
- 2 beers.
- 0 beers.
- 99999999 beers.
- a lizard in a beer glass.
- -1 beer.
- "qwertyuiop" beers.

--

Testing complete.

---

## A real customer walks into the bar...

... and asks where the bathroom is.

--

The bar goes up in flames.

---

# Testing techniques

https://en.wikipedia.org/wiki/Test_design_technique

* Boundary Value Analysis
* Equivalence Partitioning
* State Transition
* Decision Table
* Error Guessing (exploratory)
* ...
* Property-based Testing

---

## Example test - what can we test here and how?

```python
class Warehouse:
    def __init__(self, stock):
        self.stock = stock

    def in_stock(self, item_name):
        return (item_name in self.stock) and (self.stock[item_name] > 0)

    def take_from_stock(self, item_name, quantity):
        if quantity <= self.stock[item_name]:
            self.stock[item_name] -= quantity
        else:
          raise Exception(f"Oversold {item_name}")

    def stock_count(self, item_name):
        return self.stock[item_name]
```
--

```python
@pytest.fixture
def prepare_warehouse():
    yield Warehouse({"shoes": 10, "hats": 2, "umbrellas": 0})

@pytest.mark.parametrize("quantity", (-1, 0, 1, 10, 11))
def test_warehouse(prepare_warehouse, quantity):
    wh = prepare_warehouse
    assert wh.take_from_stock("shoes", quantity)
```
???
Możemy się tu pobawić BVA i Equivalence Partitioning

Ale tu i tak jest dość mała przestrzeń szukania przykładów.

---

# Property-based Testing

We focus on testing _properties_ of code

A _property_ can be thought of as a requirement/part of a contract.

e.g.

    for all (a, b, c) strings
    the concatenation of a, b and c always contains b

--

But how do we test them?

---

# `hypothesis`

---
## What is `hypothesis`

https://hypothesis.works/

    Hypothesis is a modern implementation of property based testing, designed from the ground up for mainstream languages.
    
    Hypothesis runs your tests against a much wider range of scenarios than a human tester could, finding edge cases in 
    your code that you would otherwise have missed. It then turns them into simple and easy to understand failures that 
    save you time and money compared to fixing them if they slipped through the cracks and a user had run into them 
    instead.

---
## How can it help?

Expand the coverage by testing with more examples

---
## how does it work?

[Anatomy of a `hypothesis` test](https://hypothesis.works/articles/anatomy-of-a-test/)

* applies random examples guided by `@given` strategies (integers, strings, etc.) until failure or exhaustion
  * [What can you generate and how?](https://hypothesis.readthedocs.io/en/latest/data.html)
  * Exhaustion happens by default after 100 random examples (can be adjusted)
* if failing example is found, `hypothesis` tries to "shrink" it (find a simpler version of it)
  * e.g. `-10` -> `0`

---

# `hypothesis` in Starfish (examples)

```python
import pytest
import freezegun

@pytest.mark.parametrize(
    "expiry_date",
    ("2020022", "20200221141", "2020022214305",
     "qwwertyiop", "abcdefgh", "abcdefghijkl", "abcdefghijklmn",
     ),
)
def test_create_token_with_specified_valid_until_timestamp_format_wrong(run_sf_client, expiry_date):
    with freezegun.freeze_time("2002-02-20 00:00"):
        stdout, stderr = run_sf_client(
            [
                'apikey',
                'create',
                '--description',
                'for event monitor',
                '--valid-until',
                expiry_date,
                '--format',
                'valid_until',
            ],
            expected_result=ExitCodes.INVALID_PARAMETER,
        )
        assert "Invalid valid-until value format" in stderr

```

---
# `hypothesis` in Starfish (examples)

```python
import pytest
import freezegun

@pytest.mark.parametrize(
    "expiry_date",
    ("2020022", "20200221141", "2020022214305",
     "qwwertyiop", "abcdefgh", "abcdefghijkl", "abcdefghijklmn",
     ),  # got a bug (missed test case) here
)
def test_create_token_with_specified_valid_until_timestamp_format_wrong(run_sf_client, expiry_date):
    with freezegun.freeze_time("2002-02-20 00:00"):
        stdout, stderr = run_sf_client(
            [
                'apikey',
                'create',
                '--description',
                'for event monitor',
                '--valid-until',
                expiry_date,
                '--format',
                'valid_until',
            ],
            expected_result=ExitCodes.INVALID_PARAMETER,
        )
        assert "Invalid valid-until value format" in stderr

```

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
