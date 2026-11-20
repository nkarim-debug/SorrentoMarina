# CI/CD Pipeline Report

## Pipeline stages
The workflow `.github/workflows/ci.yml` runs on every `push` and
`pull_request` to `main`/`master`, with no manual step:
1. **Checkout** (`actions/checkout@v4`): fetches the exact commit under test.
2. **Environment setup** (`actions/setup-java@v4`): installs JDK 21
   (Temurin) on a fresh `ubuntu-latest` runner and caches Maven dependencies.
3. **Database setup**: creates a disposable SQLite database from
   `schema.sql`, so tests need no external service.
4. **Automated tests** (`mvn clean test`): runs the full test suite.
   If one test fails, the pipeline stops.
5. **Build** (`mvn package -DskipTests`): produces the WAR file, only
   after the tests have passed.
6. **Artifact upload**: stores the WAR as `sorrentomarina-<commit SHA>`
   for 30 days.

## Build reproducibility
- Every run starts on a fresh virtual machine, so results do not depend
  on a developer's computer ("works on my machine").
- The Java version and distribution are fixed in the workflow file.
- Dependencies are declared in `pom.xml`, and the cache key follows its
  content, so each run uses the same libraries.
- The test database is rebuilt from `schema.sql` at every run, so no
  state leaks between executions.
- The artifact name contains the commit SHA, so each build can be traced
  to the exact source code that produced it.

## Code quality
- Tests run automatically on every change: bugs are detected before
  they reach `main`. The latest run passes 444 tests.
- The build step is reached only if all tests succeed.
- Least privilege: `permissions: contents: read` limits the rights of
  the `GITHUB_TOKEN`.
- No password is stored in the repository; credentials (for example
  Docker Hub) are read from GitHub Repository Secrets.
