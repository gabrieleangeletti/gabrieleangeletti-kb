# Writing code

NENO (non-exhaustive, non-ordered) list of things to think about when writing code:

## Abstract common tasks to reduce bugs

When you write code that does the same thing over and over it's time to abstract it. It's good to re-use code and it decreases the chances for bugs. When you write the same thing over and over, at some point you will forget something. For example, consider this code:

```python
CHUNK_SIZE = 200

partial_results = []
for i, row in enumerate(data):
    partial_results.append(process_row(row))

    if i > 0 and i % CHUNK_SIZE == 0:
        upload_results(partial_results)
        partial_results = []
```

Here we forgot to upload the latest results. If data has `437` records, we upload the first `400` and forget about the last `37`. This kind of task is general enough that it should be abstracted (this one is already in the std library but that's not the point), because by the fifth you re-implement you will make mistakes.

## Boring is good

This is bad:

```python
data = [
    line.strip().split("\t") for line in open("my_file.tab")
    if not line.startswith("#")
]
```

* Too much going on in a single line.
* The cognitive load is higher when so much stuff is happening at once.
* Is the file even closed at the end? (It is, but it's not obvious, you need to know some internals of Python).
* If I want to do additional processing on the line, where do I put that code?

This is good:

```python
data = []

with open("my_file.tab") as file:
    for line in file:
        if line.startswith("#"):
            continue

        line = line.strip()
        data.append(line.split("\t"))
```

* Less cognitive load, easier to read.
* Even if I don't know anything about Python, it's easy to guess that the file will be closed after it's done.
* It's easy to add additional code.

## Testing as documentation

TODO.

## Test as protection against future changes

TODO.

## Naming

TODO.

## How can this go wrong?

TODO.
