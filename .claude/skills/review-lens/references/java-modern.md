# Lens: Modern Java (21 → 25)

## Scope
Files: `*.java` under `src/main` and `src/test`.
Related ADRs: 0000 (Java version from the catalog; `-Werror` on main code).
Note: the project runs on Java 25, so final features up to 25 are allowed. No preview features.

## Checks

### Records for data carriers
- Look for: classes with only final fields, constructor, getters, `equals`/`hashCode`/`toString` (handwritten or
  Lombok `@Value`/`@Data`); DTOs, request/response bodies, event payloads, value objects (e.g. money).
- Skip when: JPA entities. They need a no-arg constructor, mutable state and proxies, so records don't work.
- MiniBrack anchor: API DTOs, `Money(amount, currency)`, Kafka event payloads.
- Ask yourself: "Is this type's identity just its values?"

### Sealed types + exhaustive switch
- Look for: a closed set of variants modelled with an enum plus `if/else`, or with an open interface where every
  implementation is known; `default` branches that hide missing cases.
- Skip when: the set is meant to be open for extension (plugins, strategies registered by Spring).
- MiniBrack anchor: price rule kinds, deal states (`Scheduled | Active | SoldOut | Ended`), domain results instead of
  exceptions.
- Ask yourself: "If I add a variant, will the compiler show me every place to update?"

### Pattern matching (`instanceof`, `switch`, record patterns)
- Look for: `instanceof` followed by a cast; nested `if` chains on types; manual field extraction from records.
- Skip when: one simple check where the pattern adds nothing to readability.
- MiniBrack anchor: mapping domain results to HTTP responses.
- Ask yourself: "Can the shape of the data drive this branch directly?"

### Unnamed variables `_` (Java 22+)
- Look for: unused lambda parameters, unused catch variables, record pattern components you ignore.
- Skip when: naming the variable documents something useful.
- MiniBrack anchor: test code, event handlers.
- Ask yourself: "Does this name carry meaning, or is it noise?"

### Collections and streams
- Look for: `collect(Collectors.toList())` (→ `toList()`, unmodifiable); `list.get(list.size() - 1)` (→
  `getLast()`, SequencedCollection); `Optional` used as a field or parameter, or `.get()` without a check.
- Skip when: you need a mutable list from the stream on purpose.
- MiniBrack anchor: mapping product lists, picking the latest price.
- Ask yourself: "Does the caller expect to change this list?"

### Flexible constructor bodies (Java 25)
- Look for: static helper methods that only exist to validate or compute arguments before `super(...)`/`this(...)`.
- Skip when: no `super`/`this` call involved.
- MiniBrack anchor: value objects with validation.
- Ask yourself: "Can the validation sit directly in the constructor now?"

### Lombok vs language features
- Look for: Lombok used where the language already covers it (`@Value`/`@Data` on a non-entity → record; `@Getter` on
  a type that could be a record).
- Skip when: JPA entities (`@Getter`, `@NoArgsConstructor(access = PROTECTED)` are reasonable there). Avoid
  `@Data`/`@EqualsAndHashCode` on entities entirely (see transactions lens).
- MiniBrack anchor: `Product` entity vs product DTOs.
- Ask yourself: "Would this compile cleanly without Lombok using plain Java 25?"
