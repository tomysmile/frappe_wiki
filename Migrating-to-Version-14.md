This page is intended to make it easier for users who maintain custom apps/forks to migrate their installations to Version 14.

----

### Website Routing and Rendering refactor

There [was a major refractor](https://github.com/frappe/frappe/pull/12334) done for website routing and rendering. During this refactor few methods were moved to different files. You might have to change the following code in your custom app.

```diff
- from frappe.website.render import render
+ from frappe.website.serve import get_response
...
- response = render()
+ response = get_response()
```

```diff
- from frappe.website.render import clear_cache
+ from frappe.website.utils import clear_cache
```

