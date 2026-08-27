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


building a dashboard for developers and managers where enviroment is very comples like I ahve 13 aws regions and 40+ sevices, developer makes a PR -> review -> Ci/CD -> tests -> deploy. But no idea where the PR has reached. Does it causing any problem. what feature flags are installed in each site. What version of database library site and service is using. What versin of major libraries like kafka or http it is using , does it have cve ..... any mockups for this
Edit
what is compass

In the mockup I generated, “Compass” is just a placeholder name for the internal developer/platform dashboard I was envisioning.

The idea is a single “map” of your software estate:

PR → Review → CI/CD → Tests → Deploy — see exactly where every change is stuck.
13 AWS regions × 40+ services — understand what's running where.
Environment health — production/staging/dev status and deployment failures.
Feature flags — which flags are enabled in which environment/site.
Versions — service versions plus Kafka, HTTP clients, database libraries, etc.
Security — CVEs affecting each service/library and their severity.
Dependencies — service → library → version → vulnerability relationships.
Impact analysis — e.g. “Kafka library has a critical CVE; these 7 services across these 4 regions are affected.”
Manager view — DORA metrics, deployment failures, release progress, risk.
Developer view — “My PR #4821 is currently blocked in integration tests.”

So I'd think of Compass as a product concept, not necessarily the name. The core product is essentially:

“A real-time map of everything happening from code change to production, across every service, environment, region, dependency, and risk.”

And I think the most valuable part of your idea is correlating all those dimensions, rather than building yet another CI/CD dashboard.

Ad
Ads do not influence the answers you get from ChatGPT. Your chats stay private. 
Learn about ads
