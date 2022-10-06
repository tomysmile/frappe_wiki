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


### `update_modified` behaviour in `doc.db_set` and `frappe.db.set_value`

Until version 14 `doc.db_set` or `frappe.db.set_value` used to update `modified` timestamp by default. This behaviour is now disabled by default.

If you need to update modified timestamp, you have to pass argument explicitly. Example:

```python
frappe.db.set_value("Item", "book", "is_stock_item", 1, update_modified=True)
```

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


As a good practice, always pin dependencies you heavily depend on in your apps. 

