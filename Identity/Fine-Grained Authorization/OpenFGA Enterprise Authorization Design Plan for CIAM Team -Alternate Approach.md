# OpenFGA Enterprise Authorization Design Plan for CIAM Team

## Executive Summary

This comprehensive design plan outlines a modular, scalable OpenFGA authorization architecture for your enterprise. It enables independent development and management by CIAM and domain teams, using OpenFGA's `.mod` manifests to directly define schema inline (types and relationships) for each domain module. The CIAM team manages core entities like organizations, users, groups, and applications with system roles, while domain teams independently manage their own resources (e.g., printers, IoT devices) under organizational context without system role cross-access.

***

## Architecture Overview

### Directory Structure & Modular Modeling

```
core/
  core.fga           # Core CIAM types and relationships
  core.mod           # CIAM module manifest

fleet/
  fleet.mod          # Fleet domain manifest defining types/relations inline

print/
  print.mod          # Printer domain manifest defining types/relations inline

hr/
  hr.mod             # Example HR domain manifest
```

- Each `.mod` manifest imports `core.fga` for foundational CIAM definitions.
- Each domain manifest defines or extends types inline using OpenFGA's modeling syntax.
- System roles live *only* in the core CIAM domain.
- Organization is the universal context anchor but does *not* confer cross-domain access.

***

## Step 1: Core CIAM Module

**File: core/core.fga**

```openfga
module core

type user

type organization
  relations:
    define member: [user]
    define system_admin: [user]
    define org_admin: [user]
    define users_admin: [user]
    define partner_admin: [user]
    define helpdesk: [user]
    define child: [organization]
    define parent: [organization]

type group
  relations:
    define organization: [organization]
    define member: [user, group#member]
    define owner: [user]

type application
  relations:
    define organization: [organization]
    define owner: [user]
```

**File: core/core.mod**

```yaml
schema: "1.2"
contents:
  - ./core.fga
```

***

## Step 2: Fleet Domain Module (`fleet/fleet.mod`)

```yaml
schema: "1.2"
contents:
  - ../core/core.fga

types:
  - extend type organization
      relations:
        define fleet_admin: [user, group#member]
        define fleet_operator: [user, group#member]

  - type fleet
      relations:
        define organization: [organization]
        define fleet_admin: [user, organization#fleet_admin]
        define fleet_operator: [user, organization#fleet_operator]

  - type iot_device
      relations:
        define organization: [organization]
        define fleet: [fleet]
        define assigned_user: [user]
        define device_admin: [user, fleet#fleet_admin]
        define device_operator: [user, fleet#fleet_operator]
        define can_configure: device_admin
        define can_monitor: device_operator or device_admin
```

### Tuples (`fleet/fleet-tuples.yaml`)

```yaml
tuples:
  - user: user:alice
    relation: fleet_admin
    object: organization:org-abc
  - user: user:bob
    relation: fleet_operator
    object: organization:org-abc

  - user: organization:org-abc
    relation: organization
    object: fleet:fleet-x

  - user: fleet:fleet-x
    relation: fleet
    object: iot_device:device-100

  - user: fleet:fleet-x#fleet_admin
    relation: device_admin
    object: iot_device:device-100
  - user: fleet:fleet-x#fleet_operator
    relation: device_operator
    object: iot_device:device-100
```

### Tests (`fleet/fleet-test.yaml`)

```yaml
model_file: fleet.mod

tuples:
  # Include tuples from fleet-tuples.yaml here or reference

tests:
  - name: Fleet admin has device admin rights
    check:
      - user: user:alice
        relation: can_configure
        object: iot_device:device-100
        expected: true

  - name: Fleet operator can monitor but not configure
    check:
      - user: user:bob
        relation: can_monitor
        object: iot_device:device-100
        expected: true
      - user: user:bob
        relation: can_configure
        object: iot_device:device-100
        expected: false

  - name: No access for unrelated user
    check:
      - user: user:eve
        relation: can_monitor
        object: iot_device:device-100
        expected: false
```

***

## Step 3: Print Domain Module (`print/print.mod`)

```yaml
schema: "1.2"
contents:
  - ../core/core.fga

types:
  - extend type organization
      relations:
        define print_admin: [user, group#member]
        define print_user: [user, group#member]

  - type printer
      relations:
        define organization: [organization]
        define assigned_user: [user]
        define printer_admin: [user, organization#print_admin]
        define printer_user: [user, organization#print_user]
        define can_configure: printer_admin
        define can_use: printer_user or printer_admin
```

### Tuples (`print/print-tuples.yaml`)

```yaml
tuples:
  - user: user:carol
    relation: print_admin
    object: organization:org-xyz
  - user: user:dave
    relation: print_user
    object: organization:org-xyz

  - user: organization:org-xyz
    relation: organization
    object: printer:printer-1

  - user: organization:org-xyz#print_admin
    relation: printer_admin
    object: printer:printer-1
  - user: organization:org-xyz#print_user
    relation: printer_user
    object: printer:printer-1
```

### Tests (`print/print-test.yaml`)

```yaml
model_file: print.mod

tuples:
  # Include print-tuples.yaml here or inline
  
tests:
  - name: Print admin can configure and use
    check:
      - user: user:carol
        relation: can_configure
        object: printer:printer-1
        expected: true
      - user: user:carol
        relation: can_use
        object: printer:printer-1
        expected: true

  - name: Print user can use, not configure
    check:
      - user: user:dave
        relation: can_use
        object: printer:printer-1
        expected: true
      - user: user:dave
        relation: can_configure
        object: printer:printer-1
        expected: false

  - name: Unrelated user cannot use or configure
    check:
      - user: user:eve
        relation: can_use
        object: printer:printer-1
        expected: false
      - user: user:eve
        relation: can_configure
        object: printer:printer-1
        expected: false
```

***

## Step 4: Running Tests and Adding Models

### Running Tests Locally

Use the OpenFGA CLI command to run tests against each module:

```shell
# Validate syntax of the modular model
fga model validate --file core/core.mod

# Run tests for core CIAM module (if applicable)
fga model test --tests core/tests/core-tests.yaml

# Run fleet domain tests
fga model test --tests fleet/fleet-test.yaml

# Run print domain tests
fga model test --tests print/print-test.yaml
```

Or create a shell script `test-all.sh`:

```bash
#!/bin/bash
set -e

echo "Validating core model"
fga model validate --file core/core.mod

echo "Running core CIAM tests"
fga model test --tests core/tests/core-tests.yaml

echo "Running Fleet domain tests"
fga model test --tests fleet/fleet-test.yaml

echo "Running Print domain tests"
fga model test --tests print/print-test.yaml
```

Make executable and run:

```shell
chmod +x test-all.sh
./test-all.sh
```

***

### Adding Models to OpenFGA Store

After successful validation and testing, add or update the model in your OpenFGA store:

```bash
fga model write --store-id=$FGA_STORE_ID --file core/core.mod
fga model write --store-id=$FGA_STORE_ID --file fleet/fleet.mod
fga model write --store-id=$FGA_STORE_ID --file print/print.mod
```

- Models are composable; adding domain manifests extends the global authorization model.
- Repeat this for each domain manifest for incremental, isolated changes.

***

## Summary

- Modular `.mod` manifests **directly define and extend schema inline**, improving maintainability over multiple `.fga` files.
- Tuples and tests live alongside domain modules for clear ownership.
- Tests are run per module to ensure domain-specific correctness and isolation.
- The core CIAM domain exclusively manages system roles and core identity resources.
- Domain teams manage their resource types and roles independently, linked by organization context.
- Use OpenFGA CLI commands to validate, test, and deploy your authorization models in stages.

This approach provides a future-proof, team-friendly, and scalable authorization architecture using OpenFGA’s best practices.

