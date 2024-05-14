+++
title = 'The best type checker for Python'
date = 2024-05-10T23:21:57+02:00
draft = true
+++

I've struggled choosing the best type checker for Python. Let's compare them and decide which one is the best.

## Requirements

The choice of a type checker depends on several factors. I've set the following requirements:

- Maintainability
- Integration with the IDE
- Ease of use
- Popularity

## Candidates

I've picked several candidates for comparison based on their popularity and the number of stars on GitHub.
The initial analysis of their repositories provides the following statistics:
Name     | Maintainer       | Language   | Github stars  | Opened/closed issues
---------|------------------|------------|---------------|----------------------
Mypy     | Python Community | Python     | 17.5k         | 2.6k/7.6k
Pyright  | Microsoft        | TypeScript | 12k           | 32/5.4k
Pyre     | Meta             | OCaml      | 6.7k          | 129/260
Pytype   | Google           | Python     | 4.5k          | 151/540

We can see that Mypy is the most popular type checker and is maintained by the Python community. Thus, we can be sure that it will be supported for a long time.

If we take a look at the number of issues, we can see that Mypy has the most open issues, indicating that backlog isn't being cleared efficiently. In contrast, Pyright has the fewest number of open issues and is catching up with Mypy in the number of closed issues.

Pyre and Pytype checkers are not as popular as the first two. They appear to be not as widely used in the community. I cannot say that they are bad, maybe they are used in specific projects or companies. However, I want to easily find solutions to any problem. So, let's exclude them from the following comparison.

## Detailed comparison

Let's compare Mypy and Pyright more closely.

### 1. Integration with the IDE

Mypy has good integration with popular IDE such as VS Code, PyCharm, and Vim through plugins. If you use an exotic IDE, you can still utilize Mypy via the CLI, although it might be less convenient. For a more seamless integration, you can wrap Mypy within an LSP using the `python-lsp` tool.

Pyright also offers plugins for VS Code and PyCharm, and it also includes built-in LSP server. If your IDE supports LSP, you can use Pyright directly without additional plugins. The LSP also enhances functionality with features like autocompletion, go-to definition, and renaming capabilities.

### 2. Plugins

From the Pyright documentation:
> Mypy supports a plug-in mechanism, whereas pyright does not. Mypy plugins allow developers to extend Mypy’s capabilities to accommodate libraries that rely on behaviors that cannot be described using the standard type checking mechanisms.
>
> Pyright maintainers have made the decision not to support plug-ins because of their many downsides: discoverability, maintainability, cost of development for the plug-in author, cost of maintenance for the plug-in object model and API, security, performance (especially latency — which is critical for language servers), and robustness.
>
> Instead, we have taken the approach of working with the typing community and library authors to extend the type system so it can accommodate more use cases. An example of this is PEP 681, which introduced `dataclass_transform`.

I've never had any problems with type checking using Pyright in non-strict mode. It seems to be sufficient for most cases.

### 3. Type checking approach

I don't want to provide a full list of differences between the approaches of Mypy and Pyright. You can do it yourself just by checking the [documentation](https://microsoft.github.io/pyright/#/mypy-comparison).

I want to mention only the main differences:

__Unions vs Joins__

When merging two types during code flow analysis or widening types during constraint solving, pyright always uses a union operation. Mypy typically (but not always) uses a “join” operation, which merges types by finding a common supertype.

```python
def func1(val: object):
    if isinstance(val, str):
        pass
    elif isinstance(val, int):
        pass
    else:
        return
    reveal_type(val) # mypy: object, pyright: str | int

def func2(condition: bool, val1: str, val2: int):
    x = val1 if condition else val2
    reveal_type(x) # mypy: object, pyright: str | int

    y = val1 or val2
    # In this case, mypy uses a union instead of a join
    reveal_type(y) # mypy: str | int, pyright: str | int
```

__Type Guards__

Type Guards are a way to narrow the type of a variable based on a condition.

Pyright has slightly better support for type guards than Mypy. For example, Pyright can narrow the type in following situations:
- `x == L` and `x != L` (where `L` is an expression with a literal type)
- `x in y` or `x not in y` (where `y` is instance of `list`, `set`, `frozenset`, `deque`, `tuple`, `dict`, `defaultdict`, or `OrderedDict`)
- `bool(x)` (where `x` is any expression that is statically verifiable to be truthy or falsey in all cases)

__Class and Instance Variables__

Pyright distinguishes between “pure class variables”, “regular class variables”, and “pure instance variable”. Mypy does not distinguish between class variables and instance variables in all cases.

```python
class A:
    x: int = 0 # Regular class variable
    y: ClassVar[int] = 0 # Pure class variable

    def __init__(self):
        self.z = 0 # Pure instance variable

print(A.x)
print(A.y)
print(A.z) # pyright: error, mypy: no error
```

__Circular References__

Mypy is able to deal with certain forms of circular references that Pyright cannot handle. In this case Mypy works correctly, but Pyright raises an error.
```python
T = TypeVar("T", bound="A")
class A(Generic[T]): ...
```

### 4. Stub generation

Both Mypy and Pyright can generate stubs for libraries. Mypy uses a tool called `stubgen` for this purpose, while Pyright includes a flag within the `pyright` tool to generate stubs.

Mypy does not infer the return type of functions from the code itself. In contrast, Pyright can do so and includes types in the stubs under comments for subsequent review by developers.

You can utilize Pyright to generate stubs for libraries and then use Mypy to perform type checking.

### 5. Support for new features

Pyright is a clear leader in this category, often implementing new features faster than Mypy.

For example, Mypy does not yet support the following PEPs:
- [PEP 695](https://peps.python.org/pep-0695/) - introduces new syntax for declaring type parameters for generic classes, functions, and type aliases using square brackets.
- [PEP 688](https://peps.python.org/pep-0688/) - proposes the implementation of the buffer protocol and adds an abstract class `Buffer` for objects that implement the `__buffer__` method.
- [PEP 675](https://peps.python.org/pep-0675/) - supports the `LiteralString` type, which is used to prevent security issues associated with passing SQL queries directly to a database.

Pyright has integrated support for all these PEPs, having implemented them ahead of the stable Python versions that include these features.

### 6. Ease of use

It easy to use both type checkers, as they both work well right out of the box and offer flexible configuration options.

In non-strict mode, Pyright can handle most problems with minimal configuration. However, if strict mode is needed, Mypy tends to be superior. It supports numerous plugins and, in my experience, provides slightly better error messages.

## Conclusion

> Which type checker is the best for Python?
>
> It depends.

If you are writing production code at work, I would recommend using Mypy. It has many plugins, and you can easily find a solution to most of the problems. Of course, you can't start using new features right away after a new Python version is released. However, this is usually not a significant problem, as it's uncommon to need the latest Python version in production.

Pyright is a good choice for personal projects or writing libraries, particularly when not using heavy dependencies like `boto3` or `django`. The lack of plugins plays a significant role in the choice of a type checker. However, Pyright is growing quickly, and I believe it will be the best type checker in the future. If you want to use new Python features immediately after they are released, Pyright is the best choice.

Of course, you can use both of them: Pyright in non-strict mode as an LSP and for generating stubs, and Mypy in strict mode for type checking.
