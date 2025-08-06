# Django Universal Constraints

A Django library that provides universal constraint validation for any database backend, including those that don't natively support unique/conditional constraints.

## Why Django Universal Constraints?

Some Django database backends (like `django-ydb-backend`) don't support unique constraints. This library provides transparent application-level validation that works with **any** Django database backend.

### Before (Broken)
```python
class User(models.Model):
    email = models.EmailField()
    is_active = models.BooleanField(default=True)
    
    class Meta:
        constraints = [
            UniqueConstraint(
                fields=['email'],
                condition=Q(is_active=True),  # ❌ Fails on some backends
                name='unique_active_email'
            )
        ]
```

### After (Fixed)
```python
# Same model definition - no changes needed!
class User(models.Model):
    email = models.EmailField()
    is_active = models.BooleanField(default=True)
    
    class Meta:
        constraints = [
            UniqueConstraint(
                fields=['email'],
                condition=Q(is_active=True),  # ✅ Works everywhere
                name='unique_active_email'
            )
        ]
```

## Features

- **Universal Backend Support**: Works with any Django database backend
- **Zero Code Changes**: Existing models work unchanged
- **Automatic Validation**: Transparent constraint validation via Django signals
- **Per-Database Configuration**: Configure different settings for different databases
- **Race Condition Protection**: Optional `select_for_update()` support
- **Management Commands**: CLI tools for constraint discovery and conversion
- **Production Ready**: Comprehensive error handling and logging

## Installation

Install using uv (recommended) or pip:

```bash
# Using uv
uv add django-universal-constraints

# Using pip
pip install django-universal-constraints
```

## Quick Setup

### 1. Add to INSTALLED_APPS

```python
# settings.py
INSTALLED_APPS = [
    # ... your apps
    'universal_constraints',
]
```

### 2. Configure Per-Database Settings (Optional)

```python
# settings.py
UNIVERSAL_CONSTRAINTS = {
    'database': {  # Database alias
        'EXCLUDE_APPS': ['admin', 'auth', 'contenttypes', 'sessions'],
        'RACE_CONDITION_PROTECTION': True,
        'REMOVE_DB_CONSTRAINTS': True,
        'LOG_LEVEL': 'INFO',
    },
    # Add more databases as needed
}
```

### 3. Use Database Wrapper (Optional, for REMOVE_DB_CONSTRAINTS)

If you want to remove database constraints and handle them purely at the application level (constraints wont be created during migrations):

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'universal_constraints.backend',
        'WRAPPED_ENGINE': 'django.db.backends.sqlite3',  # Your actual backend
        'NAME': 'db.sqlite3',
        # ... other WRAPPED_ENGINE backend settings
    }
}
```

## Configuration Reference

### UNIVERSAL_CONSTRAINTS Settings

Configure per-database settings in your Django settings:

```python
UNIVERSAL_CONSTRAINTS = {
    'database_alias': {
        # Apps to exclude from processing
        'EXCLUDE_APPS': ['admin', 'auth', 'contenttypes', 'sessions'],
        
        # Enable race condition protection with select_for_update()
        'RACE_CONDITION_PROTECTION': True,  # Default: True
        
        # Remove database constraints (requires backend wrapper)
        'REMOVE_DB_CONSTRAINTS': True,  # Default: True
        
        # Logging level for constraint operations
        'LOG_LEVEL': 'INFO',  # Options: DEBUG, INFO, WARNING, ERROR
    }
}
```

### Settings Explanation

- **EXCLUDE_APPS**: Django apps to skip during constraint processing
- **RACE_CONDITION_PROTECTION**: Use database locking to prevent race conditions
- **REMOVE_DB_CONSTRAINTS**: Remove constraints from database schema (requires wrapper)
- **LOG_LEVEL**: Control logging verbosity for debugging

## Usage Examples

### Basic Conditional Constraints

```python
from django.db import models
from django.db.models import Q, UniqueConstraint

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()
    is_active = models.BooleanField(default=True)
    
    class Meta:
        constraints = [
            UniqueConstraint(
                fields=['email'],
                condition=Q(is_active=True),
                name='unique_active_author_email'
            )
        ]

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    isbn = models.CharField(max_length=13)
    is_published = models.BooleanField(default=False)
    
    class Meta:
        constraints = [
            # Published books must have unique ISBNs
            UniqueConstraint(
                fields=['isbn'],
                condition=Q(is_published=True),
                name='unique_published_isbn'
            ),
            # Each author can only have one book with the same title
            UniqueConstraint(
                fields=['title', 'author'],
                name='unique_title_per_author'
            )
        ]
```

### Multiple Database Configuration

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'main_db',
        # ... other settings
    },
    'ydb_database': {
        'ENGINE': 'universal_constraints.backend',
        'WRAPPED_ENGINE': 'ydb_backend.backend',
        'NAME': 'ydb_database',
        # ... other YDB-specific settings
    }
}

UNIVERSAL_CONSTRAINTS = {
    # No entry for 'default' - PostgreSQL handles constraints natively
    'ydb_database': {
        'REMOVE_DB_CONSTRAINTS': True,
        'LOG_LEVEL': 'DEBUG',
    }
}
```

## Management Commands

### Discover Constraints

Find all constraints in your project:

```bash
uv run python manage.py discover_constraints
```

## Advanced Usage

### Manual Constraint Registration

Add constraints programmatically:

```python
from universal_constraints.validators import add_universal_constraint
from django.db.models import Q

# Add constraint to existing model
add_universal_constraint(
    User,
    fields=['username'],
    condition=Q(is_active=True),
    name='unique_active_username'
)
```

### Custom Validation Logic

Use the mixin for custom validation:

```python
from universal_constraints.validators import UniversalConstraintValidatorMixin

class MyModel(UniversalConstraintValidatorMixin, models.Model):
    name = models.CharField(max_length=100)
    is_active = models.BooleanField(default=True)
    
    class Meta:
        # Define constraints as usual
        pass
    
    def clean(self):
        # Custom validation logic
        super().clean()  # This will validate constraints
        # Add your custom validation here
```

## Database Backend Compatibility

### Supported Backends

- ✅ **SQLite** (`django.db.backends.sqlite3`)
- ✅ **PostgreSQL** (`django.db.backends.postgresql`)
- ✅ **MySQL** (`django.db.backends.mysql`)
- ✅ **YDB** (`django-ydb-backend`)
- ✅ **Any Django-compatible backend**

### Backend Wrapper

The universal backend wrapper works with any Django database backend:

```python
DATABASES = {
    'wrapped_backend': {
        'ENGINE': 'universal_constraints.backend',
        'WRAPPED_ENGINE': 'your.actual.backend',  # Any Django backend
        # ... other settings for your backend
    }
}
```

## Testing

Run the test suite:

```bash
uv sync
uv run python tests/runtests.py
```

## Troubleshooting

### Common Issues

**1. "No such table" errors in tests**
- Ensure test models are properly registered
- Check that Django migrations are applied

**2. Backend wrapper warnings**
- Use `universal_constraints.backend` as ENGINE when `REMOVE_DB_CONSTRAINTS=True`
- Set `WRAPPED_ENGINE` to your actual database backend

**3. Constraints not being validated**
- Check that the database is configured in `UNIVERSAL_CONSTRAINTS`
- Ensure the app is not in `EXCLUDE_APPS`

### Debug Logging

Enable debug logging to troubleshoot issues:

```python
UNIVERSAL_CONSTRAINTS = {
    'your_database': {
        'LOG_LEVEL': 'DEBUG',
        # ... other settings
    }
}

LOGGING = {
    'version': 1,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'universal_constraints': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
```

## Contributing

Contributions are welcome! Please see our contributing guidelines and submit pull requests to our GitHub repository.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Credits

Built with ❤️ using Django and modern Python development practices.