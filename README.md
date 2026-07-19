# go-installer
Generate Installer for your application using GoLang

## Purpose
Most of the GoLang based applications are self sufficient single executable. Sometime one may need to ship some static
files or images or platform dependent DLLs or Shared library in case of CGO based applications.

This project helps to create that installer only by configuring the single `config.json` file.
This project is internally using [binclude](https://github.com/lu4p/binclude)

## Usage
* Clone this repository
* Create your own `config.json` See example below
* Create your own `file.list` See example below
* Build using command `make build`
* Above command will product executable `out/installer`
* Make sure you have `PATH` contains `$GOPATH/bin`

## Example usage
```bash
$ out/installer --help
Usage of out/installer:
  -install-dir string
    	Install directory
  -mode string
    	Pass to mode for action [install|uninstall|status|start|stop]

$ ./out/installer -mode install -install-dir /home/adongre/1.0
```

## Typical `config.json`
```json
{
	"Components": [{
			"Name": "MyProduct",
			"TargetDir": "bin",
			"Files": [{
					"Name": "file1.txt",
					"Platform": "linux",
					"Action": {
						"command": ""
					},
					"SourceRelativePath": "./stage/file1.txt"
				},
				{
					"Name": "file2.txt",
					"Platform": "linux",
					"Action": {
						"command": ""
					},
					"SourceRelativePath": "./stage/file2.txt"
				}
			]
		},
		{
			"Name": "MyProductLib",
			"TargetDir": "lib",
			"Files": [{
					"Name": "file3.txt",
					"Platform": "linux",
					"Action": {
						"command": ""
					},
					"SourceRelativePath": "./stage/file3.txt"
				},
				{
					"Name": "file4.txt",
					"Platform": "linux",
					"Action": {
						"command": ""
					},
					"SourceRelativePath": "./stage/file4.txt"
				},
				{
					"Name": "file5.txt",
					"Platform": "linux",
					"Action": {
						"command": ""
					},
					"SourceRelativePath": "./stage/file5.txt"
				}
			]
		},
		{
			"Name": "MyProductExtra",
			"TargetDir": "extra",
			"Files": [{
				"Name": "allfiles.tar",
				"Platform": "linux",
				"Action": {
					"command": ""
				},
				"SourceRelativePath": "./stage/allfiles.tar"
			}]
		}
	]
}
```
## Typical `file.list`
```bash
./stage/file1.txt
./stage/file2.txt
./stage/file3.txt
./stage/file4.txt
./stage/file5.txt
./stage/allfiles.tar
```


```
# Design a Production-Grade AI-Powered Developer Security Workflow Using MCP, Claude Code, Snyk, and GitHub

## Role

You are a senior platform engineer designing an enterprise internal developer platform capability.

Design an AI-powered security workflow that integrates **Claude Code**, **Model Context Protocol (MCP)**, **Snyk**, and **GitHub** to perform intelligent security analysis and automated remediation **before Pull Requests are created**.

The system should be designed like a capability built by large engineering organizations.

The goal:

Move security remediation from:

```
Developer creates PR
        |
        v
CI fails
        |
        v
Developer fixes vulnerabilities
```

to:

```
Developer requests PR
        |
        v
AI security analysis
        |
        v
Safe remediation
        |
        v
Validation
        |
        v
Create PR
```

---

# Business Problem

Current developer workflow:

1. Developer creates a feature branch.
2. Developer implements changes.
3. Developer opens a Pull Request.
4. CI runs security scans using Snyk.
5. Builds fail because of dependency vulnerabilities.
6. Developers fix unrelated dependency issues after PR creation.
7. Security teams accumulate vulnerability debt.

Problems:

* Security feedback arrives too late.
* Developers context-switch after PR creation.
* Existing vulnerabilities block unrelated changes.
* Dependency upgrades are performed manually.
* Security teams lack automated remediation workflows.

---

# Target Developer Experience

Developer:

```
Create PR
```

Claude Code workflow:

```
Understand developer intent

        |

Invoke Security MCP Server

        |

Analyze vulnerabilities

        |

Determine if vulnerabilities are related
to current changes

        |

Apply safe fixes when allowed

        |

Run validation

        |

Create commits

        |

Create GitHub PR
```

Generated PR contains:

* Feature implementation
* Optional security remediation commit
* Security analysis summary
* Validation results

---

# Developer Security Intent Support

The system must understand natural language security preferences.

Examples:

## Default

Developer:

```
Create PR
```

Behavior:

```
Security scan: YES
Safe dependency fixes: YES
Major upgrades: Ask developer
```

---

## Scan only mode

Developer:

```
Create PR but no Snyk fixes
```

Interpretation:

```json
{
  "action": "create_pr",
  "security_mode": "scan_only",
  "allow_dependency_changes": false
}
```

Behavior:

```
Run Snyk scan
Generate security report
Do not modify dependency files
Do not create security commits
Create PR with findings
```

---

## Full remediation mode

Developer:

```
Create PR and fix security issues
```

Behavior:

```
Scan
Analyze
Apply safe fixes
Validate
Commit fixes
Create PR
```

---

## Security override rules

Developer preferences control remediation behavior but cannot bypass mandatory security policies.

Example:

Developer:

```
Create PR but no Snyk fixes
```

Finding:

```
New critical vulnerability introduced by this branch
```

Result:

```
BLOCK PR
```

Reason:

```
Developer disabled remediation,
but organization policy requires blocking new critical vulnerabilities.
```

---

# Architecture Overview

Design the following architecture:

```
Developer

   |
   |

Claude Code

   |
   | MCP Protocol

   |

Security MCP Server

   |
   +-----------------------+
   |                       |
   v                       v

Snyk API              AI Reasoning Engine

   |                       |

   v                       v

Dependency Graph     Compatibility Analysis

   |
   v

Fix Recommendation Engine

   |
   v

GitHub Integration

   |
   v

Pull Request
```

---

# Component 1: Claude Code

Responsibilities:

* Understand developer commands.
* Extract developer intent.
* Select security execution mode.
* Invoke MCP tools.
* Modify code.
* Update dependencies.
* Execute commands.
* Run tests.
* Create commits.
* Create GitHub Pull Requests.

Example:

User:

```
Create PR but no Snyk fixes
```

Claude extracts:

```json
{
  "intent": "create_pull_request",
  "security_mode": "scan_only"
}
```

---

# Component 2: Security MCP Server

The MCP server exposes security automation tools.

---

# MCP Tool: security.scan_before_pr()

Purpose:

Analyze repository security before PR creation.

Input:

```json
{
  "repository": "payment-service",
  "branch": "feature/payment-validation",
  "changed_files": [
    "src/payment/PaymentValidator.kt"
  ],
  "dependency_files": [
    "build.gradle.kts",
    "package.json",
    "requirements.txt"
  ],
  "security_mode": "auto_fix"
}
```

Output:

```json
{
  "vulnerabilities_found": true,
  "findings": [],
  "severity": "HIGH",
  "recommended_actions": []
}
```

---

# MCP Tool: security.analyze_fix()

Purpose:

Use AI reasoning to determine if a dependency upgrade is safe.

Input:

```json
{
  "dependency": "spring-boot",
  "current_version": "3.1.2",
  "candidate_version": "3.3.0",
  "vulnerability": "CVE-XXXX"
}
```

Output:

```json
{
  "recommended_version": "3.2.9",
  "risk_level": "MEDIUM",
  "breaking_changes": false,
  "auto_fix_allowed": true,
  "reason": "Compatible upgrade"
}
```

---

# MCP Tool: security.apply_fix()

Responsibilities:

* Update dependency files.
* Update lock files.
* Resolve dependency conflicts.
* Preserve formatting.
* Return modified files.

Supported:

```
build.gradle.kts
pom.xml
package.json
package-lock.json
requirements.txt
pyproject.toml
go.mod
```

---

# MCP Tool: security.validate_fix()

Execute:

## Kotlin

```
./gradlew test
```

## Java

```
mvn test
```

## Python

```
pytest
```

## UI

```
npm test
npm run build
```

Validate:

* Build success
* Unit tests
* Integration tests
* Dependency resolution
* Regression impact

---

# MCP Tool: security.generate_summary()

Generate GitHub PR markdown.

Example:

```markdown
## Security Analysis

Mode:
Automatic remediation

Changes:

- Upgraded jackson-databind
- Fixed CVE-XXXX

Validation:

:white_check_mark:
Gradle tests passed

:white_check_mark:
Integration tests passed
```

---

# Snyk Integration

Integrate using Snyk API.

Retrieve:

* vulnerabilities
* CVEs
* severity
* dependency paths
* fixed versions
* project metadata
* dependency graph

---

# Vulnerability Classification

## Category A

New vulnerability introduced by current change.

Action:

```
BLOCK PR
```

Example:

Developer adds vulnerable dependency.

---

## Category B

Existing vulnerability with safe upgrade.

Action:

```
Create security commit
```

Example:

```
chore(security):

Upgrade jackson-databind
2.14.1 -> 2.17.2
```

---

## Category C

Requires major upgrade.

Action:

```
Warn developer
```

Example:

```
Spring Boot 3.1 -> 3.3

Requires Spring Cloud upgrade.

Risk: HIGH
```

---

## Category D

No fix available.

Action:

```
Report only
```

---

# Dependency Intelligence

The system must understand:

* direct dependencies
* transitive dependencies
* dependency graphs
* semantic versioning
* breaking changes
* framework compatibility
* lock file updates

---

# Supported Technology Ecosystems

## Kotlin / JVM

Support:

Files:

```
build.gradle.kts
settings.gradle.kts
gradle.lockfile
```

Understand:

* Kotlin version compatibility
* JVM compatibility
* Spring Boot
* Spring Cloud
* Ktor
* Coroutines
* Android Gradle Plugin

---

## Java

Support:

```
pom.xml
build.gradle
```

Understand:

* Maven dependency tree
* Gradle dependency resolution
* BOM dependencies

---

## Python

Support:

```
requirements.txt
pyproject.toml
poetry.lock
Pipfile
```

Understand:

* pip
* Poetry
* virtual environments
* Python version compatibility

Frameworks:

* Django
* FastAPI
* Flask

---

## UI Applications

Support:

Frameworks:

* React
* Angular
* Vue
* Next.js

Files:

```
package.json
package-lock.json
yarn.lock
pnpm-lock.yaml
```

Understand:

* npm dependency graph
* frontend vulnerabilities
* bundled libraries
* build tooling
* supply chain risks

Examples:

```
react
next
axios
lodash
webpack
vite
```

---

# AI Reasoning Layer

Use LLM reasoning for:

* dependency impact analysis
* compatibility evaluation
* upgrade safety
* changed-code relevance
* remediation decisions

Example:

Finding:

```
Upgrade Spring Boot 3.1.2 -> 3.3.0
```

Repository:

```
Spring Cloud 2022.0
```

AI reasoning:

```
Risk: HIGH

Spring Boot 3.3 requires Spring Cloud 2023.

Automatic upgrade not allowed.

Developer approval required.
```

---

# GitHub Integration

Support:

* Branch creation
* Commit creation
* Pull Request creation
* PR comments
* Status checks

Use:

Preferred:

```
GitHub App
```

Avoid:

```
Personal Access Tokens
```

---

Example commits:

Commit 1:

```
feat:
Add payment validation
```

Commit 2:

```
chore(security):
Upgrade jackson-databind
```

---

# Credentials and Security

Required credentials:

## Snyk

```
SNYK_TOKEN
```

Used for:

* vulnerability scanning
* dependency analysis

---

## GitHub

Recommended:

GitHub App:

```
GITHUB_APP_ID
GITHUB_PRIVATE_KEY
GITHUB_INSTALLATION_ID
```

Permissions:

* Contents read/write
* Pull Requests read/write
* Checks read/write

---

## AI Provider

Example:

```
ANTHROPIC_API_KEY
```

Used for:

* security reasoning
* compatibility analysis

---

## Private Registries

Support:

Maven:

```
MAVEN_TOKEN
```

npm:

```
NPM_TOKEN
```

Python:

```
PYPI_TOKEN
```

---

# Deployment Architecture

## Option A: Local MCP Server

```
Developer Laptop

Claude Code

Security MCP Server

Snyk API
```

Pros:

* Simple
* Low latency

Cons:

* Limited governance

---

## Option B: Enterprise MCP Platform

Recommended:

```
Developer

Claude Code

MCP Gateway

Security Platform Service

+----------------+
|                |
Snyk          GitHub
|
Jira
|
Artifact Registry
```

Benefits:

* Central policy control
* Auditing
* Metrics
* Enterprise governance

---

# Security Controls

Implement:

## Secrets

Use:

* Vault
* AWS Secrets Manager
* Azure Key Vault

---

## Identity

Use:

* OIDC
* SSO
* Short-lived credentials

---

## Audit Logging

Capture:

* Developer intent
* AI decisions
* Dependency changes
* Approval history
* Security findings

---

## Supply Chain Protection

Detect:

* malicious packages
* dependency confusion
* typosquatting
* untrusted registries

---

# Implementation Skeleton

Recommended stack:

Backend:

```
Kotlin Spring Boot
or
Python FastAPI
```

MCP:

```
Model Context Protocol SDK
```

Storage:

```
PostgreSQL
Redis
```

Queue:

```
Kafka / SQS
```

Integrations:

```
Snyk API
GitHub API
Jira API
```

AI:

```
Claude API
```

---

# Rollout Plan

## Phase 1

Capabilities:

* Security scanning
* Reporting
* Kotlin/Python/UI support

---

## Phase 2

Capabilities:

* Safe automatic remediation
* Security commits
* Validation automation

---

## Phase 3

Enterprise:

* Policy engine
* Dashboards
* Compliance reporting
* Organization-wide adoption

---

# Success Metrics

Measure:

## Developer Productivity

* Reduction in failed PR builds
* Developer hours saved
* Faster PR completion

## Security

* Vulnerabilities fixed before PR
* Mean time to remediation
* Dependency debt reduction

## Platform Adoption

* Repositories onboarded
* PRs using security automation
* Automatic fix success rate

---

Design this as a production-grade internal developer platform capability that combines AI agents, MCP tooling, security automation, and developer experience optimization.

```
