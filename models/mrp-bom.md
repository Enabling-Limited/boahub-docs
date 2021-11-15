# Model: Bill of Materials

[Field Mapping](mrp-bom.jsonc)

*Note*: This model only handles creating the BoM, but does not implicitly mean that the linked product will be manufactured. To force the product to be manufactured, you should use the `route_ids` field on `product.template`. See the Product guide for more info.

<br />

## Example

```javascript
var bom_id = await odoo_create(
    'mrp.bom',
    {
        // make sure to set the BOAhub doc id
        'boahub_id': 'XXXX-XXXX-XXXX',

        // the product to be created
        // NOTE: this is the *template* id (e.g. the top-level `product.template`, not the `product.product`)
        'product_tmpl_id': 123,

        // how many products are manufactured (normally 1)
        'product_qty': 1,

        // bom reference
        // optional
        'code': 'This is a reference',

        // this is a static value (set to "Manufacture" not "Kit")
        'type': 'normal',

        // add the BoM lines
        'bom_line_ids': [
            [false, false, {
                // the component product id
                // NOTE: this is the *variant* id (e.g. `product.product`)
                'product_id': 123,

                // quantity to be consumed
                'product_qty': 5,
            }],
            [false, false, {
                'product': 456,
                'product_qty': 100,
            }],
        ]
    }
);
```

<br />

## Callbacks

This model does not require any callbacks, as the BoMs will only be added from BOAhub.
