# Kiln
![](https://github.com/simplybusiness/kiln/workflows/CI/badge.svg)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v1.4%20adopted-ff69b4.svg)](CODE_OF_CONDUCT.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) [![Join the chat at https://gitter.im/Kiln-security/community](https://badges.gitter.im/Kiln-security/community.svg)](https://gitter.im/Kiln-security/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Kiln is a tool that makes it easy to run a range of security tools on project repositories, automatically notify teams of issues via connectors such as Slack, and monitor trends in findings to guide AppSec efforts within an organisation.  Kiln has been designed as a collection of dockerised application security tools, a validating HTTP proxy to forward the tool output to an Apache Kafka cluster and a collection of connectors to consume data from the Apache Kafka cluster.
Some of the current use cases for Kiln are: 

- Collect application dependency security issues for Ruby and Python, with other languages to follow.
- Notify teams on Slack about issues in their application dependencies.
- Suppress issues if they are false positives (either for a specific period of time or indefinitely).
- Build custom service integrations by consuming events from Kafka.
- Stream event data from Kafka to a data lake or process the event stream directly to perform analysis, reporting and metric calculation.
- Run analyses on the data and monitor trends such as which repositories have most vulnerable dependencies, what is the average time to fix issues within and across teams. 

## Architecture
Kiln is architected as a modular, event sourcing system with only two required components: the Kiln Data Collector and an Apache Kafka cluster. When you run a Kiln Security Scanner, the tool output is send to the data-collector, which acts as a data validation point and HTTP interface to the Apache Kafka cluster. The data-collector then inserts the tool output and some additional metadata into a Kafka topic. For an introduction to Event Sourcing, checkout https://dev.to/barryosull/event-sourcing-what-it-is-and-why-its-awesome.

All Kiln Connectors are Kafka consumers that process the events in the tool output topic and respond accordingly. For example, the Slack connector can consume events as they're added to the topic, compare the application name to a list of applications it knows about and send a message to the appropriate Slack channel with new security findings.

Kiln Security Scanners are docker containers with security tools baked into the image and also include a small binary that takes the output from the tool and sends it to the Kiln Data Collector to be recorded.

![Kiln architecture diagram](docs/images/Kiln%20Architecture%20diagram.svg)

## Trying Kiln out for yourself

If you're interested in trying Kiln out, there is a [Quickstart](docs/quickstart/README.md) guide which walks you through deploying a Kubernetes cluster to AWS and then deploying Kiln to that cluster. Once you have Kiln deployed, there is also a [data analysis guide](docs/data-analysis/README.md) which walks you through setting up JupyterHub and analysing data generated by Kiln to answer the question: Given several projects, what is their average time to remediate a vulnerability in a dependency present in their default branch.

## Contributing
Please note that this project is released with a Contributor Code of Conduct. By participating in this project, you agree to abide by its terms. The Code of Conduct can be found [here](CODE_OF_CONDUCT.md).

There are many ways you can contribute to Kiln, and writing Rust is only one of them. You could:
- Try Kiln out on your own projects and file bug reports
- Write documentation or raise issues with the documentation
- Review current test coverage (both unit and integration testing) and raise issues for gaps in coverage
- Suggest new tools or service connectors
- Write about interesting ways to use the data Kiln generates

To contribute to Kiln, you'll need the following tools installed:
- Rust (stable channel, assuming 1.40 as minimum)
- Clippy (For linting)
- Cargo Make (For building docker images)
- Docker
- Docker-compose (For integration testing)
- OpenSSL 1.1.0 or higher
- CMake  (For building Kafka native dependencies)

Kiln is still in it's early stages and isn't ready for production use. However, contributions are welcome! If you want to make a change to the project:
- Open an issue to discuss the change (if the change is significant)
- Fork this repo
- Create a new branch in your fork
- Make your change
- Add new tests & ensure existing tests pass
- Ensure linting passes
- Open a PR and explain what changes you have made
- Wait for CI to pass and PR to be reviewed
- Merge!

## Versioning
Kiln follows [SemVer 2.0](https://semver.org/) for versioning and all components are versioned in lockstep.

Docker images use two sets of tags: git-$VERSION and $VERSION. The former is used for images built after a merge to the main branch, the latter is used for released versions of Kiln. Previously, images for tools also included the version of the tool in the image tag, but tool upgrades will now be handled in the same way as other Kiln updates using the semver tag.

When a version is released, it will also overwrite the less specific semver compatible tags. For example, in the 0.2 series, if version 0.2.1 was released, this would also overwrite the 0.2 and latest tags.

The CLI will attempt to pull and use tool images with a version number that matches its own.
