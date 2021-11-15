# Model: Partner

[Field Mapping](res-partner.jsonc)

*Note*: The `res.partner` model is not state-based, so only simply `create` and `update` endpoints are exposed.

<br />

## Example

**Creating a new `res.partner`**
```javascript
// creating a new company
var company_id = await odoo_create(
    'res.partner',
    {
        // COMMON FIELDS
        // make sure to set the BOAhub doc id
        'boahub_id': 'XXXX-XXXX-XXXX',

        'name': 'Some Company Ltd',

        // set the odoo company (database) this contact is
        // linked with
        'company_id': 1, // odoo id of a `res.company` record

        // address fields
        // note: this is common to all res.partner types (e.g. delivery address, branch, etc)
        'street': 'Address line 1',
        'street2': 'Address line 2',
        'suburb': 'Suburb',
        'city': 'City',
        'state_id': 1234, // odoo id of a `res.country.state` record
        'zip': '1234', // postcode
        'country_id': 1, // odoo id of a `res.country` record

        'phone': '07 123 4567',
        'mobile': '021 123 4567',
        'email': 'email@addre.ss',
        'website': 'https://www.somecompany.co.nz',



        // COMPANY-SPECIFIC FIELDS
        // these will both be static values
        'company_type': 'company',
        'property_payment_term_id': 1,
        'property_account_position_id': 1,

        // set the customers' pricelist
        'pricelist_id': 1234,
    }
);

// creating a branch
var branch_id = await odoo_create(
    'res.partner',
    {
        // add common fields here


        // BRANCH-SPECIFIC FIELDS

        // make sure to fill parent_id
        'parent_id': company_id,

        // set to 'company' type
        'company_type': 'company',

        // set the branch pricelist
        'pricelist_id': 1234,
    }
);

// creating a new 'contect'/'user'
var contact_id = await odoo_create(
    'res.partner',
    {
        // add common fields here


        // CONTACT-SPECIFIC FIELDS

        // contact must be linked with a parent company/branch
        'parent_id': branch_id,
        'parent_id': company_id,

        // set as 'individual' type
        'company_type': 'person',

        // set the contact type (choose one)
        'type': 'contact',  // standard contact
        'type': 'invoice',  // invoice address (will be used as default for 'partner_invoice_id')
        'type': 'delivery', // delivery address (will be used as default for 'partner_shipping_id')
        'type': 'other',    // misc address
        'type': 'private',  // private address (only accessible to some Odoo users)
    }
);
```

**Updating an existing `res.partner`**
```javascript
// note, you can operate on multiple records
var partner_ids = [123];
var success = await odoo_update(
    'res.partner',
    partner_ids,
    {
        // one or more fields from the res-partner.jsonc fields list
    }
);
```

<br />

## Callbacks

Odoo will post data back to BOAhub when some events/updates occur.

### `partner_updated`

Called when a partner (who is being tracked by BOAhub) is updated.


### `subcontact_created`

Called when a sub-contact is added to an existing BOAhub contact (e.g. a new delivery address)
