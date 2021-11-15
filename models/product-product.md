# Model: Product (Variant & Template)

[Field Mapping](product-product.jsonc)

*Note*: Odoo handles products in an unintuitive way, with two separate models:
 1. `product.template` (template) is a 'template config' for products, and is shown on the Odoo web UI. This table stores most of the product information
 2. `product.product` (variant) is an 'implementation' for a single product, and is the model that must be used in `sale.order` and `mrp.bom` records.

When creating a new product, you must create a `product.template`, Odoo will automatically create a corresponding `product.product`. You must then query the template record to retrieve the variant's Odoo id.

<br />

## Example

```javascript
// step #1: create the template
var template_id = await odoo_create(
    'product.template',
    {
        // make sure to set the BOAhub doc id
        'boahub_id': 'XXXX-XXXX-XXXX',

        // the user-friendly name
        'name': 'Some Product',

        // the reference code
        'default_code': 'AB0202-04',

        // MRP products are only sold, not purchased
        'sale_ok': true,
        'purchase_ok': false,

        // this is a storable product (e.g. tracks stock)
        'type': 'product',

        // the sale price
        'list_price': 99.99,


        // static fields
        'product_categ_id': 1,

        // select the desired routes
        'route_ids': [
            // regular product - ship from stock
            [4, 1, false],

            // manufactured product - trigger an MO
            [4, 1, false],
        ],
    }
);

// step #2: retrieve the variant id
var product_id;
var result = await odoo_read(
    'product.template',
    [template_id],
    [
        'product_variant_id',
    ]
);

// note: result returns a list like [{'id': <product-id>, <your-fields>}]
// for example, result would be:
result = [
    {
        'id': 123, // e.g. `template_id`,
        'product_variant_id': 155,
    }
];

// because we called odoo_read() with only 1 template_id, we can just
// access element #1
if (result && result[0]) {
    product_id = result[0]['product_variant_id'];
}
```

<br />

## Callbacks

Odoo will post data back to BOAhub when some events/updates occur.

### `product_updated`

Called when the product is updated (e.g. the name, barcode, etc)
