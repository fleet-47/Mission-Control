# Project brief

## Purpose

Mission-Control will coordinate software work from human–agent pair programming through to multi-agent swarms. It should connect intent, specifications, tasks, implementation, and verification evidence while allowing existing tools to retain their responsibilities.

Fleet is the first intended user. The core should remain reusable by other software projects.

## Milestone 1: documented project and design

This milestone is complete when the repository contains a reviewed project specification and phased delivery plan, with:

- A clear product boundary and first end-to-end acceptance journey.
- Shared architectural principles, a concrete quality baseline, and the approval boundary for exceptions and requirement changes.
- A domain model connecting specifications, task dependencies, code, and verification evidence.
- A defined repository persistence and concurrency model, including recovery from mistaken changes.
- Evidence-backed integration and reuse decisions, with remaining risks stated explicitly.
- A GitHub Wayfinder map, decision issues, native relationships, and a linked Project that expose the route to implementation.

The current [Wayfinder map](https://github.com/fleet-47/Mission-Control/issues/1) records initial constraints and unresolved areas. This brief defines the milestone; it is not a completed architecture specification.

## Design workflow

Follow [Matt Pocock's Wayfinder](https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/wayfinder/SKILL.md): establish the destination, investigate factual questions, discuss human decisions, and expand the map only as the next questions become clear.

Decision details belong in their issue's resolution comment. The map links to those resolutions. Repository specifications will describe the resulting product contract, with links back to the reasoning rather than copied research transcripts.

Research does not imply an integration has been built or tested. Proposed behavior remains proposed until reviewed. A running implementation must not be reported complete merely because a document or task record exists.

## Reuse and delivery

Prefer existing Rust components and simple interfaces. The first milestone evaluates integration boundaries before importing code or defining a detailed implementation backlog.

Existing projects may remain standalone dependencies, be included as submodules, or contribute selected content after a reuse decision. Archival follows accounting for the work and confirming its replacement; it is not part of initial setup.

This is a public repository. Publish only project-appropriate material with permission; check licensing before importing third-party code, and keep secrets, private audit material, and raw execution logs out of commits and issues.
