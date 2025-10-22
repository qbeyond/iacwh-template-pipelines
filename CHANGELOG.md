# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.2.1] - 2025-10-22

### Added 

- added parameter `azureRmUseIdTokenGeneration` for setting `backendAzureRmUseIdTokenGeneration` and `environmentAzureRmUseIdTokenGeneration`
    - `azureRmUseIdTokenGeneration` must be set to `true` when using a `service connection with Workload Identity Federation` and `AzureRm Provider version >3.80` to prevent authentication issues