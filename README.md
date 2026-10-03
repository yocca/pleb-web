# pleb-web

Next.js web app for Pleb, a happy hour finder. Shows bars and restaurants with happy hours near you, starting with the West Village in NYC.

## Local development

Runs against `pleb-api`; see its README for the docker compose setup.

## Specs

This repo uses [OpenSpec](https://github.com/Fission-AI/OpenSpec) for spec-driven changes. Project context and sourcing rules are in `openspec/config.yaml`; current specs are in `openspec/specs/`, proposed changes in `openspec/changes/`. In Claude Code, use `/opsx:propose`, `/opsx:apply` and `/opsx:archive`.
