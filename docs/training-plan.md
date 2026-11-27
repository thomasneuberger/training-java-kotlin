# Training plan: modern Java (backend) and Kotlin (Android frontend)

## Audience and approach

- Experienced developer, mainly .NET (backend/cloud, web/desktop), plus Angular/React/Vue. Solid database knowledge, but no Java/Kotlin/Android experience.
- Learning by building: every module ends with a working increment of the example app **Fritz** (5x5 Tic-Tac-Toe; 3 in a row wins; the *Fritz rule* gives the opponent one last move after the first player reaches 3 in a row; see <https://github.com/thomasneuberger/fritz>).
- Each topic is introduced by mapping it to its .NET / TypeScript counterpart (see the cheat sheet at the end).
- Target: ~12 weeks at ~6–8 h/week. Each module has theory, a hands-on task and a "definition of done".

## Target architecture

| Part | Technology |
|---|---|
| Android app | Kotlin, Gradle (Kotlin DSL), Jetpack Compose, ViewModel/StateFlow, Retrofit/Ktor client, MSAL for Android |
| Backend | Java 21 LTS, Maven, Spring Boot 3.x (REST + WebSocket/STOMP), Spring Data JPA/Hibernate, Flyway |
| Storage | Existing Azure SQL Server instance for application data and profile-image references; Azure Blob Storage for image files |
| Auth | Microsoft Entra ID (OIDC / OAuth2, JWT access tokens validated by the backend) |
| Hosting | Docker image on one or more Azure Container Apps, provisioned with Bicep |
| CI/CD | GitHub Actions (backend image + Android build/Play upload) |

Assumed repository layout: `backend/` (Maven Java project), `android/` (Gradle Kotlin app), `infrastructure/` (Bicep), `docs/`.

## Modules

### Phase 0 – Tooling (week 1)
1. **Editor and setup**: introduce IntelliJ IDEA for Java backend development and Android Studio for Android/Kotlin, including their common workflows and useful shortcuts. Explain that VS Code is familiar and usable, but the JetBrains IDEs are the standard integrated environments for these parts of the stack; compare when VS Code is a reasonable alternative. Set up JDK 21 (Temurin), Git, Docker and Azure CLI.
2. **Recommended plugins and tooling**: configure relevant IDE support such as the Spring Boot and Kotlin plugins, Android tooling, database integration, and code-quality/formatting support (for example, Checkstyle or Spotless where appropriate). Cover the IDE's built-in Maven/Gradle integration and command-line wrappers.
3. **Build tools**: Maven for the backend and Gradle with Kotlin DSL for Android; compare both with MSBuild/NuGet, including dependency management, project layout and running tests.
- *Deliverable*: "Hello World" in Java and Kotlin built from the command line.

### Phase 1 – Modern Java (weeks 1–3)
3. **Java language for C# developers**: packages vs. namespaces, classes, interfaces, generics (type erasure), exceptions (checked vs. unchecked), `var`, text blocks.
4. **Modern features**: records, sealed interfaces, pattern matching (`switch`, `instanceof`), enums, `Optional`, streams (vs. LINQ), `java.time`, virtual threads, `CompletableFuture`.
5. **Testing**: JUnit 5, AssertJ, Mockito (vs. xUnit/Moq).
- *Deliverable*: the **Fritz game engine** as a pure Java library (board, moves, win detection, Fritz rule) with thorough unit tests, ported from the Blazor `GameService`.

### Phase 2 – Backend with Spring Boot (weeks 3–6)
6. **Spring Boot basics**: dependency injection (vs. ASP.NET Core DI), configuration/profiles (vs. `appsettings`), controllers, validation, error handling (ProblemDetails), OpenAPI docs.
7. **Database access**: JDBC, Spring Data JPA/Hibernate (vs. EF Core), connecting to Azure SQL (mssql-jdbc), Flyway migrations, transactions, Testcontainers for integration tests.
8. **Domain & API**: entities `Player` (display name and image reference), `Game`, `Move`; REST API for create/join/list games and submit moves, server-side rule validation using the game engine.
9. **Real-time play**: WebSocket/STOMP (vs. SignalR); consequences when running multiple container instances (shared state in DB, optional Redis / Azure Web PubSub as backplane).
10. **Profile images**: upload endpoint, size/type validation, Azure Blob Storage in a storage account, database references and serving with caching. Keep the container private and authorize access to images.
- *Deliverable*: fully tested backend running locally against Azure SQL (or Testcontainers).

### Phase 3 – Authentication with Entra ID (week 7)
11. **Java/Spring integration**: configure Entra app registrations for the API and Android client using the already-known OIDC/OAuth2, scopes, PKCE and token concepts. Focus on Java configuration and the Spring Security OAuth2 resource-server support.
12. **Securing the API**: configure issuer and audience validation, map the validated `oid` claim to a `Player`, auto-provision on first login, and apply authorization rules (only participants may move). Test protected endpoints with mocked JWTs and a real Entra token.
- *Deliverable*: protected endpoints, tested with real Entra tokens and with mocked JWTs.

### Phase 4 – Kotlin and Android (weeks 8–10)
13. **Kotlin for C# developers**: null safety, data classes, extension functions, sealed classes, `when`, scope functions, coroutines & Flow (vs. async/await, `IAsyncEnumerable`), kotlinx.serialization.
14. **Android fundamentals**: project structure, Activity, manifest, resources, Gradle modules, lifecycle, navigation.
15. **Jetpack Compose**: composables, state hoisting, theming/Material 3, previews (vs. Blazor/React components), board UI for Fritz.
16. **App architecture**: MVVM with ViewModel + StateFlow, repository pattern, Hilt DI, Room for local caching/offline-first, testing (JUnit, Turbine, Compose UI tests).
17. **Networking & auth**: Retrofit/Ktor client, MSAL sign-in, token handling, profile screen (name + image via image picker/Coil), real-time game updates.
- *Deliverable*: a playable app: sign in, edit profile, create/join a game, play against another player.

### Phase 5 – Deployment and operations (weeks 11–12)
18. **Containerize the backend**: Dockerfile (multi-stage, JRE image), health endpoints (Actuator), logging, metrics.
19. **Azure Container Apps**: briefly provision infrastructure with Bicep; cover ingress, scaling rules, secrets & managed identity for SQL and Blob Storage access, Key Vault, revisions and multiple replicas.
20. **Environments**: `dev` / `staging` / `prod` – separate Container Apps (and databases or schemas), config per environment, separate Entra app registrations, GitHub Actions with environment approvals.
21. **Android environments**: Gradle build types and product flavors (`dev`, `staging`, `prod`) with different application IDs, API URLs and Entra client IDs; signing configs and secrets handling.
22. **Google Play release**: create a Play Console developer account (none exists yet), complete registration, payment and required verification; check current account-specific testing and production-access requirements before scheduling release. Cover Play App Signing, Android App Bundle, internal / closed / open testing tracks and production, promotion between tracks (this maps to the staging concept), store listing, privacy policy and data-safety form, versioning, automated upload with GitHub Actions (e.g. Gradle Play Publisher).
23. **Wrap-up**: monitoring (Application Insights/Azure Monitor), crash reporting, retrospective and next steps (Kotlin Multiplatform, Spring Native, etc.).

## Suggested milestones

| Week | Milestone |
|---|---|
| 1 | Tooling set up, Java/Kotlin hello world |
| 3 | Game engine in Java with tests |
| 6 | Backend with DB, REST and real-time API |
| 7 | Entra ID–secured API |
| 10 | Android app playable end-to-end locally |
| 12 | Deployed to Azure Container Apps (staging + prod), app submitted to a Play internal/closed track, subject to account registration and verification |

### Optional later stage – AI opponent

After two-player online play and deployment work, add a computer opponent using the existing server-side game engine. Explore a rule-based or search-based move selection strategy that respects the Fritz final-move rule, add difficulty levels, and test legal moves and game outcomes. This is an extension, not a prerequisite for the core milestones.

## .NET → Java/Kotlin cheat sheet

| .NET / TS | Java / Kotlin |
|---|---|
| NuGet, MSBuild, `.csproj` | Maven Central, Maven (`backend/`), Gradle (`android/`) |
| ASP.NET Core | Spring Boot |
| Entity Framework Core | Spring Data JPA / Hibernate |
| LINQ | Streams API / Kotlin collection functions |
| `async`/`await`, `Task` | `CompletableFuture`, virtual threads / Kotlin coroutines |
| SignalR | WebSocket/STOMP, SSE |
| xUnit, Moq | JUnit 5, Mockito / MockK |
| `appsettings.json` | `application.yml`, profiles |
| Blazor/React components | Jetpack Compose |
| MSAL.NET | MSAL for Java / Android |
