# Vinyl for WordPress on Wodby

What this service adds to the Vinyl service it is based on.

## WordPress preset

`VARNISH_CONFIG_PRESET` is set to `wordpress`, so the image renders its WordPress rules into `/etc/varnish/preset.vcl`:

- URLs containing `wp-login`, `wp-admin`, `preview=true` or `xmlrpc.php` go to the backend uncached. When the admin area is on its own host, set `VARNISH_WP_ADMIN_SUBDOMAIN`: requests to a matching host are then passed instead, and the URL rule is not applied.
- Requests with an `ak_action` or `app-download` query parameter are passed (Jetpack mobile).
- A `replytocom` query parameter is removed.
- Only cookies matching `VARNISH_WP_PRESERVED_COOKIES` are kept (by default the PHP session, WordPress login and post-password cookies and the WooCommerce cart and session cookies); all others are removed before the cache lookup. A request that still has one of them is not cached, so signed-in users and visitors with a cart always get pages from WordPress. `VARNISH_KEEP_ALL_COOKIES` does not change this.

The preset template is the `wordpress` config of the manifest (`/etc/gotpl/presets/wordpress.vcl.tmpl`). Override it there, not in the rendered file.

## Purging

The preset adds no purge logic of its own; the base service's `PURGE` and `BAN` handling applies. WordPress does not purge by itself: a plugin has to send the requests to this service on port `6081`. This service does not configure WordPress.

## Check the result

- An anonymous page requested twice returns `X-VC-Cache: HIT` the second time.
- The reason for a miss is in `X-VC-Cacheable`, delivered only when the backend response carries the header `X-VC-Debug: true`.
