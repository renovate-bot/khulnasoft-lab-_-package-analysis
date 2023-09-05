[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/khulnasoft-lab/package-analysis/badge)](https://api.securityscorecards.dev/projects/github.com/khulnasoft-lab/package-analysis)

# Package Analysis

The Package Analysis project analyses the capabilities of packages available on open source repositories. The project looks for behaviors that indicate malicious software: 

- What files do they access? 
- What addresses do they connect to? 
- What commands do they run? 

The project also tracks changes in how packages behave over time, to identify when previously safe software begins acting suspiciously. 

This effort is meant to improve the security of open source software by detecting malicious behavior, informing consumers selecting packages, and providing researchers with data about the ecosystem. 

This code is designed to work with the
[Package Feeds](https://github.com/khulnasoft-lab/package-feeds) project,
and originally started there.

For examples of what this project has detected, check out the
[case studies](docs/case_studies.md).

## How it works

The project's components are:

- A [scheduler](./cmd/scheduler/) - creates jobs for the analysis worker from
  Package Feeds.
- Analysis (one-shot [analyze](./cmd/analyze/) and [worker](./cmd/worker/)) -
  collects package behavior data through static and dynamic analysis of each
  package.
- A [loader](./function/loader/) - pushes the analysis results into BigQuery.

The goal is for all of these components to work together and provide extensible,
community-run infrastructure to study behavior of open source packages and to
look for malicious software. We also hope that the components can be used
independently, to provide package feeds or runtime behavior data for anyone
interested.

The Package Analysis project currently consists of the following pipeline:

![image](docs/images/Pipeline%20diagram.png)

1. Package repositories are monitored for new packages.
1. Each new package is scheduled to be analyzed by a pool of workers.
1. A worker performs dynamic analysis of the package inside a sandbox.
1. Results are stored and imported into BigQuery for inspection.

Sandboxing via [gVisor](https://gvisor.dev/) containers ensures the packages are
isolated. Detonating a package inside the sandbox allows us to capture strace
and packet data that can indicate malicious interactions with the system as well
as network connections that can be used to leak sensitive data or allow remote
access.
