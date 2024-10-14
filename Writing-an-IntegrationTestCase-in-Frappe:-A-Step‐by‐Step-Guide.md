This article provides concrete examples of how to write an `IntegrationTestCase` in Frappe, showcasing three different variants: a Doctype test case with extra dependencies and ignores, a non-doctype test case using extra dependencies, and a Doctype test case using a module's `test_records` attribute.

## 1. Doctype Test Case with Extra Dependencies and Ignores

In this variant, we create a Doctype test case that specifies additional dependencies and ignores certain dependencies using module-level attributes.

### Example

```python
# Module-Level Configuration
EXTRA_TEST_RECORD_DEPENDENCIES = ["Address"]
IGNORE_TEST_RECORD_DEPENDENCIES = ["Contact"]

import frappe
from frappe.tests import IntegrationTestCase

class CustomerTestCase(IntegrationTestCase):
    # Implicitly sets the doctype to "Customer" based on the doctypes json definition file

    def test_customer_retrieval(self):
        # test records are already created and loaded
        customer_test_record = self.globalTestRecords["Customer"][0]
        customer = frappe.get_doc("Customer", {"customer_name": customer_test_record["name"})
        self.assertEqual(customer.customer_name, "Test Customer")
```

## 2. Non-Doctype Test Case Using Extra Dependencies

This example demonstrates a non-doctype test case where extra dependencies are declared to specify required test records.

### Example

```python
# Module-Level Configuration
EXTRA_TEST_RECORD_DEPENDENCIES = ["Item", "Customer"]

import frappe
from frappe.tests import IntegrationTestCase

class AppTestCase(IntegrationTestCase):
    # No implicit doctype since it's a non-doctype test case
    # But test records are loaded according to the module co fig above

    def test_customer_retrieval(self):
        # test records are already created and loaded
        customer_test_record = self.globalTestRecords["Customer"][0]
        customer = frappe.get_doc("Customer", {"customer_name": customer_test_record["name"})
        self.assertEqual(customer.customer_name, "Test Customer")
```

## 3. Doctype Test Case Using Module's `test_records` Attribute

In this variant, we use the module's `test_records` attribute to allow for downstream monkey patching.

### Example

```python
# Module-Level Configuration
test_records = [
    {"doctype": "Sales Invoice", "customer": "Existing Customer"}
]

import frappe
from frappe.tests import IntegrationTestCase

class InvoiceTestCase(IntegrationTestCase):
    # Implicitly sets the doctype to "Sales Invoice" based on the doctype's json definition
    # This also loads Customer records, because it's a linked field of a Sales Invoice

    def test_customer_retrieval(self):
        # test records are already created and loaded
        customer_test_record = self.globalTestRecords["Customer"][0]
        customer = frappe.get_doc("Customer", {"customer_name": customer_test_record["name"})
        self.assertEqual(customer.customer_name, "Test Customer")
```

## Conclusion

These examples illustrate how to write `IntegrationTestCase` in Frappe for different scenarios. Whether using module-level attributes like `EXTRA_TEST_RECORD_DEPENDENCIES` and `IGNORE_TEST_RECORD_DEPENDENCIES`, or leveraging module attributes for test records, these strategies provide flexibility and control over the testing process.

