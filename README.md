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



Your objective is NOT to improve the code.

Your objective is to make the current CI failure pass.

Rules:

1. Make the smallest change possible.
2. Do not refactor unrelated code.
3. Do not modify tests unless the test is incompatible with the new API.
4. Do not change dependency versions unless dependency resolution is the failure.
5. Do not suppress compiler warnings/errors.
6. Do not disable tests.
7. Do not weaken static analysis.
8. Do not change CI configuration unless the failure is CI-related.
9. Preserve existing application behavior.
10. If you cannot establish a plausible root cause, return NEEDS_HUMAN_REVIEW.


Yes. The key is to stop thinking of this as a single “AI agent that fixes CI failures” and instead build a controlled repair loop where the LLM is one component inside a deterministic state machine.

For your Kotlin/shared-library upgrade scenario, I would structure it like this:

Upgrade Agent
     │
     ▼
Create branch + apply upgrade
     │
     ▼
Commit + PR
     │
     ▼
Run CI
     │
     ├── GREEN ───────────────► Done
     │
     ▼
Collect failure evidence
     │
     ▼
Classify failure
     │
     ├── Infrastructure ──────► Retry / escalate
     ├── Flaky test ──────────► Retry / classify
     ├── Dependency ──────────► Repair agent
     ├── Compilation ─────────► Repair agent
     ├── Test failure ────────► Repair agent
     ├── Static analysis ─────► Repair agent
     └── Unknown ─────────────► Repair agent / human
                                      │
                                      ▼
                              Generate fix plan
                                      │
                                      ▼
                              Apply minimal patch
                                      │
                                      ▼
                              Local validation
                                      │
                                      ▼
                              Commit + CI
                                      │
                                      └───────► loop

The important part is that the LLM should not control the loop. Your orchestrator should.

1. Make the agent a state machine

Don't give the model a prompt like:

"Look at the CI failure and fix the code."

Instead, your application should maintain something like:

UpgradeJob
 ├── repository
 ├── branch
 ├── baseCommit
 ├── targetLibraryVersion
 ├── currentCommit
 ├── attemptNumber
 ├── ciRuns[]
 ├── changes[]
 ├── failureHistory[]
 └── status

And explicitly transition through states:

UPGRADE
  ↓
VALIDATE
  ↓
CI_RUNNING
  ↓
CI_FAILED
  ↓
ANALYZE
  ↓
PATCHING
  ↓
VALIDATE
  ↓
CI_RUNNING

The agent never decides whether to run CI again. Your orchestrator does.

That alone tends to make these systems dramatically more reliable.

2. Give the agent very constrained tools

Instead of giving it arbitrary shell access, expose tools such as:

get_ci_status()
get_ci_logs()
get_failed_tests()
get_changed_files()
get_git_diff()
search_repository()
read_file()
apply_patch()
run_gradle_task()
run_test()
git_commit()

And preferably:

run_targeted_test(testName)
run_module_build(module)
run_full_build()

The agent should have a small toolbox.

For example:

{
  "tool": "apply_patch",
  "description": "Apply a minimal patch to the repository",
  "constraints": {
    "must_be_related_to_failure": true,
    "max_files": 10
  }
}

This is much safer than:

agent.execute("whatever shell command you want")
3. Separate diagnosis from fixing

This is one of the biggest improvements I'd make.

Don't ask one LLM call to:

Analyze logs and fix the problem.

Use two phases.

Phase A — Diagnosis

Have the model produce structured output:

{
  "category": "COMPILATION",
  "root_cause": "HttpClient API changed in shared-library version 7",
  "confidence": 0.91,
  "affected_files": [
    "src/main/kotlin/foo/Bar.kt"
  ],
  "evidence": [
    "Unresolved reference: HttpRequest",
    "API removed in shared library 7"
  ],
  "proposed_fix": "Replace HttpRequest with NewHttpRequest"
}

Do not allow code modification in this call.

Then your orchestrator can decide whether the diagnosis is actionable.

Phase B — Repair

Give the repair agent:

CI failure
+
diagnosis
+
relevant source files
+
git diff
+
library migration information

and tell it:

Apply the smallest possible change that addresses this failure. Do not refactor unrelated code.

This dramatically reduces random changes.

4. Build a failure taxonomy

Your agent should first determine what kind of failure it is.

For your environment I'd probably start with:

COMPILATION
DEPENDENCY_RESOLUTION
UNIT_TEST
INTEGRATION_TEST
STATIC_ANALYSIS
FORMATTER
API_MIGRATION
KOTLIN_COMPILER
GRADLE
CI_INFRASTRUCTURE
FLAKY_TEST
UNKNOWN

For example:

Gradle cannot download artifact
        ↓
DEPENDENCY_RESOLUTION
        ↓
Don't ask coding agent to modify Kotlin

Or:

Unresolved reference: SomeHttpClient
        ↓
API_MIGRATION
        ↓
Repair agent

Or:

Docker daemon unavailable
        ↓
CI_INFRASTRUCTURE
        ↓
Retry CI

This prevents the LLM from trying to "fix" infrastructure problems by changing application code.

5. Give the agent migration knowledge

This is particularly important for your shared-library situation.

You have something extremely valuable:

You know that many repositories are migrating from version X → Y.

Create a migration knowledge base.

For example:

Shared Library 5 → 6

Netty:
    old API → new API

HTTP:
    OldHttpClient → NewHttpClient

JSON:
    ObjectMapper configuration changed

Jakarta:
    javax.foo → jakarta.foo

Kotlin:
    Some extension removed

Then when the agent sees:

Unresolved reference: Foo

it can search:

migration knowledge:
Foo was removed in shared-library 6
replacement = Bar

rather than trying to infer everything from the CI log.

This will make the agent much more deterministic.

6. Give it the previous attempts

A common mistake is that every repair attempt starts fresh.

Don't do that.

Give the agent a repair history:

Attempt 1
---------
Failure:
Unresolved reference Foo

Diagnosis:
Foo replaced by Bar

Change:
Foo -> Bar

Result:
Compilation succeeded
New failure:
IllegalStateException in test X


Attempt 2
---------
Failure:
IllegalStateException in test X

Diagnosis:
New HTTP client requires explicit timeout

Change:
Added timeout

Result:
CI failed
New failure:
...

This is important because otherwise the agent can repeatedly make the same mistake.

Even better, explicitly tell it:

Do not repeat a previously attempted change unless new evidence indicates that the previous change was incorrect.

7. Use "minimal patch" as a hard requirement

I'd make the repair agent follow something like:

Your objective is NOT to improve the code.

Your objective is to make the current CI failure pass.

Rules:

1. Make the smallest change possible.
2. Do not refactor unrelated code.
3. Do not modify tests unless the test is incompatible with the new API.
4. Do not change dependency versions unless dependency resolution is the failure.
5. Do not suppress compiler warnings/errors.
6. Do not disable tests.
7. Do not weaken static analysis.
8. Do not change CI configuration unless the failure is CI-related.
9. Preserve existing application behavior.
10. If you cannot establish a plausible root cause, return NEEDS_HUMAN_REVIEW.

That last one is particularly important.

Don't force the agent to make a change.

8. Add a verification gate before CI

You don't want:

LLM changes code
      ↓
CI
      ↓
LLM changes code
      ↓
CI

Instead:

LLM changes code
      ↓
git diff
      ↓
compile affected module
      ↓
run affected tests
      ↓
static analysis
      ↓
CI

For example:

./gradlew :module:compileKotlin
./gradlew :module:test --tests FooTest

If the local verification fails, feed that failure back to the repair agent before spending another CI cycle.

CI is expensive and slow. Local Gradle feedback is your fast feedback loop.

9. Have a hard attempt budget

Never create:

while (!green) {
    askLLM()
}

Use:

MAX_ATTEMPTS = 8

And perhaps:

MAX_FILES_CHANGED = 20
MAX_TOTAL_DIFF_LINES = 500
MAX_CI_RUNS = 8
MAX_SAME_FAILURE = 2

If:

attempt 1 → failure A
attempt 2 → failure B
attempt 3 → failure C
attempt 4 → failure C

stop.

If the same failure occurs twice, that's a strong signal that the agent is stuck.

10. Detect "progress"

You can make this much smarter.

Store a normalized representation of every CI failure.

For example:

failure_signature =
    hash(
       error_type +
       exception_class +
       source_file +
       line +
       normalized_message
    )

Then:

Attempt 1: A
Attempt 2: B
Attempt 3: C
Attempt 4: C

means:

No progress
→ stop / change strategy

But:

A → B → C → GREEN

is healthy.

You can even score progress:

Compilation errors: 37 → 12 → 3 → 0
Tests failing:       12 → 4 → 1 → 0

That's a very good signal that the agent is converging.

11. Don't feed the entire CI log to the LLM

This is another likely source of inconsistency.

CI logs can be enormous.

Build a failure extractor.

Instead of:

100,000 lines of CI output

produce:

BUILD FAILURE

Task:
:foo:test

Primary failure:
FooTest.shouldCreateRequest

Exception:
IllegalArgumentException

Message:
Unsupported media type application/json

Relevant stack:
...

Compilation errors:
...

Changed files since previous successful build:
...

Previous attempt:
...

Git diff:
...

Then optionally give the model access to the full log through a get_ci_log_context() tool.

This makes the reasoning much more focused.

12. Use different agents for different jobs

You don't necessarily need one gigantic agent.

I'd use something like:

                 ┌──────────────┐
                 │ Orchestrator │
                 └──────┬───────┘
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
       Classifier    Diagnoser    Validator
            │           │
            └─────┬─────┘
                  ▼
             Repair Agent
                  │
                  ▼
             Code Review

The orchestrator is deterministic.

LLMs do:

classification
reasoning
diagnosis
patch generation

Your application does:

git
CI
state
retries
budgets
validation
branch management
13. Add a "repair reviewer"

This is a very useful second LLM pass.

After the repair agent modifies code:

Repair Agent
     ↓
patch
     ↓
Reviewer Agent

Ask the reviewer:

Given:
- original CI failure
- diagnosis
- proposed patch

Determine whether this patch plausibly fixes the failure.

Return:

APPROVE
REJECT
NEEDS_MORE_CONTEXT

For example:

{
  "decision": "REJECT",
  "reason": "Patch changes JSON serialization globally but the failure is isolated to HTTP request construction.",
  "risk": "HIGH"
}

Then don't even run CI.

14. Give the agent a "stop" action

Your agent should be allowed to say:

{
  "status": "NEEDS_HUMAN",
  "reason": "Failure appears to be a semantic behavior change in the shared library. No migration rule found."
}

This is much better than an agent that always produces a patch.

In production, knowing when not to change code is a major part of reliability.

15. The overall algorithm

I'd implement your system approximately like this:

upgrade repository
      │
      ▼
create PR
      │
      ▼
┌──────────────────────────────┐
│          repair loop         │
│                              │
│  run CI                      │
│    │                         │
│    ├── GREEN → SUCCESS       │
│    │                         │
│    ▼                         │
│  extract failure             │
│    │                         │
│    ▼                         │
│  classify                    │
│    │                         │
│    ├── infra → retry         │
│    ├── flaky → retry         │
│    └── code → continue       │
│                              │
│  diagnose                    │
│    │                         │
│    ▼                         │
│  check migration KB          │
│    │                         │
│    ▼                         │
│  generate minimal patch      │
│    │                         │
│    ▼                         │
│  review patch                │
│    │                         │
│    ├── reject → regenerate   │
│    │                         │
│    ▼                         │
│  local validation            │
│    │                         │
│    ├── fail → repair         │
│    │                         │
│    ▼                         │
│  commit                      │
│    │                         │
│    ▼                         │
│  update PR                   │
│    │                         │
│    ▼                         │
│  repeat                      │
│                              │
│  max attempts → HUMAN        │
└──────────────────────────────┘
The most important architectural change

If I had to pick one thing to change in your current implementation, it would be this:

Don't build an autonomous agent that is responsible for "upgrade until CI passes." Build a deterministic CI-repair orchestrator that invokes an LLM for bounded diagnosis and patch generation.

Think of the LLM as:

a highly capable repair function

rather than:

the system controller

That distinction is what makes these systems much more predictable.

For your particular Kotlin/shared-library migration

You can make this even stronger by collecting successful migrations over time.

Every successful repair can become a training/example record:

Shared library:
    6.2 → 7.0

Failure:
    unresolved reference Foo

Repository context:
    HTTP client usage

Successful patch:
    Foo(...) → Bar(...)

Validation:
    compile + tests + CI green

Confidence:
    HIGH

After you've upgraded 20–50 repositories, you'll have a very useful migration playbook. The agent can first search that playbook and apply known transformations, using LLM reasoning only for cases it hasn't seen before.

That will likely be much more reliable than trying to make the LLM itself increasingly clever.

If you tell me what CI/CD system you're using (GitHub Actions, GitLab, Jenkins, Azure DevOps, etc.) and how your current agent is implemented (LangGraph, OpenAI Agents SDK, custom Kotlin/Python service, etc.), I can sketch a concrete architecture—including the agent state machine, tool interfaces, prompts, failure schema, retry logic, and an example implementation.
