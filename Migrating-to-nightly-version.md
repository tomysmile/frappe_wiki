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
2. `frappe.permission.has_permission` function no longer expects misleading "raise_exception" parameter. [#24266](https://github.com/frappe/frappe/pull/24266)