# Language Specialists Plugin

A plugin that bundles commonly used language-specific specialist agents.

## Included Agents

### 1. nextjs-developer
Expert in Next.js 14+ App Router, Server Components, and performance optimization.

**Key Features:**
- Full support for Next.js 14+ App Router
- Optimization of Server Components and Client Components
- Performance tuning (targeting Core Web Vitals > 90)
- SEO optimization (targeting score > 95)
- Edge Runtime utilization
- Full-stack feature implementation

**Use Cases:**
- Building Next.js applications
- Performance optimization
- Implementing Server Actions
- SEO improvements
- Deploying to Vercel

### 2. golang-pro
Expert in Go 1.21+, concurrent programming, and microservices.

**Key Features:**
- Idiomatic Go code implementation
- High-performance concurrent systems
- Cloud-native microservices
- gRPC/REST API implementation
- Table-driven test creation
- Benchmarking and profiling

**Use Cases:**
- Go microservices implementation
- CLI tool development
- High-performance system programming
- Concurrency optimization
- Kubernetes Operator implementation

## Installation

```bash
/plugin install language-specialists
```

Or install from local path:

```bash
/plugin install /path/to/watarus-claude-plugins/language-specialists
```

## Usage

Once the plugin is installed, both agents become available.

### Launching Agents

Claude Code automatically launches the appropriate agent at the right time, but you can also launch them manually:

```bash
/agents
```

Then select from the agent list.

### Automatic Triggering

Agents are automatically triggered in the following scenarios:

**nextjs-developer:**
- Working on Next.js projects
- Performance optimization requests
- SEO improvement requests
- App Router implementation

**golang-pro:**
- Working on Go projects
- Implementing concurrent processing
- Building microservices
- gRPC/REST API implementation

## Requirements

- Claude Code v1.0.0 or higher

## License

The agents included in this plugin follow their original licenses.

### Agent Licenses

All agents included in this plugin are sourced from the following repository and provided under the MIT License:

**Original Repository:** [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)

**Copyright:** © 2024 Awesome Claude Subagents Contributors

**License:** MIT License

Included agents:
- [nextjs-developer](https://github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/02-language-specialists/nextjs-developer.md)
- [golang-pro](https://github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/02-language-specialists/golang-pro.md)

See the full MIT License text [here](https://github.com/VoltAgent/awesome-claude-code-subagents/blob/main/LICENSE).

### This Plugin's License

This plugin itself (structure and README) is also provided under the MIT License.
