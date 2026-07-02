Introspection: inspecting and modifying an object's attributes at runtime using
`dir()`, `hasattr()`, `getattr()`, `setattr()`, and `delattr()`. The exercise teaches
that Python objects are flexible — attributes can be added, removed, and checked
dynamically.

- `dir(obj)` returns a list of all attribute and method names on an object
- `hasattr(obj, 'name')` returns True if the attribute exists
- `getattr(obj, 'name')` gets the value; `setattr(obj, 'name', val)` sets it; `delattr(obj, 'name')` removes it
- `obj.__dict__` is the instance's attribute dict — you can iterate, check, and modify it directly

## Reading the Account Class

The given `Account` uses `**kwargs` and `self.__dict__.update(kwargs)` — this dumps
any keyword argument directly into the instance's attributes. So
`Account("Alice", zip="1010", addr="Main St")` will have `self.zip` and `self.addr`
alongside `self.name`, `self.id`, `self.value`.

```python
acc = Account("Alice", zip="1010", addr="Main St", bref="problem")
print(acc.__dict__)
# {'zip': '1010', 'addr': 'Main St', 'bref': 'problem', 'id': 1, 'name': 'Alice', 'value': 0}
```

## Corruption Rules

A corrupted account has any of these:
- even number of attributes
- any attribute name starting with `b`
- no attribute starting with `zip` OR no attribute starting with `addr`
- missing `name`, `id`, or `value`
- `name` not a `str`, `id` not an `int`, `value` not an `int` or `float`

```python
def is_corrupted(self, account):
    attrs = account.__dict__
    if len(attrs) % 2 == 0:
        return True
    if any(k.startswith('b') for k in attrs):
        return True
    if not any(k.startswith('zip') for k in attrs):
        return True
    if not any(k.startswith('addr') for k in attrs):
        return True
    if not all(k in attrs for k in ('name', 'id', 'value')):
        return True
    if not isinstance(attrs.get('name'), str):
        return True
    if not isinstance(attrs.get('id'), int):
        return True
    if not isinstance(attrs.get('value'), (int, float)):
        return True
    return False
```

## Bank Class

```python
class Bank:
    def __init__(self):
        self.accounts = []

    def add(self, new_account):
        if not isinstance(new_account, Account):
            return False
        if any(a.name == new_account.name for a in self.accounts):
            return False
        self.accounts.append(new_account)
        return True

    def transfer(self, origin, dest, amount):
        src = next((a for a in self.accounts if a.name == origin), None)
        dst = next((a for a in self.accounts if a.name == dest), None)
        if src is None or dst is None:
            return False
        if amount < 0 or amount > src.value:
            return False
        if self.is_corrupted(src) or self.is_corrupted(dst):
            return False
        if origin == dest:
            return True   # valid but no movement
        src.transfer(-amount)
        dst.transfer(amount)
        return True
```

## fix_account

`fix_account` needs to repair a corrupted account by adding or removing attributes.
The key operations are `setattr` to add missing ones and `delattr` to remove bad ones:

```python
def fix_account(self, name):
    if not isinstance(name, str):
        return False
    account = next((a for a in self.accounts if a.name == name), None)
    if account is None:
        return False

    # Remove attributes starting with 'b'
    for key in list(account.__dict__.keys()):
        if key.startswith('b'):
            delattr(account, key)

    # Add addr if missing
    if not any(k.startswith('addr') for k in account.__dict__):
        setattr(account, 'addr', '')

    # Add zip if missing
    if not any(k.startswith('zip') for k in account.__dict__):
        setattr(account, 'zip', '')

    # Fix odd attribute count if needed
    if len(account.__dict__) % 2 == 0:
        setattr(account, 'fix', True)

    return not self.is_corrupted(account)
```

## dir() vs __dict__

`obj.__dict__` only contains instance attributes — things set on `self`.
`dir(obj)` returns everything: instance attributes, class attributes, inherited
methods, and all the dunder methods. For checking corruption, `__dict__` is what
you want — it's just the instance's own attributes.

```python
account.__dict__.keys()   # {'name', 'id', 'value', 'zip', 'addr'}
dir(account)              # also includes 'transfer', '__init__', '__str__', ...
```
