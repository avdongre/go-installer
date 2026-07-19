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
You are a senior platform engineer designing an AI-powered developer security workflow.

Goal:
Build a Security MCP Server that integrates with Claude Code to perform pre-PR security checks and automated remediation before a GitHub Pull Request is created.

Problem:
Currently developers create PRs, CI runs Snyk scans, and builds fail because of dependency vulnerabilities unrelated to their code changes. Developers waste time fixing existing dependency issues after the PR is already created.

We want to move security remediation earlier in the workflow.

Target developer experience:

Developer workflow:

1. Developer works on a feature branch.
2. Developer asks Claude Code:
   "Create a PR"
3. Before creating the PR, Claude invokes a Security MCP Server.
4. Security MCP Server:
   - analyzes dependency vulnerabilities
   - checks whether issues are introduced by this change
   - identifies safe upgrade versions
   - applies automatic fixes where appropriate
   - validates the build/tests
   - returns a security summary
5. Claude creates the GitHub PR with:
   - feature changes
   - optional security remediation commit
   - security summary


Architecture requirements:

Design the system with these components:

1. Claude Code
Responsibilities:
- Understand developer intent
- Call MCP tools
- Modify code
- Run commands
- Create commits
- Create PRs


2. Security MCP Server

The MCP server should expose tools such as:

security.scan_before_pr()

Input:
{
  repository,
  branch,
  changed_files,
  dependency_files
}

Output:
{
  vulnerabilities_found,
  severity,
  affected_dependencies,
  recommended_actions
}


security.analyze_fix()

Input:
{
  dependency,
  current_version,
  vulnerability,
  candidate_versions
}

Output:
{
  recommended_version,
  risk_level,
  compatibility_analysis,
  auto_fix_allowed
}


security.apply_fix()

Input:
{
  dependency_changes
}

Action:
- modify dependency files
- update lock files
- return changed files


security.validate_fix()

Action:
- run build
- run unit tests
- check regression


security.generate_summary()

Output:
Markdown summary for GitHub PR.


3. Snyk Integration

Integrate with Snyk using API.

The system should retrieve:

- vulnerabilities
- severity
- CVE information
- fixed versions
- dependency paths
- project information


The system should classify findings:

Category A:
New vulnerability introduced by current change

Action:
BLOCK PR


Category B:
Existing vulnerability unrelated to change but has safe automatic fix

Action:
Create security commit


Category C:
Existing vulnerability requiring major upgrade

Action:
Warn developer


Category D:
No fix available

Action:
Report only


4. Dependency intelligence

The system should support:

Java:
- Maven pom.xml
- Gradle build files

JavaScript:
- package.json
- package-lock.json

Python:
- requirements.txt
- pyproject.toml

Go:
- go.mod


The agent should understand:

- direct dependency vs transitive dependency
- semantic versioning
- breaking changes
- dependency conflicts
- lock file updates


5. GitHub Integration

The system should support:

- branch creation
- commit creation
- PR creation
- PR comments


Security fixes should be committed separately:

Example:

Commit 1:
feat:
Add payment validation


Commit 2:
chore(security):
Upgrade jackson-databind from 2.14.1 to 2.17.2


PR description:

Feature:
Add payment validation

Security changes:
- Upgraded jackson-databind
- Fixed CVE-XXXX

Validation:
- Unit tests passed
- Integration tests passed


6. AI reasoning layer

The system should use an LLM to reason about:

- whether dependency changes are safe
- whether upgrades are compatible
- whether the vulnerability affects the changed code
- whether to automatically fix or ask the developer


Example:

Snyk finding:

Upgrade Spring Boot 3.1.2 to 3.3.0


Agent analysis:

Current project:
Spring Cloud 2022.0

Risk:
HIGH

Reason:
Spring Boot 3.3 requires Spring Cloud 2023

Recommendation:
Do not auto-upgrade.
Ask developer.


7. Hook integration

Design how Claude Code invokes this workflow.

Preferred flow:

User:
"Create PR"


Claude Code:

Before:
gh pr create


Invoke:

security.scan_before_pr()


If fixes required:

security.apply_fix()


Run:

mvn test


Then:

git commit


Then:

gh pr create


8. Deployment architecture

Provide options:

Option A:
Local MCP server

Developer laptop:

Claude Code
 |
Security MCP Server
 |
Snyk API


Option B:
Enterprise MCP service

Developer laptop:

Claude Code
 |
MCP Gateway
 |
Security MCP Service
 |
Snyk/GitHub/Jira


Recommend the best enterprise architecture.


9. Security considerations

Address:

- API key management
- Snyk token handling
- GitHub permissions
- audit logs
- developer approval for risky changes
- preventing malicious dependency changes


10. Deliverables

Produce:

1. Architecture diagram
2. Component design
3. MCP server API design
4. Claude Code hook configuration
5. Example MCP tool definitions
6. Snyk API integration approach
7. GitHub PR workflow
8. Sample implementation skeleton
9. Rollout plan for developers
10. Metrics:

Measure:
- reduction in failed PR builds
- vulnerabilities fixed before PR
- developer time saved
- security debt reduction


Design this as a production-grade internal developer platform capability similar to what large engineering organizations build.
```
