# Frappe Test Record Loading: Startup Sequence

This article explains the startup sequence in Frappe that leads to the loading of test records. Understanding this sequence is crucial for developers who want to ensure that their integration tests are set up correctly.

## Overview

The process of loading test records in Frappe involves several steps, starting from the initial environment setup to the invocation of test record generation for each `IntegrationTestCase`. This sequence ensures that all necessary dependencies are resolved and test data is available for testing.

## Startup Sequence

### 1. Environment Setup

The first step occurs during the environment setup, immediately after the test runner execution. The `_create_global_test_record_dependencies` method creates global test record dependencies for a given app.

```python
@staticmethod
def _create_global_test_record_dependencies(app: str, category: str):
    """Create global test record dependencies"""
    test_module = frappe.get_module(f"{app}.tests")
    if hasattr(test_module, "global_test_dependencies"):
        for doctype in test_module.global_test_dependencies:
            make_test_records(doctype, commit=True)
```

### 2. IntegrationTestCase Setup

For each `IntegrationTestCase`, the `setUpClass` method sets up the test environment, including loading necessary test records.

```python
class IntegrationTestCase(UnitTestCase):
    @classmethod
    def setUpClass(cls) -> None:
        super().setUpClass()

        # Site initialization
        frappe.init(cls.TEST_SITE)

        # Create test record dependencies
        cls._newly_created_test_records = []
        if cls.doctype and cls.doctype not in frappe.local.test_objects:
            cls._newly_created_test_records += make_test_records(cls.doctype)
        elif not cls.doctype:
            to_add, ignore = get_missing_records_module_overrides(cls.module)
            for doctype in to_add:
                cls._newly_created_test_records += make_test_records(doctype)

        # Flush changes
        frappe.db.commit()
```
**Note**: Doctype tests load their dependencies automatically based on the current doctype, while all other tests rely on explicit declaration of module level dependencies. More below.


### 3. Dependency Discovery

The `get_missing_records_doctypes` function performs a depth-first traversal to discover all dependencies for a specified DocType, while also taking into account its module level overrides.

```python
def get_missing_records_doctypes(doctype, visited=None) -> list[str]:
    if visited is None:
        visited = set()

    if doctype in visited:
        return []

    visited.add(doctype)

    module, test_module = get_modules(doctype)
    meta = frappe.get_meta(doctype)
    link_fields = meta.get_link_fields()

    unique_doctypes = dict.fromkeys(df.options for df in link_fields if df.options != "[Select]")
    
    to_add, to_remove = get_missing_records_module_overrides(test_module)
    unique_doctypes.update(dict.fromkeys(to_add))
    if to_remove:
        unique_doctypes = {k: v for k, v in unique_doctypes.items() if k not in to_remove}

    result = []
    for dep_doctype in unique_doctypes:
        result.extend(get_missing_records_doctypes(dep_doctype, visited))

    result.append(doctype)
    return result
```

### 4. Module-Level Overrides

The `get_missing_records_module_overrides` function checks for any module-level overrides that specify additional dependencies or exclusions.

```python
def get_missing_records_module_overrides(module) -> [list, list]:
    to_add = []
    to_remove = []

    if hasattr(module, "EXTRA_TEST_RECORD_DEPENDENCIES"):
        to_add += module.EXTRA_TEST_RECORD_DEPENDENCIES

    if hasattr(module, "IGNORE_TEST_RECORD_DEPENDENCIES"):
        to_remove += module.IGNORE_TEST_RECORD_DEPENDENCIES

    return to_add, to_remove
```

## Conclusion

The startup sequence for loading test records in Frappe involves several key steps that ensure a comprehensive and efficient setup of integration tests. By understanding this sequence—from global dependency creation during environment setup to per-test case dependency resolution—developers can effectively manage their testing environments. This guide highlights how these processes work together and provides insight into customizing and optimizing the loading of test records through module-level overrides and dependency discovery.

