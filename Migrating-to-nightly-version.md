This page is intended to make it easier for users who maintain custom apps/forks to migrate their installations to `develop` branch aka the nightly version. This page assumes that you're on `v15.x.x`

---

### Removed view-specific translations

In the previous versions it was possible to add translations for a specific view only. For this, the `get_translated_dict` hook was used. In the previous versions we started sending all translations regardless of the view. Now we also removed the `get_translated_dict` hook. If you included view-specific translations in this way, please move them to your app's translation files.

Along with this change we also included the following changes:

- Currency codes and timezone names no longer get translated. (This used to be the case in **Setup Wizard** and **System Settings**.)
- Translations of country names are be available everywhere by default. 
- The method `frappe.geo.country_info.get_translated_dict` is deprecated and only returns translations of country names. Use `get_translated_countries` instead.