# Phase 1 – Modern Java

**Time:** about 18–24 hours over weeks 1–3  
**Goal:** learn modern Java by building a well-tested, framework-independent Fritz game engine.

This phase is for an experienced C# and TypeScript developer. Use familiar concepts as a starting point, but learn Java's own type system, APIs, conventions, and trade-offs rather than translating C# syntax line by line. Use the JDK 21 and IDE setup from [Phase 0](phase-0-tooling.md). The engine will later become part of the Spring Boot backend.

## 1. Set up a small Java project

Create a Maven project in `backend/`, using Java 21 and the Maven wrapper. Keep the game rules in ordinary Java classes with no Spring or database dependencies. Use a package such as `com.example.fritz.game`; Java package names are conventionally lowercase and map to directories under `src/main/java`.

The matching tests go under `src/test/java`. Add JUnit Jupiter, AssertJ, and Mockito as test-scoped dependencies in the Maven project. Use the IDE's Maven support or run tests from the project directory:

```shell
./mvnw test
```

Commit the wrapper and build files along with the project. This makes the Java version, dependencies, and test command reproducible for the next phase.

## 2. Java foundations for C# developers

### Packages, classes, and interfaces

A Java package groups types and controls name qualification; it is similar to a C# namespace, but the package declaration and directory structure are normally kept aligned. A public top-level class is normally in a file with the same name.

Use classes for objects with identity or behavior that changes over time. Use interfaces to describe capabilities and keep callers independent of a particular implementation. Java classes have single class inheritance but can implement multiple interfaces. Access is expressed with `public`, `protected`, package-private (no modifier), and `private`; package-private is often useful for implementation details shared within a package.

**Exercise:** create a `Player` enum, a `Position` type, and an interface for a legal game action. Put each public type in its own appropriately named file and import it from a second package.

### Generics and type erasure

Java generics provide compile-time type safety, much like C# generics, but most generic type information is erased at runtime. Consequently, you cannot write `new T()`, and `value instanceof List<String>` is not allowed: the runtime can check `List`, but not its erased type argument. Prefer a typed collection such as `List<Move>` over a raw `List`, and use a `Class<T>` token when an API genuinely needs a runtime type.

**Exercise:** represent a sequence of moves with `List<Move>`. Try to inspect its element type with `instanceof`; explain why that check cannot verify each element's generic type.

### Exceptions

Checked exceptions must be caught or declared with `throws`; unchecked exceptions are `RuntimeException` subclasses and do not have that compile-time requirement. This differs from C#, where exceptions are not divided this way. Use exceptions for exceptional failures, not ordinary game outcomes such as an occupied cell. Choose and document a consistent policy for invalid moves: return a result that explains rejection, or throw a deliberate argument/state exception.

**Exercise:** distinguish an invalid position, an occupied position, and a move after the game is over. Test the chosen behavior and make sure a rejected move does not alter the game.

### `var` and text blocks

`var` is local-variable type inference, not dynamic typing. The compiler still assigns a static type, and `var` cannot be used for fields, method parameters, or return types. Use it when the initializer makes the type clear; use an explicit type when that improves readability.

Text blocks make multi-line strings easier to read:

```java
String boardText = """
    X . O . .
    . X . . .
    O . X . .
    . . . . .
    . . . . .
    """;
```

**Exercise:** print a board using a text block, then decide whether the same representation belongs in the game engine or in a user-interface layer.

## 3. Modern Java language and APIs

### Records, enums, and sealed types

Use a record for a small immutable value whose identity is its data. A compact constructor can validate its invariant:

```java
public record Position(int row, int column) {
    public Position {
        if (row < 0 || row >= 5 || column < 0 || column >= 5) {
            throw new IllegalArgumentException("Position is outside the board");
        }
    }
}
```

An enum is a closed set of named values, such as `Player.X` and `Player.O`. A sealed interface restricts which types can implement it; this is useful when a result has a small, known set of alternatives. Records, enums, and sealed types are not interchangeable: choose based on whether you need a value, a fixed set of constants, or a restricted family of variants.

**Exercise:** model move results or game status with a sealed interface and records. Keep mutation and validation in the game object rather than in UI code.

### Pattern matching

Java supports pattern matching for `instanceof` and `switch` in JDK 21. A switch expression returns a value and must cover all alternatives when switching over a sealed hierarchy:

```java
static String describe(GameStatus status) {
    return switch (status) {
        case Playing(var currentPlayer) -> currentPlayer + " to move";
        case FinalMove(var winner, var finalMover) ->
            finalMover + " gets the final move; " + winner + " has the first row";
        case Finished(var winner) ->
            winner.map(player -> player + " wins").orElse("Draw");
    };
}
```

Use pattern matching to make the alternatives explicit, not to replace a simple conditional that is easier to understand.

### `Optional`

`Optional<T>` represents a value that may be absent, for example a winner when a game ends in a draw. Use operations such as `map`, `orElse`, and `ifPresent` to handle absence explicitly. Avoid calling `get()` without first proving a value is present. `Optional` is generally intended for return values; do not use it automatically for every field or method parameter.

**Exercise:** make the winner query return `Optional<Player>` and test both a win and a draw without a null check.

### Streams and `java.time`

Streams transform or aggregate collections; they are Java's closest common counterpart to LINQ. A stream pipeline has a source, intermediate operations such as `filter` or `map`, and a terminal operation such as `toList` or `anyMatch`. Use a loop when it makes stateful game logic clearer; streams are not inherently better or faster.

For example, `moves.stream().filter(Move::isLegal).toList()` creates a list of legal moves. Do not reuse a consumed stream; create a new stream from its source for another traversal.

Use `java.time` types instead of legacy date classes: `Instant` for a point on the timeline, `Duration` for elapsed time, `LocalDate` for a calendar date, and `ZonedDateTime` when a time zone matters.

**Exercise:** use a stream to list legal moves, then identify why detecting a win across the board's rows, columns, and diagonals may be clearer as a dedicated method with loops. Add a `Duration` measurement only as a separate experiment, not as game-rule state.

### Concurrency: `CompletableFuture` and virtual threads

`CompletableFuture` composes asynchronous work and is useful when an operation already has an asynchronous result. Virtual threads (available in JDK 21) make blocking, thread-per-task code more scalable; they are not a replacement for understanding shared mutable state or synchronization.

The Fritz engine is small, deterministic, and synchronous. Do not add concurrency to it. Instead, write a separate short experiment that runs independent tasks on virtual threads, and compare composing asynchronous results with `CompletableFuture` to the familiar C# `Task` model. Be ready to explain why neither belongs in the game-rule implementation.

## 4. Test-first development

Use JUnit 5 (`@Test`, assertions, and parameterized tests where useful) to define behavior. AssertJ provides readable assertions:

```java
@Test
void aNewGameStartsWithAnEmptyBoardAndXToMove() {
    Game game = new Game();

    assertThat(game.currentPlayer()).isEqualTo(Player.X);
    assertThat(game.winner()).isEmpty();
    assertThat(game.board()).hasSize(5);
}
```

Mockito creates test doubles for collaborators. Use it when testing a class that depends on an external collaborator; do not mock the board or rules when testing the game engine itself. Prefer real domain objects for these unit tests.

Write focused tests for one behavior at a time. Include normal cases, boundary cases, and rejected actions. Run `./mvnw test` after each small increment, and keep tests deterministic: they should not require Azure, a network connection, a clock delay, or a running server.

## 5. Build the Fritz game engine

Use the existing [Fritz game rules](https://github.com/thomasneuberger/fritz#game-rules) and the original [`GameService`](https://github.com/thomasneuberger/fritz/blob/main/FritzApp/Services/GameService.cs) as the behavior reference. Port the rules, not the Blazor UI or localization. The game is a 5×5 board; a player wins by placing three of their symbols consecutively horizontally, vertically, or diagonally.

Implement a pure Java model with clear responsibilities. One reasonable starting point is:

- `Player` — the two players (`X` and `O`).
- `Position` — a row and column, validated against the 5×5 board.
- `Move` — the player and position for one action.
- `Board` — cell occupancy, move validation, and win detection.
- `Game` — current player, game status, turn changes, and winner.

Keep the board representation private and expose a safe read-only view. A move must be in bounds, target an empty cell, and be made while the game accepts moves. A rejected move must not partially change state. Start with an understandable implementation; refactor only after tests capture its behavior.

Implement the Fritz rule as an explicit state transition:

1. During normal play, alternate players after each valid move.
2. When a player first makes three in a row, remember that player and give the opponent exactly one final move.
3. If the opponent also makes three in a row on that final move, the opponent wins.
4. Otherwise, the player who made the first row wins.
5. Once the game is over, reject further moves. If the board fills before anyone makes three in a row, the game is a draw.

Check all possible three-cell windows. On a 5×5 board, a window can begin anywhere it fits in a row, column, or either diagonal direction. Checking windows (rather than only the whole row) also handles overlapping rows and a run of more than three symbols.

### Suggested implementation order

1. Create the empty board and represent the current player.
2. Add legal move placement, player switching, and tests for bounds, occupied cells, and turn order.
3. Add horizontal, vertical, and both diagonal win detection; test each direction and edge positions.
4. Add the Fritz final-move state and test both outcomes: the opponent wins by also making a row, or the first player wins when they do not.
5. Add draw detection, game-over behavior, and a reset/new-game operation.
6. Refactor names and types where appropriate, keeping all tests passing.

### Definition of done

- The game engine is a pure Java library with no Spring, UI, database, or network dependency.
- The board is 5×5; legal moves, turn order, and three-in-a-row detection work in all four directions.
- Tests cover invalid and occupied positions, rejected moves leaving state unchanged, both Fritz-rule outcomes, draws, reset, and moves after game over.
- `./mvnw test` passes from `backend/`.
- You can explain how Java packages, interfaces, generics/type erasure, exceptions, records, sealed types, pattern matching, `Optional`, streams, and `java.time` differ from familiar .NET or TypeScript concepts.
- You can describe appropriate uses for Mockito, `CompletableFuture`, and virtual threads—and why the game engine does not need the latter two.
