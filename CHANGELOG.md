# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2025-01-08

### Changed
- **BREAKING CHANGE**: Removed `REMOVE_DB_CONSTRAINTS` setting and related functionality
- Simplified architecture to pure application-level validation approach
- Removed database backend wrapper complexity and migration interception
- Eliminated singleton pattern from AutoDiscovery class for better testability
- Removed redundant utility functions and cleaned up dead code
- Improved code quality and maintainability

### Removed
- `REMOVE_DB_CONSTRAINTS` configuration option
- Database constraint interception during migrations
- Backend wrapper functionality (`universal_constraints/backend/` directory)
- Singleton pattern implementation
- Redundant utility functions and entire `utils.py` module

### Technical Details
- The library now focuses exclusively on application-level constraint validation via Django signals
- Database backends handle constraint support according to their own capabilities
- Simplified configuration with only essential settings: `EXCLUDE_APPS`, `RACE_CONDITION_PROTECTION`, `LOG_LEVEL`
- Maintained 100% backward compatibility for core constraint validation functionality
- All 44 tests continue to pass with improved code quality

### Migration Guide
If you were using `REMOVE_DB_CONSTRAINTS` setting:
1. Remove the `REMOVE_DB_CONSTRAINTS` setting from your `UNIVERSAL_CONSTRAINTS` configuration
2. The library will continue to work with pure application-level validation
3. Database-level constraints will be handled by your database backend as intended

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
