## go

> Go standards for K8s projects

# Go
chain:gofmt→vet→golangci-lint→test
doc:≤80ch|pkg-comment req for public
pattern:accept-interface,return-struct|fmt.Errorf("%w",err)|ctx 1st|defer Close()
naming:Export=Pascal|unexport=camel|acronym-consistent(URL/Url)
error:never _=f()|wrap+ctx|sentinel sparingly
concurrency:mutex/chan for shared|goroutine exit strategy|ctx cancel
test:table-driven|t.Parallel() safe|*_test.go
k8s:graceful(SIGTERM/INT)|json-log|probes|no-hardcoded-secrets

## Security Scans (verify phase)
- `govulncheck ./...` — Go vulnerability check
- `gosec ./...` — Go security linter
- `trivy fs .` — filesystem/dependency vulnerability scan
Run all three before claiming implementation is complete.

---
> Source: [ArangoGutierrez/promptsLibrary](https://github.com/ArangoGutierrez/promptsLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-08 -->
