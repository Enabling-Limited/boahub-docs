# Model: Pricelist

[Field Mapping](product-pricelist.jsonc)

*Note*: This model is not created directly by BOAhub. As such, the create endpoint is not exposed.

<br />

## Callbacks

Odoo will post data back to BOAhub when some events/updates occur.

### `pricelist_updated`

Called when the pricelist is updated (e.g. via the Odoo UI). This will trigger an update/delete command so only affected records are updated.

### `pricelist_update_cron`

Called on a schedule (at 1:00am per day). This will trigger a *full* update of *all* pricelists, to ensure they are in sync regarding time-based rules.
