# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2025-01-08

## [1.0.1] - 2025-01-07

### Added
- Initial production release
- Universal constraint validation for any Django database backend
- Multi-database support with per-database configuration
- Conditional and non-conditional unique constraint support
- Race condition protection with select_for_update()
- Comprehensive test suite with 44 tests
- Management commands for constraint discovery
- Complete bookstore demo with multi-database setup

### Features
- Application-level validation via Django pre_save signals
- Automatic constraint conversion from Django models
- Support for UniqueConstraint and unique_together
- Q object condition evaluation
- Professional documentation and examples
