This page is intended to make it easier for users who maintain custom apps/forks to migrate their installations to Version 14.

---

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

### New Build System based on esbuild

The new build system does not support `build.json`. To make sure bundles are built correctly, you need to create a bundle file for each key in `build.json`.

Let's say your build.json looks like this:
```json
{
    "js/my_app.js": [
        "public/js/utils.js",
        "public/js/main.js",
        "public/js/support.js"
    ],
    "js/another_file.js": [
        "public/js/another_file.js"
    ],
    "css/my_app.css": [
        "public/less/components.less",
        "public/less/style.less"
    ],
}
```

Create a file named `my_app.bundle.js` in the `public/js` directory and import the 3 files.

```js
// public/js/my_app.bundle.js

import './utils';
import './main';
import './support';
```

Now, since another_file.js is built with a single file, we can just rename that file from `another_file.js` to `another_file.bundle.js`

Now, create a file named `my_app.bundle.less` in the `public/less` directory and import the 2 less files.

```css
@import './components.less';
@import './style.less';
```

Now, assuming you included some of these files as part of app bundle or website bundle in hooks.py, you may need to do the following changes:

```diff
- app_include_js = ['/assets/js/my_app.js']
+ app_include_js = ['my_app.bundle.js']
- app_include_css = ['/assets/css/my_app.css']
+ app_include_css = ['my_app.bundle.css']
```

If you included the assets manually by explicitly writing the script tag in HTML files then you need to do the following changes:
```diff
// for js files
- <script src="/assets/js/my_app.js" type="text/javascript">
+ {{ include_script('my_app.bundle.js') }}

// for css files
- <link href="/assets/css/my_app.css" rel="stylesheet">
+ {{ include_style('my_app.bundle.css') }}
```

If you were lazy loading assets using `frappe.require` you need to do the following changes:

```diff
- frappe.require('/assets/js/another_file.js', ...)
+ frappe.require('another_file.bundle.js', ...)
```

You can test if your bundles are being compiled by running the bench build command for your app:

```sh
bench build --app my_app
```

Finally, you can delete the `build.json` file, you no longer need it.

### Website routing and rendering refactor

There was [a major refractor](https://github.com/frappe/frappe/pull/12334) done for website routing and rendering. During this refactor, few methods were moved to different files. You might have to change the following code in your custom app.

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

