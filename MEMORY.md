# Project Memory

This file captures project learnings that persist across agent sessions.

## Project Overview

**tf-az-overlays-bastionhost** is a Terraform module that deploys an Azure Bastion Host into a hub virtual network. Part of the POps-Rox overlay module ecosystem.

- **Repo:** `POps-Rox/tf-az-overlays-bastionhost`
- **License:** MIT (Microsoft Corporation)
- **Terraform:** >= 1.3
- **Providers:** azurerm ~> 3.22, azurenoopsutils ~> 1.0

## What It Deploys

- Azure Bastion Host (Basic or Standard SKU)
- Public IP for the Bastion service
- Dedicated `AzureBastionSubnet` subnet
- Optional management locks on Bastion host and PIP
- Optional resource group creation

## Key Architecture Patterns

- **Naming:** Uses `azurenoopsutils_resource_name` data sources for consistent Azure CAF naming. Custom names override via `custom_bastion_name`, `custom_public_ip_name`, `custom_ipconfig_name`.
- **Tagging:** Default tags via `locals-tags.tf` with `deployedBy`, `environment`, `workload` fields. Merged with user-provided `add_tags`.
- **Module dependencies:** Consumes `tf-az-overlays-azregionslookup` (region short names) and `tf-az-overlays-resourcegroup` (optional RG creation) from the POps-Rox org.
- **Resource group:** Either reference an existing one (`existing_resource_group_name`) or create a new one (`create_bastion_resource_group = true`).
- **File layout:** Resources split by concern — `resources.bastion.tf` (host + PIP), `resources.bastion.network.tf` (subnet), `resources.bastion.locks.tf` (locks), `scaffolding.tf` (region lookup + RG).

## Testing

- **Framework:** Terratest + Azure terraform-module-test-helper (Go 1.19)
- **Location:** `test/e2e/terraform_test.go`, `test/upgrade/upgrade_test.go`
- **Run:** `cd test && go test ./e2e/...`
- **Examples:** `examples/basic/` used by e2e tests

## CI/CD

- **GitHub Actions:** `.github/workflows/ci.yml`
- **Pipeline:** terraform fmt → terraform validate → TFLint → example validation
- **Linting:** `terraform fmt -check -recursive`, `tflint --recursive --minimum-failure-severity=error`, `terraform validate`

## Conventions

- Variables split across: `variables.tf` (main), `variables-naming.tf`, `variables-tags.tf`, `variables-locks.tf`
- Microsoft copyright headers on all `.tf` files
- Uses `tfmod-scaffold` via GNUmakefile for shared tooling
- Documentation via MkDocs (`mkdocs.yml`)

## Recent Decisions

- 2026-03-24: Teamwork framework installed and configured. All 17 agent files customized for this Terraform module project.
