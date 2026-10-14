# Python and Kotlin Mapping

Use this reference when a migration needs explicit type, control-flow, mutability, or error-behavior decisions. It is guidance, not a promise that every construct has a direct equivalent.

## Core principles

- Make important runtime assumptions explicit in Kotlin types and validation.
- Prefer immutable values and typed data classes for stable contracts.
- Preserve Python's observable behavior before optimizing allocation or syntax.
- Put conversions at clear boundaries rather than relying on implicit numeric or nullable conversions.

## Common mappings

| Python | Kotlin candidate | Review points |
| --- | --- | --- |
| `int` | `Int` or `Long` | Range, overflow, serialization, and division behavior. |
| `float` | `Float` or `Double` | Precision, model API requirements, and tolerance. |
| `bool` | `Boolean` | Python truthiness is broader than Kotlin Boolean checks. |
| `str` | `String` | Encoding, normalization, and nullable values. |
| `None` | `null` | Use nullable types and define absent versus empty semantics. |
| `list[T]` | `List<T>`, `MutableList<T>`, or typed array | Boxing and mutation can affect performance and behavior. |
| `tuple` | Data class, `Pair`/`Triple`, or typed list | Prefer named fields when values have business meaning. |
| `dict` | Data class or `Map<K, V>` | Define key presence, ordering, and value types. |
| `set` | `Set<T>` | Ordering must not be assumed unless Python code sorts it. |
| generator | `Sequence<T>` or loop | Check one-shot behavior and lazy versus eager evaluation. |
| exception | Specific Kotlin exception or sealed result | Preserve which failures are recoverable and how callers observe them. |

## Semantics to check

- Python's `/` produces floating-point division, while integer division in Kotlin requires deliberate handling.
- Python negative indices and slices need explicit bounds logic in Kotlin.
- Python truthiness of empty collections, zero, and `None` does not map automatically to Kotlin Boolean expressions.
- Python dictionaries preserve insertion order in current implementations; use `LinkedHashMap` or a data class only when order is observable.
- Python integers have arbitrary precision; Kotlin fixed-width integers need range checks for large values.
- Shallow versus deep copy, aliasing, and mutation can affect downstream results.
- `NaN`, infinity, signed zero, and equality behavior should be tested when numerical outputs matter.

## Public API guidance

Use data classes for stable result schemas. Keep platform-specific input adapters, such as `Bitmap` or `ByteBuffer`, separate from pure algorithm types when possible. Make ownership, thread expectations, and lifecycle explicit in documentation.
