# Writing code

NENO (non-exhaustive, non-ordered) list of things to think about when writing code:

## Naming

Naming stuff is one of the most important aspects in software engineering. Everything must have a name - variables, functions, classes, modules, packages. Names should be as simple as possible, but not simpler than that. They say what the thing is, or does, they are consistent across the code base and they make reading code easier.

### Variables

Variable names should be explicit, and give enough information so that the reader knows what's going on. A good name is a name that doesn't make us ask questions. When you read this - `int duration = 1800;` - you ask questions. Duration of what? What's the unit? When you read this - `int half_hour_in_seconds = 1800;` - you know that this variable represents half an hour, and the unit is seconds. No need to ask questions. Also, if part of the name doesn't add additional information, remove it.  In `int half_hour_in_seconds_integer = 1800;`, the `_integer` suffix is not needed, because the type of the variable already says it's an integer.

### Functions

Ideally, you should be able to understand what a function does by reading its signature. The signature is the name, the parameters, and the return type. A good side-effect of this principle is that it forces you (or motivates you at least), to write good code.

First of all, it will be easier to respect the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle). According to this principle, each function must do one thing and one thing only. And it's very hard to find a good name for a function that does many things. Imagine a function that downloads a web page from the internet, counts the occurrences of each word and uploads the results to a database. How would you call this function? `FetchDataComputeOccurrencesUploadToDb`? That's a ridiculous name. This is also a good test for the quality of the code you write. If it's hard to come up with a name for a function, that's a [code smell](https://martinfowler.com/bliki/CodeSmell.html). In the example we just made, it's clear that the function does too many things and need to be split in three. Of course, at some higher level in the hierarchy you will have wrapper functions that do more than one thing. But in those cases, the function should only call the lower-level functions and do nothing else. And those lower-level functions must be logically related. If they are, then it will be easy to name the high-level one.

Also, functions with good names improve re-usability of your code. When a function does one thing there's a higher chance that the same use case will appear somewhere else and you will be able to re-use the code. In general, when a function is named well it will be easier to re-use it. A good name has to say what the function does. When I say what the function does I mean its input-output behaviour. The name should not mention what the function does internally, because that can change at any time. But the name should say anything that a user of the function should be aware of. Consider the following function:

```python
def write(data: Union[Dict, List], options: Dict[str, Any]) -> None:
    if "mode" not in options:
        options["mode"] = "w"
    options["path"].open(mode=options["mode"]).write_text(json.dumps(data))
```

This function writes a json file, but it also modifies the options dictionary. In Python, objects are passed by reference, which means:

```python
options = {}
write([1.0, 2.0], options=options)
print(options)
# {"mode": "w"}
```

This function has a side-effect on one of its parameters which can't be guessed by looking at its signature. We could either change the name to something that explains why the side-effect, or we can get rid of it altogether:

```python
def write(data: Union[Dict, List], path: pathlib.Path) -> None:
    path.open(mode="w").write_text(json.dumps(data))
```

This is another example of how following good naming principles has the great side-effect of making us write better code.

### Comments

Good code with good names doesn't need comments. When you have to add a comment to your code is because you failed to express that idea in the code itself. Of course, there are exceptions to this. For example, if you're forced to do something weird or not natural because of an external issue, that's a good place for a comment. Or to explain why you can't do something. In general, a good use of comments is when, if you had to express that idea in the code itself, the result would be messier. This is a good example of a comment:

```python
tree = KDTree(leaf_size=40)
for point in points:
    nearest_neighbor = tree.query(point, k=1)
```

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

## How can this go wrong?

TODO.
