This page is intended to make it easier for users who maintain custom apps/forks to migrate their installations to `develop` branch aka the nightly version. This page assumes that you're on `v15.x.x`

---

### Removed view-specific translations

In the previous versions it was possible to add translations for a specific view only. For this, the `get_translated_dict` hook was used. In the previous versions we started sending all translations regardless of the view. Now we also removed the `get_translated_dict` hook. If you included view-specific translations in this way, please move them to your app's translation files.

Web forms are the only exception. They don't receive all translations. Instead, they get a small dictionary with translations for the labels used in the web form.

Along with this change we also included the following changes:

- Currency codes and timezone names no longer get translated. (This used to be the case in **Setup Wizard** and **System Settings** only.)
- Translations of country names are be available everywhere by default. 
- The method `frappe.geo.country_info.get_translated_dict` is deprecated and only returns translations of country names. Use `get_translated_countries` instead.
- The methods `frappe.get_lang_dict`, `frappe.translate.get_dict`, `frappe.translate.get_lang_js` and `frappe.translate.get_dict_from_hooks` have been removed. Translations are available via `_()`.
- `doc.meta.__messages` does no longer hold doc-specific translations

https://github.com/frappe/frappe/pull/22962


### Permission system breaking changes

1. `has_permission` hooks now need to explicitly return True. Current behaviour allowed returning `None` or non-False value to allow user. [#24253](https://github.com/frappe/frappe/pull/24253)
2. `frappe.permission.has_permission` function no longer accepts misleading "raise_exception" parameter, use `print_logs` instead. [#24266](https://github.com/frappe/frappe/pull/24266)


### Default sorting changed from `modified` to `creation`

- Starting with v16 all list views by default will be sorted by creation.
- This decision was taken considering multiple things:
    - `creation` is better default for most business documents (currently primary use case for many Frappe apps)
    - `creation` ensures stable list view even when things are rapidly changing. 
    - `creation` is better from performance point of view as the field doesn't change and hence index updates are not required. This reduces contention on index updates.


What do you as app developer need to do?
- Change default sort ordering in your all of your DocType from `modified` to `creation`
- If you don't change it then both `creation` and `modified` will be indexed. 
- If you change sorting to `creation` then modified index will be dropped while migrating. 


**WARNING:** This change also affects how database APIs work. Most Frappe database APIs implicitly sorted by `modified`, they'll now implicitly sort by `creation`. 

Example: `frappe.get_all("DocType")` was actually translated to `frappe.get_all("DocType", order_by="modified desc")`. This is now changed to `frappe.get_all("DocType", order_by="creation desc")`

It's possible that some of your business logic dependent on this, so it's advisable to audit your code to find out if `modified` ordering affects any of your business logic and explicitly add `order_by='modified desc'`.

Note: You can re-add index by adding following code in your doctype controller. 

```
def on_doctype_update():
    frappe.db.add_index("Doctype Name", ["modified"])
```

