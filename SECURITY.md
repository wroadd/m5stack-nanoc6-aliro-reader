# Security policy

## Scope

This repository is an experimental reference for Aliro NFC and Matter credential
attribution. It has not undergone a production security audit and must not be
treated as a complete physical-access-control system.

## Credential handling

- The implementation reads only the public access-credential key needed for
  matching.
- Raw credential keys, PINs, Matter fabric data, and persistent transaction
  secrets must never be logged.
- Do not publish flash or NVS dumps from a commissioned device.
- Use secure boot, flash encryption, protected provisioning, and an appropriate
  hardware threat model before production deployment.

## Version coupling

Credential attribution uses the tested Aliro library's internal NVS namespace
and key naming. Treat any `esp_aliro_lib` upgrade as a security-sensitive change:
review the storage format, rebuild from clean sources, and repeat both successful
and rejected-credential tests.

## Reporting a vulnerability

Please use GitHub's private vulnerability reporting feature for this repository.
Do not include real credentials, private keys, PINs, fabric identifiers, device
addresses, or flash dumps in an issue.
