---
name: bootstrap-go
description: Use when scaffolding a Go project — invoked as a Wave-0 bootstrap node in plan-as-folder when a spec declares Go or go.mod is detected
---

# Bootstrap Go

Create `Makefile`, `.golangci.yml`, and `go.mod` for a Go project.

## When to Use

- A spec's **Technology** or **Contracts** section declares Go
- `go.mod` exists at the project root (re-bootstrap)
- Referenced by `bootstrap-prompt.md` during bootstrap-node dispatch

Run this as a `bootstrap` kind node. The agent running this step must produce exactly the three files below.

## Files Created

All at the project root:

### `Makefile`

```makefile
BINDIR ?= out

.PHONY: build test lint fmt clean coverage all

build:
	@mkdir -p $(BINDIR)
	for dir in $$(go list -f '{{if eq .Name "main"}}{{.Dir}}{{end}}' ./...); do \
		go build -o $(BINDIR)/$$(basename $$dir) $$dir; \
	done

test:
	go test -race -cover ./...

lint:
	golangci-lint run ./...

fmt:
	gofmt -s -w .

clean:
	rm -f coverage.out
	rm -rf $(BINDIR)

coverage:
	go test -coverprofile=coverage.out ./... && go tool cover -func=coverage.out

all: fmt lint build test
```

### `.golangci.yml`

```yaml
version: "2"

linters:
  enable:
    - govet
    - errcheck
    - staticcheck
    - revive
    - ineffassign
    - unused
    - misspell
  settings:
    govet:
      enable-all: true
    staticcheck:
      checks: ["all"]
    revive:
      rules:
        - name: exported
          severity: warning

run:
  timeout: 5m
```

### `go.mod`

Check if `go.mod` exists. If it does: no-op. If not:

1. Derive module name from the project root directory name: `$(basename $(pwd))`
2. Run `go mod init <module-name>`
3. **Note:** the bare directory name is a placeholder. Replace `<module-name>` with your actual module path (e.g., `github.com/org/project`) before the first push.
4. Run `go mod tidy`

## Usage

This skill is referenced by `bootstrap-prompt.md` during bootstrap-node dispatch. The step file's **Contract** section specifies exact targets and any deviations from the templates above.
