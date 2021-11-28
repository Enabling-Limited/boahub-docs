# Model: Stock Picking

[Field Mapping](stock-picking.jsonc)

*Note*: This model is not created directly by BOAhub (rather it is created automatically when the `sale.order` is confirmed). As such, the create endpoint is not exposed.

<br />

## Callbacks

Odoo will post data back to BOAhub when some events/updates occur.

### `picking_updated`

Called when the picking is updated (e.g. validated). Note this will perform an upsert to re-sync BOAhub (e.g. in case the picking was backordered, so we need to update the original picking, and create a new one).

Note this callback is also documented in the Sale Order doc.


## Example Odoo -> BOAhub Payload

```jsonc
[
    {
        "id": 123, // Odoo id
        "boahub_portal_url": "https://odoo.boa.co.nz:8069/my/deliveries?token=abc123def456ghi678",
        "name": "AKLSI/OUT/001234",
        "date": "2021-11-28T20:43:00Z", // note: in UTC
        "date_done": false,
        "carrier_tracking_url": "track.gosweetspot.co.nz/abcdef123456",
        "partner_id": 123,
        "status": "waiting",

        "lines": [
            {
                "product_id": 1,
                "description": "A04",
                "quantity": 200.00,
                "uom": "Unit", // e.g. or "Meter"
            },
            {
                "product_id": 2,
                "description": "A06",
                "quantity": 100.00,
                "uom": "Unit(s)",
            },
        ]
    }
]
```