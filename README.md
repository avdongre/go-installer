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
A single prompt is usually not enough for a project of this size. If you ask Claude to "build the whole thing," you'll likely get an over-engineered architecture that doesn't match your requirements.

A better approach is to give Claude the role of a **Principal AI Systems Architect** and ask it to produce the architecture, interfaces, implementation plan, and then generate the code incrementally.

Here's a prompt I would use.

---

# Prompt

```text
You are a Principal Staff Software Engineer and AI Architect with expertise in:

- Distributed systems
- Kubernetes (EKS)
- AWS
- Agentic AI systems
- MCP (Model Context Protocol)
- LLM orchestration
- Event driven architectures
- Production observability
- LangGraph
- OpenTelemetry
- Datadog
- Coralogix
- Kafka
- Aurora PostgreSQL
- CockroachDB
- Python
- FastAPI

Your task is to design and implement a production-grade AI Incident Investigation Platform.

The platform should help engineers investigate customer tickets automatically by collecting evidence from multiple systems, correlating the evidence, generating hypotheses, validating those hypotheses, and producing a root cause analysis with confidence scores.

This system is NOT a chatbot.

It is an AI-powered investigation engine.

======================================================
Problem
======================================================

Today when a customer reports an issue, engineers manually perform all of these steps:

• Understand the customer issue
• Determine affected services
• Open Datadog
• Search Coralogix logs
• Look at Kubernetes
• Check pod restarts
• Check deployments
• Check Kafka
• Check Aurora
• Check CockroachDB
• Search Github commits
• Search Confluence
• Read previous incidents
• Correlate everything mentally

This may take 30-90 minutes.

The goal is to reduce this to less than 5 minutes.

======================================================
Environment
======================================================

Application runs on AWS.

Infrastructure:

- Amazon EKS
- Multiple microservices
- Variable replica counts
- Aurora PostgreSQL
- Kafka
- CockroachDB

Observability

- Datadog
- Coralogix

Knowledge

- Github
- Confluence
- Historical tickets
- Runbooks
- Postmortems

Everything should be accessed using MCP servers wherever possible.

======================================================
High Level Architecture
======================================================

The architecture should consist of:

1. Ticket Understanding Agent

2. Dependency Discovery Agent

3. Investigation Planner

4. Evidence Collectors

5. Correlation Engine

6. Hypothesis Generator

7. Hypothesis Validator

8. RCA Generator

9. Report Generator

10. Learning Engine

Each component should have clear responsibilities.

======================================================
Agent Responsibilities
======================================================

Ticket Understanding Agent

Input:

Customer ticket

Output:

- time window
- affected customer
- keywords
- suspected services
- error messages
- priority

----------------------------------------------------

Dependency Discovery Agent

Discovers

- upstream services
- downstream services
- Kafka topics
- databases
- queues
- APIs
- dependencies

Produces an application dependency graph.

----------------------------------------------------

Investigation Planner

Based on the ticket

Determine

Which systems need investigation.

Example

Payment issue

↓

Need

Datadog

Coralogix

Payment Service

Order Service

Kafka

Aurora

Github

Not every incident needs every data source.

The planner should minimize unnecessary work.

======================================================
Evidence Collection Agents
======================================================

Every evidence collector should ONLY gather facts.

Never infer root cause.

Datadog Agent

Collect

CPU

Memory

Latency

Errors

APM traces

Deployments

Container restarts

Network

Service health

Coralogix Agent

Collect

Logs

Exceptions

Stack traces

Correlation IDs

Log spikes

Error frequencies

Kubernetes Agent

Collect

Pod restarts

OOMKilled

CrashLoopBackoff

Node pressure

Scaling events

Rolling deployments

Evictions

Kafka Agent

Collect

Consumer lag

Broker health

ISR

Producer retries

Consumer group state

Topic health

Aurora Agent

Collect

Slow queries

Deadlocks

CPU

Memory

Connections

Replication

Failovers

Cockroach Agent

Collect

Range status

Leaseholder movement

Hot ranges

Retries

Unavailable ranges

Github Agent

Collect

Deployments

Commits

PRs

Merged changes

Feature flags

Configuration changes

Confluence Agent

Retrieve

Runbooks

Architecture

Previous incidents

Known issues

Operational procedures

Historical Incident Agent

Retrieve

Resolved tickets

Root causes

Solutions

Timelines

Lessons learned

======================================================
Evidence Model
======================================================

All evidence should be normalized.

Example

{
    "timestamp":"",
    "source":"",
    "service":"",
    "severity":"",
    "type":"",
    "summary":"",
    "details":"",
    "confidence":0.92,
    "metadata":{}
}

======================================================
Timeline Builder
======================================================

Merge all evidence chronologically.

Example

10:01 Deployment

10:03 Pod restart

10:04 Kafka lag

10:05 DB connections exhausted

10:06 Payment timeout

10:07 Customer ticket

======================================================
Hypothesis Generator
======================================================

Generate

Top 5 hypotheses.

Every hypothesis must include

Description

Supporting evidence

Contradicting evidence

Confidence

Missing evidence

Next steps

Never invent evidence.

Only use collected evidence.

======================================================
Validation Agent
======================================================

Each hypothesis should be challenged.

Attempt to disprove it.

Reduce hallucinations.

Example

Hypothesis

Aurora issue

Validation

CPU normal

Connections normal

No failover

↓

Reject

======================================================
Confidence Scoring
======================================================

Confidence should consider

Number of evidence sources

Historical similarity

Metric anomalies

Error frequency

Deployment timing

Correlation IDs

Known incidents

======================================================
Learning Engine
======================================================

After every resolved incident

Store

Symptoms

Timeline

Root Cause

Fix

Verification

Postmortem

Tags

Affected services

Databases

Deployments

Incident embeddings

======================================================
Technology
======================================================

Backend

Python

FastAPI

LangGraph

Pydantic

OpenAI SDK

Anthropic SDK

AsyncIO

Frontend

React

TypeScript

Tailwind

Storage

PostgreSQL

pgvector

Redis

Graph

NetworkX

Neo4j optional

Knowledge

Vector search

Embeddings

RAG

======================================================
Architecture Requirements
======================================================

Everything must be:

Async

Strongly typed

Modular

Testable

Production ready

Dependency injected

Config driven

Dockerized

Observable

Structured logging

Tracing

Retry logic

Circuit breakers

Caching

======================================================
Project Structure
======================================================

Design a complete folder structure.

Include

agents/

models/

services/

repositories/

mcp/

prompts/

api/

workers/

tests/

config/

knowledge/

frontend/

======================================================
Implementation Plan
======================================================

Do NOT write all code immediately.

Instead produce

Phase 1

Architecture

Phase 2

Models

Phase 3

MCP interfaces

Phase 4

Evidence collectors

Phase 5

Planner

Phase 6

Correlation

Phase 7

Hypothesis engine

Phase 8

Validation

Phase 9

Learning engine

Phase 10

UI

======================================================
Code Generation Rules
======================================================

Generate production-quality code only.

Avoid toy examples.

Every class must have

Docstrings

Logging

Error handling

Type hints

Unit tests

Dependency injection

Use SOLID principles.

Prefer composition over inheritance.

======================================================
Deliverables
======================================================

I want this project built incrementally.

Start by producing:

1. Complete software architecture
2. Sequence diagrams
3. Component diagrams
4. Data model
5. MCP interface definitions
6. Database schema
7. LangGraph workflow
8. Repository structure
9. Detailed implementation roadmap

Do NOT generate implementation code until the architecture is finalized.

After architecture approval, implement one module at a time in production quality.
```

---

## One improvement I would make

I would **not** implement this as "10 independent AI agents talking to each other." That's the pattern many demos use, but it often becomes expensive, slow, and difficult to debug.

Instead, I would structure it as a **stateful investigation graph** using LangGraph:

* A single orchestrator owns the investigation state.
* Specialized investigator nodes (Datadog, Coralogix, Kubernetes, Kafka, Aurora, GitHub, Confluence, etc.) only collect normalized evidence.
* A correlation node builds a unified timeline and links related events using correlation IDs, trace IDs, deployment IDs, and service dependencies.
* A reasoning node generates hypotheses from the collected evidence.
* A validation node attempts to falsify each hypothesis before assigning a confidence score.
* A report node produces the final RCA and recommended next actions.
* A learning node stores the resolved incident, timeline, evidence, and remediation into your knowledge base for future retrieval.

This graph-based approach is easier to observe, retry, cache, and extend than a network of autonomous agents, while still giving you the benefits of specialized reasoning where it adds the most value. In practice, it is also a much better fit for production incident investigation systems.

```
