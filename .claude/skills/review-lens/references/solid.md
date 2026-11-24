# Lens: SOLID

## Scope
Files: `*.java` under `src/main`. Ignore: tests, `package-info.java`, Spring `@Configuration`, migrations.
Related ADRs: 0000 (package by feature, package-private by default).

## Checks

### SRP: Single Responsibility
- Look for: a class that changes for more than one reason, e.g. a service that validates, maps DTOs, computes
  prices and sends events; a controller with business rules; an entity that calls repositories or HTTP.
- Skip when: the class is small (< ~100 lines) and the "responsibilities" always change together; splitting would
  produce classes with one method that only delegate.
- MiniBrack anchor: product service vs pricing vs stock. They change for different business reasons.
- Ask yourself: "Who would ask me to change this class? Is it one person/team or several?"

### OCP: Open/Closed
- Look for: `switch`/`if` chains on a type, enum or string that grow with each new case; adding behaviour by editing
  an existing, tested class.
- Skip when: ≤ 2 cases and none planned; an exhaustive `switch` over a sealed type or enum (the compiler forces you to
  handle new cases, which is often better than a strategy registry).
- MiniBrack anchor: price rules (regular, discount, flash deal, bundle) → one strategy per rule.
- Ask yourself: "What would I edit to add the next case? Could I add it without touching existing code?"

### LSP: Liskov Substitution
- Look for: subclasses or implementations that throw `UnsupportedOperationException`, ignore arguments, weaken
  guarantees or need `instanceof` checks by callers.
- Skip when: there is no inheritance or only one implementation.
- MiniBrack anchor: a `PriceRule` that "doesn't apply" should return "no change", not throw.
- Ask yourself: "Can every caller use any implementation without knowing which one it got?"

### ISP: Interface Segregation
- Look for: interfaces where implementers leave methods empty or throwing; callers depending on a wide interface but
  using one method.
- Skip when: the interface is Spring Data (`JpaRepository`). Its width is a framework choice, not your design.
- MiniBrack anchor: deal service needs "read price" only, not the whole catalog API.
- Ask yourself: "Does every user of this interface need every method?"

### DIP: Dependency Inversion
- Look for: domain/business logic constructing its own collaborators (`new SomeClient()`), depending on concrete
  infrastructure (HTTP clients, Kafka templates) directly; field injection (`@Autowired` on fields).
- Skip when: suggesting an extra interface with a single implementation inside one feature package. Constructor
  injection of the concrete class is enough until there is a second implementation or a module boundary.
- MiniBrack anchor: publishing catalog events. Business code shouldn't know it's Kafka.
- Ask yourself: "If this dependency changed technology, how much of my business code would change?"

### Visibility (supports all of the above)
- Look for: `public` classes or methods in a feature package that nothing outside the package uses.
- Skip when: Spring or JPA requires it (e.g. entity constructors for the JPA provider; most Spring beans work fine
  package-private).
- MiniBrack anchor: ADR 0000. A feature's public surface is its real API.
- Ask yourself: "Which of these types must another package see?"
