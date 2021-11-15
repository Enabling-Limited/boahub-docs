# BOAhub to Odoo Integration

This document outlines the Odoo model schema/fields, along with descriptions on the flow when creating records via the XML-RPC API.


## Models

Click the links below to see details for a particular model.

 * [Sale Order](models/sale-order.md)
 * [Product](models/sale-order.md)
 * [Bill of Materials](models/mrp-bom.md)
 * [Pricelist](models/sale-order.md)
 * [Partner (Company, Branch, User)](models/sale-order.md)
 * [Stock Picking](models/sale-order.md)


## Odoo XML-RPC API

Note, I'm assuming you have an XML-RPC wrapper similar to:

```javascript
var xmlrpc = require('xmlrpc');
const ODOO_HOST = 'testodoo.boa.co.nz';
const ODOO_PORT = 8069;
const ODOO_DB = 'BOAHub_Odoo';
const ODOO_UID = 2;
const ODOO_APIKEY = 'xxxx';

// XMLRPC client - this is immutable, so it can be reused (probably thread-safe too)
// note: use createSecureClient so that TLS is used
const odoo = xmlrpc.createSecureClient({
    host: ODOO_HOST,
    port: ODOO_PORT,
    path: '/xmlrpc/2/object',
});

// helper functions - might be worth putting these in a class
async function odoo_exec(model, method, args, kwargs) {
    args = args || [];
    kwargs = kwargs || {};
    return new Promise((resolve, reject) => {
        odoo.methodCall(
            // 'execute_kw' is always the method
            'execute_kw',

            // XMLRPC args - these are the args shown in the
            // Python reference in the docs
            [
                // required
                ODOO_DB, ODOO_UID, ODOO_APIKEY,

                // args should be an array of positional args
                model,
                method,
                args,
                kwargs,
            ],

            // this is called when the request returns
            (err, val) => {
                if (err) {
                    reject(err);
                }
                else {
                    resolve(val);
                }
            }
        );
    });
}
async function odoo_search(model, domain, kwargs) {
    kwargs_skip_sync(kwargs);
    return odoo_exec(model, 'search', [domain], kwarg);
}
async function odoo_read(model, ids, fields, kwarg) {
    kwargs_skip_sync(kwargs);
    return odoo_exec(model, 'read', [ids, fields], kwarg);
}
async function odoo_create(model, values, kwarg) {
    kwargs_skip_sync(kwargs);
    if (typeof(values) != Array) {
        values = [values];
    }
    return odoo_exec(model, 'create', values, kwarg);
}
async function odoo_update(model, ids, new_values, kwarg) {
    kwargs_skip_sync(kwargs);
    return odoo_exec(model, 'write', [ids, new_values], kwarg);
}
async function odoo_call(model, method, args, kwargs) {
    kwargs_skip_sync(kwargs);
    return odoo_exec(mode, method, args, kwargs);
}

function kwargs_skip_sync(kwargs) {
    // inject context key to the kwargs variable
    // tihs key will ensure that update calls on any records
    // do not trigger a sync back to BOAhub (because they
    // came from BOAhub..)
    kwargs = kwargs || {};
    if !kwargs['context'] {
        kwargs['context'] = {}
    }
    kwargs['context']['boahub_skip_sync'] = true;
    return kwargs
}
```

### `create`

The `create` call will create a new record, and return its Odoo `id` (a 32-bit integer).

Example usage:
```javascript
var sale_order_id = await odoo_create(
    // must pass the model name
    'sale.order',

    // then the values we want to use for the created record
    {
        'partner_id': 123,
        'boahub_id': 'XXXX-XXXX-XXXX',

        // ...etc
    }
);
```

<br />

It's also possible to create multiple records with one API call:
```javascript
var sale_order_ids = await odoo_create(
    'sale.order',

    // pass array of order values
    [
        // order #1
        {
            'partner_id': 123,
            'boahub_id': 'XXXX-XXXX-XXXX',
        },

        // order #2
        {
            'partner_id': 456,
            'boahub_id': 'YYYY-YYYY-YYYY',
        }
    ]
);
```

<br />

Relational fields can be set either via their ID (for many2one) or via a special update syntax (for many2many or one2many).

```javascript
var sale_order_id = await odoo_create(
    'sale.order',
    {
        // set a many2one via id
        'partner_id': 123,

        // set a one2many or many2many via a set of ids
        //
        // note this uses the special 'update syntax'
        // ref: https://www.odoo.com/documentation/14.0/developer/reference/addons/orm.html#odoo.models.Model.write
        //
        // you should pass an array of arrays, where each sub-array is one of the commands
        // from the link above.

        // method #1: pass ids of already-created records
        'tag_ids': [
            // [4, ID, false] will add a new record to the list
            [4, 123, false],
            [4, 456, false],
            [4, 789, false],

            // [6, false, IDS] will replace the list with a new set of records from `IDS`
            [6, false, [123, 456, 789]],
        ]

        // method #2: create records on the fly (pass values, rather than ids)
        'order_line': [
            // [false, false, VALUES] will create a new record, and add it to the list
            [false, false, {
                'product_id': 123,
                'description': 'This is a sale order line',
            }],
            [false, false, {
                'product_id': 456,
                'description': 'This is another sale order line',
            }]
        ]
    }
);
```