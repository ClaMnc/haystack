# Dataclasses

* Status: accepted
* Deciders: @tholor
* Date: 2021-10-14

Technical Story: https://github.com/deepset-ai/haystack/pull/1598

## Context and Problem Statement

Originally we implemented Haystack's primitive based on Python's vanilla `dataclasses`. However, shortly after we realized this causes issues with FastAPI, which uses Pydantic's implementation. We need to decide which version (vanilla Python's or Pydantic's) to use in our codebase.

## Decision Drivers 

* The Swagger autogenerated documentation for REST API in FastAPI was broken where the dataclasses include non-standard fields (`pd.dataframe` + `np.ndarray`)

## Considered Options

* Switch to Pydantic `dataclasses` in our codebase as well.
* Staying with vanilla `dataclasses` and find a workaround for FastAPI to accept them in place of Pydantic's implementation.

## Decision Outcome

Chosen option: **1**, because our initial concerns about speed proved negligible and Pydantic's implementation provided some additional functionality for free (see below).

### Positive Consequences

* We can now inherit directly from the primitives in the REST API dataclasses, and overwrite the problematic fields with standard types.
* We now get runtime type checks "for free", as this is a core feature of Pydantic's implementation.

### Negative Consequences

* Pydantic dataclasses are slower. See https://github.com/deepset-ai/haystack/pull/1598 for a rough performance assessment.
* Pydantic dataclasses do not play nice with mypy and autocomplete tools unaided. In many cases a complex import statement, such as the following, is needed:

```python
if typing.TYPE_CHECKING:
    from dataclasses import dataclass
else:
    from pydantic.dataclasses import dataclass
```

## Pros and Cons of the Options

### Switch to Pydantic `dataclasses`

* Good, because it solves the issue without having to find workarounds for FastAPI.
* Good, because it adds type checks at runtime.
* Bad, because mypy and autocomplete tools need assistance to parse its dataclasses properly. Example:

```python
if typing.TYPE_CHECKING:
    from dataclasses import dataclass
else:
    from pydantic.dataclasses import dataclass
```

* Bad, because it introduces an additional dependency to Haystack (negligible)
* Bad, because it adds some overhead on the creation of primitives (negligible)

### Staying with vanilla `dataclasses`

* Good, because it's Python's standard way to generate data classes
* Good, because mypy can deal with them without plugins or other tricks.
* Good, because it's faster than Pydantic's implementation.
* Bad, because does not play well with FastAPI and Swagger (critical).
* Bad, because it has no validation at runtime (negligible)

## Links <!-- optional -->

* https://pydantic-docs.helpmanual.io/usage/dataclasses/
* https://github.com/deepset-ai/haystack/pull/1598
* https://github.com/deepset-ai/haystack/issues/1593
* https://github.com/deepset-ai/haystack/issues/1582
* https://github.com/deepset-ai/haystack/pull/1398 
* https://github.com/deepset-ai/haystack/issues/1232

<!-- markdownlint-disable-file MD013 -->