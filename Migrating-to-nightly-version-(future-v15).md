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


As a good practice, always pin dependencies you heavily depend on in your apps. 

### Migrate to vue3

If your custom app is using `vue 2`, `vuex 3`, `vue-router 2` or `vuedraggable 2.24.3` you will have to migrate the code to support `vue 3`, `vuex 4.0.2`, `vue-router 4.1.5` and `vuedraggable 4.1.0`

I have mentioned some points that will help you migrate check the description of PR [#18247](https://github.com/frappe/frappe/pull/18247)

