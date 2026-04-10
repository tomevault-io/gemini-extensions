## certgen

> A Go-based web application for generating self-signed X.509 certificates (CA, server, and client certificates). It provides both a web UI and an MCP (Model Context Protocol) server interface.

# CertGen - Certificate Generator Service

## Overview
A Go-based web application for generating self-signed X.509 certificates (CA, server, and client certificates). It provides both a web UI and an MCP (Model Context Protocol) server interface.

## Technology Stack
- **Language**: Go 1.23
- **Web Framework**: Standard library `net/http`
- **Cryptography**: ECDSA P-384 curve for key generation
- **External Dependencies**: 
  - `github.com/mark3labs/mcp-go` - MCP server integration
- **Frontend**: Single HTML template + Pico.css (classless CSS framework)
- **Deployment**: Multi-stage Docker build with Alpine runtime

## Project Structure
```
certgen/
├── main.go                    # Entry point, flag parsing, server startup
├── go.mod / go.sum            # Go module dependencies
├── Dockerfile                 # Multi-stage build (golang:1.25 → alpine)
├── assets/
│   ├── assets.go              # embed.FS for templates and static files
│   ├── templates/index.html   # Web UI template
│   └── static/pico.css        # CSS framework
└── internal/
    ├── server/server.go       # HTTP server, routing, handlers
    ├── certificate/generator.go  # Core cert generation logic
    ├── mcp/mcp.go             # MCP server with 3 tools
    └── random/generator.go    # Random test data generation
```

## Architecture Components

### 1. HTTP Server (`internal/server`)
- Serves web UI at `/`
- REST endpoints:
  - `POST /generate/ca` - Generate CA certificate (JSON body → ZIP download)
  - `POST /generate/cert` - Generate server/client cert (multipart form with CA files → ZIP)
  - `GET /gen/random/{ca,server,client}` - Random form data generators
- Static files served from embedded FS at `/static/`

### 2. Certificate Generator (`internal/certificate`)
- `GenerateCA(CAConfig)` - Self-signed CA with ECDSA P-384
- `GenerateCert(CertConfig, caCert, caKey)` - Server or client cert signed by CA
- `CertBundle` struct with helper methods:
  - `UnifiedPEM()` - cert + key
  - `ChainPEM()` - cert + CA cert
  - `FullChainPEM()` - cert + CA cert + key
- Output formats: `.crt`, `.key`, `.pem`, `-chain.pem`, `-fullchain.pem`

### 3. MCP Server (`internal/mcp`)
- **Library**: `github.com/mark3labs/mcp-go v0.43.1` (mark3labs MCP Go SDK)
- Uses `server.NewMCPServer()` to create the server instance
- Exposed at `/mcp` endpoint via `mcpServer.NewStreamableHTTPServer()`
- Three registered tools:
  - `generate_ca` - CA generation
  - `generate_server_certificate` - Server cert with DNS/IP SANs
  - `generate_client_certificate` - Client auth cert
- Tool responses formatted as markdown with PEM blocks

### 4. Random Generator (`internal/random`)
- Generates sample organization data with matching country/locality pairs
- Supports UK/DE/US/FR/NL locales with appropriate suffixes (Ltd., GmbH, Inc., etc.)

## Key Technical Details
- **Key Algorithm**: ECDSA with P-384 curve
- **Serial Numbers**: Cryptographically random 128-bit integers
- **CA Constraints**: `IsCA=true`, `MaxPathLen=1`
- **Server Certs**: `ExtKeyUsageServerAuth` + DNS/IP SANs
- **Client Certs**: `ExtKeyUsageClientAuth` only
- **Port**: Default `:80`, configurable via `-addr` flag or `PORT` env var

## Data Flow
1. User submits form via web UI or calls MCP tool
2. Server parses request and builds config struct
3. Certificate generator creates ECDSA keypair and X.509 cert
4. PEM-encoded output bundled into ZIP (web) or markdown (MCP)
5. Response returned to client

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pvormste)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pvormste)
<!-- tomevault:4.0:gemini_md:2026-04-08 -->
