# Model: Stock Picking

[Field Mapping](stock-picking.jsonc)

*Note*: This model is not created directly by BOAhub (rather it is created automatically when the `sale.order` is confirmed). As such, the create endpoint is not exposed.

<br />

## Callbacks

Odoo will post data back to BOAhub when some events/updates occur.

### `picking_updated`

Called when the picking is updated (e.g. validated). Note this will perform an upsert to re-sync BOAhub (e.g. in case the picking was backordered, so we need to update the original picking, and create a new one).

Note this callback is also documented in the Sale Order doc.
