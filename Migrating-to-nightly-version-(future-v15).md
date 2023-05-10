This page is intended to make it easier for users who maintain custom apps/forks to migrate their installations to Version 15. 

---

### Lazy loading images on website

Browsers now support the lazy loading of images natively. If you were using Framework's lazy loading trick you can simply replace it with native lazy loading. 

```diff
- <div class="website-image-lazy" data-class="img-class" data-src="image.jpg" data-alt="image"></div>
+ <img class="img-class" src="image.jpg" alt="image" loading="lazy"></img>
``` 

### Return type of various date utilities. 


- `get_year_ending` used to return string date instead of `datetime.date`. This behavior is now made consistent with other utilities.
- `get_timespan_date_range` used to return string date tuples for some cases, now it always returns `datetime.date` tuples.


### No default index on `modified` field in child tables

Frappe v15 will drop the default index on `modified` field because it was rarely used. 

If your queries require an index on the `modified` field you should selectively add it on your doctypes. 


### Dropped python dependencies

We drop dependencies which aren't used anymore. This means if you were indirectly importing them in your app they will start breaking. 

Following python dependencies are removed:

- `googlemaps`
- `urllib3`
- `gitdb`
- `pyasn1`
- `pypng`
- `google-auth-httplib2`
- `pyOpenSSL`
- `schedule`


As a good practice, always pin dependencies you heavily depend on in your apps. 

### Migrate to vue3

If your custom app is using `vue 2`, `vuex 3`, `vue-router 2` or `vuedraggable 2.24.3` you will have to migrate the code to support `vue 3`, `vuex 4.0.2`, `vue-router 4.1.5` and `vuedraggable 4.1.0`

I have mentioned some points that will help you migrate check the description of PR [#18247](https://github.com/frappe/frappe/pull/18247)



### Removed database methods

Following unused functionality/methods are removed from `frappe.db`. 

- `db.sql` - `as_utf8` parameters is not supported anymore.
- `db.sql` - `formatted` parameter is not supported anymore.
- `db.set_value` - `for_update` parameter is not removed and not required anymore as updates happen in single query.
- `db.set` - Use `doc.db_set` instead.
- `db.touch` - This method is removed.
- `db.clear_table` - Use `db.truncate` instead
- `db.update` - Use `db.set_value` instead
- `db.set_temp` & `db.get_temp` - These methods are removed

### Validate from and to dates

With `doc.validate_from_to_dates(from_date_field: str, to_date_field: str)` you can validate that the date value of one field is before the date value of another field.

Previously, if either date was missing, we compared the other field with the current date. For example, if `from_date` was not set, but `to_date` was, then we validated that `to_date` was in the future and threw an error if not.

Now, if either date field is empty, we don't validate anything.

### Deprecated support for `device` in HTTP sessions

Since support for Cordova was dropped before a few major releases, there's no need to differentiate between `mobile` and `desktop` sessions anymore. Consequently, the PR [#18729](https://github.com/frappe/frappe/pull/18729) drops support for this from the internal Sessions API. In other words, specifying the `device` parameter as `mobile` when logging into your Frappe site will not be treated differently anymore.

Additionally, the system setting for **Session Expiry Mobile** has now been removed.

### Import `compare` from utils

Previously, you were able to use `frappe.compare(val1, operator, val2)`. Now you'll have to import `compare` from `frappe.utils` to use it:

```python
from frappe.utils import compare

compare(val1, operator, val2)
```

### Setting Single DocType value using `db.set_value` is not supported 

`db.set_value` was able to set single value if doctype and docname are same or docname is `None`. This behaviour is error prone and hence we have deprecated this. Use the explicit API for setting single values instead. 

```diff
// Using None
- frappe.db.set_value("Single Doctype", None, "field", "value")
+ frappe.db.set_single_value("Single Doctype", "field", "value")

// Using same docname as doctype
- frappe.db.set_value("Single Doctype", "Single Doctype", "field", "value")
+ frappe.db.set_single_value("Single Doctype", "field", "value")
```

### Access to local scope by client scripts is no longer supported

You can no longer access local variables like `this` in your client scripts. This usage was never intended.

Further reading:
- [Example of unsupported usage](https://github.com/frappe/frappe/blob/94398aab0ebf850ec6a418346af4b4e4434715fc/frappe/email/doctype/notification/notification.js#L4:L12)
- [Pull Request](https://github.com/frappe/frappe/pull/19882)
- [MDN Article](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval#never_use_eval!)


### Renamed timezone utils

```diff
- from frappe.utils.data import convert_utc_to_user_timezone, get_time_zone
+ from frappe.utils.data import convert_utc_to_system_timezone, get_system_timezone
```

- [Deprecation PR](https://github.com/frappe/frappe/pull/20253)
- [Removal PR](https://github.com/frappe/frappe/pull/20255)


### Deprecation of `job_name` parameter in `frappe.enqueue`

To deduplicate jobs Frappe now uses RQ Job's `job_id` parameter, if you were using `job_name` to identify if duplicate job exists you should change code to use `job_id` instead.

```diff
- from frappe.core.page.background_jobs.background_jobs import get_info

- enqueued_jobs = [d.get("job_name") for d in get_info()]
- if self.name not in enqueued_jobs:
- 	enqueue(..., job_name=self.name)


+ from frappe.utils.background_jobs import enqueue, is_job_enqueued

+ job_id = f"data_import::{self.name}"
+ if not is_job_enqueued(job_id):
+ 	enqueue(..., job_id=job_id)
```
