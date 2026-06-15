# Distributed Systems Course Projects

Projects for the Distributed Systems course at the University of Tehran.

This repository collects multiple course assignments and experiments, grouped by assignment number. Each folder contains its own implementation details, notes, and in some cases screenshots or experiment results.

## Repository Structure

- CA1: Go-based assignments focused on distributed computing and containerized services.
	- `part1`: core interfaces and worker logic
	- `part2`: performance experiments, plots, and result analysis
	- `part3`: Dockerized service for running and testing the application
- CA2: Distributed systems services and integrations built around multiple virtual machines and RPC-based communication.
	- `auth-vm`: authentication service
	- `file-vm`: file service and file transfer support
	- `web-vm`: web front end that interacts with the backend services
	- `pubsub`: publisher/subscriber example for event-driven communication
	- `rpc-study`: RPC-related study material and experiments

## How to Use This Repo

1. Open the folder for the assignment you want to inspect.
2. Read the README inside that subproject for setup and run instructions.
3. Follow the screenshots, results, or notebook files included in the relevant folder when present.

## Notes

- The repository is organized as a course portfolio, so each subproject may use a different runtime setup.
- Most code is written in Go.
- If you are evaluating a specific assignment, start with the README inside that assignment folder rather than this top-level file.

## University Course Context

These projects were completed as part of the Distributed Systems course at the University of Tehran.
