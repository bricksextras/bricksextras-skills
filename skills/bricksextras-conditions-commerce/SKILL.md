---
name: bricksextras-conditions-commerce
description: "Use when adding a BricksExtras commerce or membership-plugin element display condition (_conditions): WooCommerce cart contents/total/weight, product purchased/in-cart/in-stock/type, coupon-applied checks, or membership status from EDD, MemberPress, Restrict Content Pro, WishList Member, SureMembers, Paid Memberships Pro, WooCommerce Subscriptions/Memberships, or FluentCart. Covers the 'current product' subfamily's context requirement, 'purchased' vs 'just purchased' distinction, and where compare doubles as a status filter. For general (non-commerce) x_* conditions, see the separate bricksextras-conditions skill."
---

**Requires:** BricksExtras 1.7.3+, Bricks 2.4+, plus whichever commerce/membership plugin a given condition targets

# BricksExtras: Commerce & membership conditions (`x_*`)

Same underlying mechanism as `bricksextras-conditions` (real Bricks filters; MCP-writable as a normal `_conditions` element setting — see that skill for per-connection validation behavior) — this skill covers the value-format specifics of the WooCommerce/cart/product group and the membership-plugin group (EDD, MemberPress, RCP, WishList Member, SureMembers, PMP, WooCommerce Subscriptions/Memberships, FluentCart).

## General failure mode: fails open when the integration is absent, real result once it's active

Nearly every condition in this group returns `true` (condition considered met — i.e. the element still renders) when its required plugin class/function isn't active, or when `value` is empty — never a hard error, and never silently `false`. Once the integration is confirmed active and a real value is supplied, the condition evaluates for real and can genuinely return `false`.

**Exceptions that fail closed instead** (return `false`, not `true`, even when nothing is obviously "wrong"):
- `x_product_in_cart`/`x_product_in_cart_id` — returns `false` outright if the given product ID doesn't resolve to a real product at all (`wc_get_product()` fails), regardless of `compare`.
- `x_product_in_cart_has_coupon`/`x_product_in_cart_has_coupon_id` — returns `false` whenever WooCommerce/value/cart/applied-coupons are missing, the opposite default from the rest of the file.
- `x_user_has_order_with_status` — returns `false` (not `true`) when WooCommerce or the current user is unavailable.

## The "current product" subfamily requires being on/inside an actual product

`x_product_is_virtual`, `x_product_is_downloadable`, `x_product_in_stock`, `x_product_weight`, `x_product_rating`, `x_product_type`, `x_product_on_backorder`, `x_product_backorders_allowed`, `x_product_upsell_count`, `x_product_crosssells_count`, and `x_current_product_in_cart` all read from **the current post** (`get_the_ID()`), not from a `value`-supplied product ID. Each explicitly returns `false` — not the usual fail-open `true` — if the current post's type isn't `product`. Use these only on a single product page or inside a product query-loop item; elsewhere they always evaluate false regardless of `compare`/`value`.

By contrast, `x_product_in_cart`/`_id`, `x_product_has_category`/`_has_tag`, `x_user_purchased_product`/`_id`, and the coupon conditions take an explicit product ID/term in `value` and work from any page context.

## "Purchased" vs "just purchased" — genuinely different checks, not a naming variant

- **`x_user_purchased_product`/`x_user_purchased_product_id`** — a general purchase-history check via `wc_customer_bought_product()`. Works on any page, checks the current user's entire order history for that product.
- **`x_user_just_purchased_product`** — **only evaluates meaningfully on the WooCommerce order-received/thank-you URL** (`is_wc_endpoint_url('order-received')`) or a URL carrying `?order_id=&order_key=` query params matching a real order. It reads that specific just-completed order's line items, not the user's full purchase history. Placed anywhere else, there's no order context to read and it returns `true` (fail-open — no order found).
- **`x_fluentcart_user_purchased_product`/`_id`** — general FluentCart purchase-history check (via the customer's successful order items), works anywhere.
- **`x_user_just_purchased_fluentcart_product`/`_id`** — same page-context dependency as the WooCommerce "just purchased" variant, but keyed off a `?trx_hash=` query parameter from a FluentCart checkout redirect instead of WooCommerce's own endpoint.

Don't substitute the "just purchased" variants for a general "has this user ever bought X" check — they're specifically for thank-you-page-style "you just bought this" messaging.

## `x_product_has_category`/`x_product_has_tag` accept term ID or slug — unlike the general post-category condition

Both use `has_term($value, 'product_cat'/'product_tag')`, which accepts either a numeric term ID or a slug string. This is looser than the general (non-WooCommerce) `x_post_category`/`x_post_tag` conditions covered in `bricksextras-conditions`, which strictly `intval()` the value and only work with numeric term IDs — don't assume the same value format applies across both condition families just because they sound similar.

## Membership status conditions: `compare` sometimes doubles as a status filter, not a pure operator

For `x_rcp_membership_level`, `x_wishlist_member`, and `x_fluentcart_user_subscription`/`_id`, `compare` is `==`/`!=` **only when checking plain membership**; any other value passed as `compare` (e.g. `"active"`, `"cancelled"`, `"expired"`, `"pending"`) is instead used as a literal status filter in the underlying query/lookup. This is a dual-purpose field, not a strict comparison operator — check the condition's own registered `compare` options (visible via `condition_options()`/the real Builder UI) before assuming `==`/`!=` are the only valid values.

## `x_pmp_membership_level`'s `"non-members"` value is a sentinel, not a real level

Setting `value: "non-members"` checks whether the current user has **no** active Paid Memberships Pro level at all (`! pmpro_hasMembershipLevel()`), rather than checking membership in a level literally named "non-members". Any other `value` is treated as a real level ID/name passed to `pmpro_hasMembershipLevel($value)`.

## Straightforward boolean/numeric checks (no special value-format gotchas)

`x_edd_subscribed`/`_id`, `x_edd_product_purchased`/`_id`, `x_memberpress_membership`/`_id`, `x_wp_members_membership`/`_id`, `x_sure_members_access_groups`/`_id`, `x_woocommerce_subscriptions`/`_id`, `x_woocommerce_memberships`/`_id`, `x_rcp_has_active_membership`, `x_rcp_has_paid_membership`, `x_cart_empty`, `x_cart_total`, `x_cart_total_minus_shipping`, `x_cart_weight`, `x_cart_count` — all take a plain ID (membership/product) or numeric `value` with the standard `==`/`!=`/`>=`/`<=`/`>`/`<` operator set, no reformatting or dual-purpose fields.

## Build workflow

1. Confirm the target commerce/membership plugin is actually active on this site — every condition here silently no-ops (fails open) rather than erroring if it isn't, which can look like "the condition just doesn't work" during testing.
2. For the "current product" subfamily, place the element on a real product page/loop item — elsewhere they always evaluate false.
3. Pick "purchased" vs "just purchased" deliberately based on whether the check should look at full order history or only a just-completed order/redirect context.
4. For RCP/WishList Member/FluentCart subscription status checks, check that condition's own `compare` options before assuming only `==`/`!=` are valid — many accept a status string instead.
