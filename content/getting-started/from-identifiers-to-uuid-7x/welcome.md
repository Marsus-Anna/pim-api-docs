# From product identifier to product UUID

::: warning
This guide only concerns Serenity users.
:::

::: info
The goal of this guide is to help you to move from a product-identifier-based logic to a UUID one.
:::

## What will happen?

Soon (summer 2022), we will deploy **6** new API endpoints, endpoints that have the same role as already-existing ones:
- `GET /api/rest/v1/products-uuid`, same as [GET /api/rest/v1/products](https://api.akeneo.com/api-reference.html#get_products)
- `POST /api/rest/v1/products-uuid`, same as [GET /api/rest/v1/products](https://api.akeneo.com/api-reference.html#post_products)
- `PATCH /api/rest/v1/products-uuid`, same as [PATCH /api/rest/v1/products](https://api.akeneo.com/api-reference.html#patch_products)
- `GET /api/rest/v1/products-uuid/{uuid}`, same as [GET /api/rest/v1/products/{code}](https://api.akeneo.com/api-reference.html#get_products__code_)
- `PATCH /api/rest/v1/products-uuid/{uuid}`, same as [PATCH /api/rest/v1/products/{code}](https://api.akeneo.com/api-reference.html#patch_products__code_)
- `DELETE /api/rest/v1/products-uuid/{uuid}`, same as [DELETE /api/rest/v1/products/{code}](https://api.akeneo.com/api-reference.html#delete_products__code_)

And later (during the last quarter of 2022), we will make optional the product identifier value (`pim_catalog_identifier` attribute).

## Why do we do that?

At the time these lines are written (July 2022), a PIM product contains one and only one way of unique identification: the so-called field `identifier` (the only `pim_catalog_identifier` attribute of the whole product).
In Serenity, this field value is the SKU (Stock Keeping Unit) of the product, but what if you need to identify your product with several product identifiers (SKU, EAN, GTIN,...)?
Adding classic fields won't do the job: you need a kind of identifier field for each product.
And how will you identify your product if its SKU has changed?

That's the purpose of the brand-new product UUID feature.

But before making it happen, a product must have a **unique** and **immutable** way to identify it: that's why we introduce the product UUID (for Universally Unique Identifier).

## What are the impacts?

Of course, [Products endpoints](https://api.akeneo.com/api-reference.html#Product) will remain available (and they be enriched with a `UUID` property), even when new API endpoints will be ready for use.
Nevertheless, when the current product identifier will become optional:
- [GET /api/rest/v1/products](https://api.akeneo.com/api-reference.html#get_products) won’t return products with empty product identifiers (in other words, you may miss products if you continue to use this endpoint);
- `associations` property for [GET /api/rest/v1/products](https://api.akeneo.com/api-reference.html#get_products) or [GET /api/rest/v1/products/{code}](https://api.akeneo.com/api-reference.html#get_products__code_) may contain **NULL** values (product associated with a product without identifier);
-  [GET /api/rest/v1/products/{code}](https://api.akeneo.com/api-reference.html#get_products__code_) could result in a 404 error (if the identifier is removed from the product).
  ...and you could encounter some issues with your code.

## What do we advise you?

As soon as product UUID API endpoints are released, jump on the bandwagon right away:
- Change [product-identifier-based products endpoints](https://api.akeneo.com/api-reference.html#Product) with UUID ones.
- Every time you identify a product with its identifier in your application, be sure to replace it by corresponding product UUID. You will easily find the match between product identifier and UUID in both [products endpoints](https://api.akeneo.com/api-reference.html#Product) and new ones.

## An example of what you may have to do?

Let's imagine you developed a product-identifier-based solution that synchronizes PIM products to a random eCommerce solution.

You persist the correlation between PIM products and eCommerce ones in a correlation table.

```code
    TABLE correlation {
      akeneo_identifier,
      ecommerce_identifier
  }
  ```

Today, its running is as follows:

```code
  Get all Akeneo products with GET /api/rest/v1/products
  For each retrieved products:
    identifier = identifier of the product
    Get the ecommerce_identifier matching identifier by querying the correlation table
    if ecommerce_identifier was not found
      Add the product to the eCommerce
      Add (identifier, ecommerce_identifier) in the correlation table
    else
      Update the product identified by ecommerce_identifier in the eCommerce
  ```

How do you turn your application into a product-UUID one?

Firstly, replace the way of getting Akeneo products: from `GET /api/rest/v1/products` to `GET /api/rest/v1/products-uuid`.
Then, add a `UUID` column to your correlation table.

```code
    TABLE correlation {
      akeneo_identifier,
      akeneo_uuid,
      ecommerce_identifier
  }
  ```

And at last, step by step, replace `akeneo_identifier` in your correlation table with `uuid`

```code
    Get all Akeneo products with GET /api/rest/v1/products-uuid
    For each retrieved products:
        identifier = identifier of the product
        uuid = uuid of the product
        Get the ecommerce_identifier matching uuid in the correlation table
        if ecommerce_identifier was not found
            Get the ecommerce_identifier matching identifier in the correlation table
            if ecommerce_identifier was found
                Add uuid and identifier in the correlation table
                Update the product in the eCommerce
            else
                Add the product to the eCommerce
                Add (identifier, uuid, ecommerce_identifier) in the correlation table
        else
            Update the product identified by ecommerce_identifier in the eCommerce
  ```

::: info
If you need help, don't hesitate to contact us!
:::
