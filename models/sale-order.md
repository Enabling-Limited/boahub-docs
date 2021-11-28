# Model: Sale Order

[Field Mapping](sale-order.jsonc)

<br />

## States

The `sale.order` model has several states, stored in the `state` field
 * `draft`: This is a Quotation (default)
 * `sent`: This is a Quotation, and a 'confirmation' email has been sent
 * `sale`: This is a confirmed Sale Order
 * `done`: This Order is complete (pickings delivered & invoice paid)
 * `cancel`: This is a cancelled Order/Quotation

To create an SO, the following process should be used:
 1. Create the SO in BOAhub, and assign it a doc id
 2. Call the `create` XML-RPC method (returns the Odoo ids)
 3. Call the `boahub_confirm` method to confirm the SO + generate pickings (returns details about the pickings)
 4. Update the SO in BOAhub with the returned picking details

<br />

## Example

```javascript
// step #1: create the sale order
var sale_order_id = await odoo_call(
    'sale.order',
    'boahub_create',
    {
        // make sure to set the BOAhub doc id
        'boahub_id': 'XXXX-XXXX-XXXX',

        // the BOAhub "order" id/ref field
        'boahub_order_id': 'ABC123',

        // top-level partner linked to the SO (e.g. the company/branch)
        'partner_id': 123,

        // delivery address (will flow through to the picking doc)
        'partner_shipping_id': 456,

        // invoice address/email (will flow through to the invoice)
        'partner_invoice_id': 789,

        // a client order ref. this is shown on the portal, and the SO PDF doc
        // optional
        'client_order_ref': 'Some Reference',

        // an internal order ref. this is shown on the portal, and the SO PDF doc
        // optional
        'name': 'Internal Reference'

        // the order date. if left blank, it will default to now()
        // note: this is ISO8601 format, and *must* be passed in UTC
        // optional
        'date_order': '2021-01-01T12:30:00Z' // 12:30pm in UTC

        // sales user/team. these will be hardcoded once the records are set up
        'user_id': 123,
        'team_id': 123,

        // which company should the SO be linked to?
        // refer to the `country-company-mapping.jsonc` doc
        'company_id': 1,

        // order lines
        'order_line': [
            [false, false, {
                // the product (product.product, *not* product.template)
                'product_id': 123,

                // the description. if left blank, Odoo will default this to the product
                // name. it's show on the portal and PDF doc
                // optional
                'description': 'An Order Line',

                // how many products are being sold?
                'product_uom_qty': 100,


                // optionally, you can override the unit price or discount with the
                // fields below. note that Odoo will auto-fill the unit price based
                // on the customer pricelist, so it isn't mandatory to set these
                'price_unit': 100.00,
                'discount': 50.00,
            }],

            // another example
            [false, false, {
                'product_id': 456,
                'product_uom_qty': 10,
                'discount': 25.00,
            }],
        ]
    }
);

// step #2: confirm the sale order
var result = await odoo_call(
    'sale.order',
    'boahub_confirm',

    // boahub_confirm expects a lists of ids
    [
        [sale_order_id]
    ],
);

// boahub_confirm() returns a result like:
// the schema for the returned 'pickings' and 'invoices' dict is documented
// elsewhere.
result = {
    // 1234 is the SO id
    1234: {
        'pickings': [
            // list of picking values (could be multiple)
            {
                'id': 123,
                'name'; 'AKLSI/OUT/1234',
                // ...etc
            },
        ],
        'invoices': [
            // as above - list of invoices
            {
                // ...
            }
        ]
    },

    // 5678 is the SO id
    5678: {
        'pickings': [{}. {}],
        'invoices': [{}, {}],
    }
}
```

<br />

## Callbacks

Odoo will post data back to BOAhub when some events/updates occur.

### `picking_updated`

Called when the picking is updated (e.g. validated). Note this will perform an upsert to re-sync BOAhub (e.g. in case the picking was backordered, so we need to update the original picking, and create a new one).

Note this callback is also documented in the Picking doc.

### `invoice_update`

Called when the invoice is created/updated.

### `order_update`

Called when the sale order is updated (e.g. if the status changes).
