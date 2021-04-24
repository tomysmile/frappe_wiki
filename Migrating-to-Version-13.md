This page is intended to make it easier for users who maintain custom apps / forks to migrate their installations to Version 13.

---


### Whitelisting `Document` methods

If you were accessing any document methods using one of the following constructs, the method needs to be whitelisted in the doctype class. Additionally, if you were calling methods in the `options` of `Button` fields, those will need to be whitelisted as well.

```js
frm.call("my_method")
// or
frappe.call({
    doc: frm.doc,
    method: "my_method"
})
```

```diff
class ToDo:
+   frappe.whitelist()
    def my_method(self):
        pass
```

Docs: https://frappeframework.com/docs/user/en/api/form#frmcall

### Replace `jenv` hook with `jinja`



1. Open hooks.py
1. Rename the `jenv` hook to `jinja`
2. For each string in the `methods` list remove the part before `:` (colon) including the colon. It should only be a list of method paths.
3. Repeat the same for strings in `filters`.

For e.g.,
```diff
- jenv = {
+ jinja = {
    "methods": [
-        "get_fullname:custom_app.jinja.get_fullname"
+        "custom_app.jinja.get_fullname"
    ],
    "filters": [
-        "format_currency:custom_app.jinja.currency_filter"
+        "custom_app.jinja.format_currency"
    ]
}
```

custom_app/jinja.py
```diff
- def currency_filter():
+ def format_currency():
	...
```

Docs: https://frappeframework.com/docs/user/en/python-api/hooks#jinja-customization