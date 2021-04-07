This page is intended to make it easier for users who maintain custom apps / forks to migrate their installations to Version 13.

---


### Whitelisting for `Document` methods

If you were accessing any document method using one of the following constructs, the method will need to be whitelisted in the relevant class.

- ```
  frm.call("method_name")
  ```
- ```
  frappe.call({
      doc: ...,
      method: "method_name"
  })
  ```

Docs: https://frappeframework.com/docs/user/en/api/form#frmcall