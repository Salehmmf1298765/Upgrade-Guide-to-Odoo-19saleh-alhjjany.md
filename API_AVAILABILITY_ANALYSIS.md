# API Availability Analysis (Tigro Mobile App API Spec vs Odoo 18 Standard + Tigro-Live Customization)

## Scope

This document is based on:

- `custom_18_part/Tigro-Live/tigro api.csv` (API specification received from the client).
- A scan of existing customizations under `custom_18_part/Tigro-Live` (as requested).
- A scan of Odoo standard/enterprise addons under `odoo/addons`.

`custom_18_part/Tigro_App` was intentionally ignored.

## Key findings (evidence-based)

- No implementation for the requested `/api/...` endpoints was found in the existing `Tigro-Live` customization.
  - The only matches for `/api/` inside `Tigro-Live` were in the CSV itself plus unrelated technical logs and vendor JS.
- `Tigro-Live` contains website/eCommerce related customizations (themes, gift wrap, payment gateway) but they expose different routes than the client’s `/api/...` spec.
  - Example existing JSON routes:
    - MyFatoorah: `/payment/myfatoorah/...` (see `custom_18_part/Tigro-Live/myfatoorah_gateway/controllers/main.py`).
    - Gift wrap: `/shop/cart/giftwrap` (see `custom_18_part/Tigro-Live/odoo_website_giftwrap/controllers/main.py`).

## How to read the table

- **Standard Odoo (No Customization)**:
  - Indicates whether Odoo provides the *underlying feature / business object* (models + standard flows) out-of-the-box.
  - This does **not** mean Odoo already provides the same REST endpoint.
- **Exists in Tigro-Live Custom**:
  - Indicates whether the feature/route is already implemented in `custom_18_part/Tigro-Live`.
- **Needs New API / Interface Layer**:
  - For almost all items, a dedicated API layer (new Odoo controllers) is required to expose a stable JSON contract matching the `/api/...` spec.

Legend: `Yes` / `Partial` / `No`

## Endpoint availability matrix

| Domain | API Name | Method | Endpoint | Standard Odoo (No Customization) | Exists in Tigro-Live Custom | Needs New API / Interface Layer | Notes / Evidence |
|---|---|---:|---|---|---|---|---|
| Auth | Login | POST | `/api/auth/login` | Yes | No | Yes | Odoo provides JSON login via `/web/session/authenticate` (`odoo/addons/web/controllers/session.py`). |
| Auth | Register | POST | `/api/auth/register` | Yes | No | Yes | Odoo signup exists via `auth_signup` (`odoo/addons/auth_signup`). |
| Auth | Guest Login | POST | `/api/auth/guest` | No | No | Yes | Guest *session/account* concept not provided as a standard API feature (website has a public user, but not a dedicated guest-account endpoint). |
| Auth | Social Login | POST | `/api/auth/social` | Yes | No | Yes | OAuth available via `auth_oauth` (`odoo/addons/auth_oauth`). |
| Auth | Logout | POST | `/api/auth/logout` | Yes | No | Yes | Odoo session logout exists (`/web/session/destroy`, `/web/session/logout`). |
| Auth | Refresh Token | POST | `/api/auth/refresh` | No | No | Yes | Standard Odoo is session-cookie based; no refresh-token REST endpoint found in scanned addons. |
| Auth | Forgot Password | POST | `/api/auth/forgot-password` | Yes | No | Yes | Password reset supported by `auth_signup` (standard web flow). |
| Auth | Reset Password | POST | `/api/auth/reset-password` | Yes | No | Yes | Password reset supported by `auth_signup` (standard web flow). |
| Auth | Change Password | POST | `/api/auth/change-password` | Yes | No | Yes | Password change available via portal/account flows (no matching `/api/...` endpoint). |
| Auth | Deactivate Account | POST | `/api/auth/deactivate` | No | No | Yes | Self-service deactivation is not a standard customer feature; would require custom logic (archive user/partner rules). |
| Auth | Delete Account | DELETE | `/api/auth/delete` | No | No | Yes | Self-service deletion is not standard; would require custom logic and GDPR/retention rules. |
| Auth | Verify Email/OTP | POST | `/api/auth/verify` | Partial | No | Yes | Email verification exists in signup flows; OTP verification is not standard. |
| Auth | Resend OTP | POST | `/api/auth/resend-otp` | No | No | Yes | OTP resend not found as a standard feature in scanned addons. |
| Profile | Get Profile | GET | `/api/customer/profile` | Yes | No | Yes | Customer profile is `res.partner` + portal account (`odoo/addons/portal`). |
| Profile | Update Profile | PUT | `/api/customer/profile` | Yes | No | Yes | Partner update exists via portal account/edit flows. |
| Profile | Get Preferences | GET | `/api/customer/preferences` | Partial | No | Yes | Language exists; app-specific notification preferences not found as a standard object. |
| Profile | Update Preferences | PUT | `/api/customer/preferences` | Partial | No | Yes | Same as above. |
| Address | Get Addresses | GET | `/api/address` | Yes | No | Yes | Odoo supports multiple partner addresses (delivery/invoice/other). |
| Address | Add Address | POST | `/api/address` | Yes | No | Yes | Address creation exists (partner child addresses). |
| Address | Update Address | PUT | `/api/address/{id}` | Yes | No | Yes | Address update exists. |
| Address | Delete Address | DELETE | `/api/address/{id}` | Yes | No | Yes | Address deletion exists (record rules / portal restrictions to be enforced). |
| Address | Set Default Address | POST | `/api/address/{id}/default` | Yes | No | Yes | Default shipping/billing is stored on sale order / partner; requires API logic. |
| Location | Get Countries | GET | `/api/location/countries` | Yes | No | Yes | `res.country` is standard. |
| Location | Get Cities | GET | `/api/location/cities` | Yes (if installed) | No | Yes | City model exists in `base_address_extended` (`odoo/addons/base_address_extended/models/res_city.py`). |
| Location | Get Provinces | GET | `/api/location/provinces` | Yes | No | Yes | `res.country.state` is standard. |
| Location | Get Areas | GET | `/api/location/areas` | No | No | Yes | Areas/zones are not a standard geographic model in scanned addons. |
| Catalog | Get SubCategories | GET | `/api/categories/{id}` | Yes | No | Yes | `product.public.category` (website categories) is standard in `website_sale`. |
| Catalog | Get Brands | GET | `/api/catalog/brands` | No | Yes | Yes | `Tigro-Live` provides a custom `product.brand` model (see `custom_18_part/Tigro-Live/emipro_theme_base/models/product_brand.py`). |
| Products | Search Products | GET | `/api/products/search` | Yes | No | Yes | Search exists in `website_sale` and product models; needs API layer. |
| Products | Get Product Listing | GET | `/api/products` | Yes | No | Yes | Product listing exists in `website_sale`; needs API layer. |
| Products | Get Product Details | GET | `/api/products/{id}` | Yes | No | Yes | Product detail exists; needs API layer. |
| Products | Get Related Products | GET | `/api/products/{id}/related` | Yes | No | Yes | Related products available via `accessory_product_ids` / `alternative_product_ids` (see `odoo/addons/website_sale/models/product_template.py`). |
| Products | Get Upsell Products | GET | `/api/products/{id}/upsell` | Yes | No | Yes | Upsell/cross-sell strategy supported via alternative/accessory products (standard). |
| Products | Get Featured Products | GET | `/api/products/featured` | Partial | No | Yes | Can be implemented using website publishing/sequence/ribbons; no single standard “featured” API. |
| Products | Get Flash Sale Products | GET | `/api/products/flash-sale` | No | No | Yes | Flash-sale concept not found as a standard model/feature in scanned addons. |
| Products | Check Availability | GET | `/api/products/{id}/availability` | Yes | No | Yes | Stock availability exists via stock + `website_sale_stock`. |
| Cart | Get Cart | GET | `/api/cart` | Yes | No | Yes | Standard cart is `sale.order` stored in website session (`website_sale`). |
| Cart | Add Item | POST | `/api/cart/items` | Yes | No | Yes | Website provides cart update routes like `/shop/cart/update_json` (`odoo/addons/website_sale`). |
| Cart | Update Item Qty | PUT | `/api/cart/items/{id}` | Yes | No | Yes | Same as above. |
| Cart | Remove Item | DELETE | `/api/cart/items/{id}` | Yes | No | Yes | Same as above. |
| Cart | Clear Cart | DELETE | `/api/cart` | Yes | Partial | Yes | `Tigro-Live` has `/shop/clear_cart` (see `custom_18_part/Tigro-Live/emipro_theme_base/controllers/main.py`) but not `/api/cart`. |
| Cart | Apply Coupon | POST | `/api/cart/coupon` | Yes | No | Yes | Coupons/loyalty supported via `loyalty`, `sale_loyalty`, `website_sale_loyalty` (see manifests under `odoo/addons`). |
| Cart | Remove Coupon | DELETE | `/api/cart/coupon` | Yes | No | Yes | Same as above. |
| Cart | Apply Wallet | POST | `/api/cart/wallet` | Partial | No | Yes | eWallet is part of `loyalty` (manifest mentions eWallet) but applying it in checkout requires an API integration layer. |
| Cart | Remove Wallet | DELETE | `/api/cart/wallet` | Partial | No | Yes | Same as above. |
| Cart | Check Stock | POST | `/api/cart/check-stock` | Yes | No | Yes | Stock checks exist but an explicit API endpoint is not standard. |
| Checkout | Validate Checkout | POST | `/api/checkout/validate` | Yes | No | Yes | Checkout flow exists in `website_sale`; API endpoint is custom. |
| Checkout | Get Delivery Fee | GET | `/api/checkout/delivery-fee` | Yes | No | Yes | Shipping fee calculation exists via `delivery` + `website_sale` checkout integration. |
| Checkout | Select Address | POST | `/api/checkout/address` | Yes | No | Yes | Address selection exists in checkout (website flow). |
| Checkout | Select Payment Method | POST | `/api/checkout/payment-method` | Yes | No | Yes | Payment provider selection exists in standard checkout (`payment` addons). |
| Checkout | Place Order | POST | `/api/checkout/place-order` | Yes | No | Yes | Order creation exists; API endpoint is custom. |
| Checkout | Retry Payment | POST | `/api/checkout/retry-payment` | Yes | No | Yes | Payment retry possible via standard payment transactions; API endpoint is custom. |
| Orders | Get Orders List | GET | `/api/orders` | Yes | No | Yes | Sale orders + portal access exist (`sale`, `portal`). |
| Orders | Get Order Details | GET | `/api/orders/{id}` | Yes | No | Yes | Same as above. |
| Orders | Cancel Order | POST | `/api/orders/{id}/cancel` | Partial | No | Yes | Order cancellation exists, but customer-facing cancel rules are typically custom. |
| Orders | Reorder | POST | `/api/orders/{id}/reorder` | Yes | No | Yes | Reorder functionality exists in website sale assets (`website_sale_reorder.js`). |
| Orders | Track Order | GET | `/api/orders/{id}/track` | Partial | No | Yes | Order status is standard; shipment tracking depends on carrier integration and a custom API contract. |
| Orders | Download Invoice | GET | `/api/orders/{id}/invoice` | Yes | No | Yes | Invoices are standard (`account`) and can be exposed on portal; API endpoint is custom. |
| Orders | Confirm Delivery | POST | `/api/orders/{id}/confirm` | No | No | Yes | Customer confirmation of delivery is not a standard feature. |
| Orders | Create Return | POST | `/api/orders/{id}/return` | No | No | Yes | Customer return requests are not a standard portal/API feature in scanned addons. |
| Payments | Get Payment Methods | GET | `/api/payments/methods` | Yes | Partial | Yes | Payment providers exist in `payment`; Tigro-Live includes MyFatoorah provider (`custom_18_part/Tigro-Live/myfatoorah_gateway`). |
| Payments | Initiate Payment | POST | `/api/payments/initiate` | Yes | Partial | Yes | MyFatoorah exposes JSON routes under `/payment/myfatoorah/...` but not the requested `/api/...` path. |
| Payments | Verify Payment | POST | `/api/payments/verify` | Yes | Partial | Yes | Same as above. |
| Payments | Check Payment Status | GET | `/api/payments/{id}/status` | Yes | Partial | Yes | Standard payment transactions exist; API endpoint mapping needed. |
| Delivery | Get Delivery Zones | GET | `/api/delivery/zones` | No | No | Yes | Delivery carriers exist; “zones” model/endpoint is not standard. |
| Delivery | Get Delivery ETA | GET | `/api/delivery/eta` | No | No | Yes | ETA typically depends on logistics integration; not found as standard. |
| Delivery | Live Track Delivery | GET | `/api/delivery/{orderId}/live` | No | No | Yes | Live tracking is not standard; requires external tracking integration + API. |
| Delivery | Change Delivery Slot | POST | `/api/delivery/{orderId}/slot` | No | No | Yes | Delivery slot management is not standard in scanned addons. |
| Wishlist | Get Wishlist | GET | `/api/wishlist` | Yes | No | Yes | Wishlist exists via `website_sale_wishlist` (`odoo/addons/website_sale_wishlist`). |
| Wishlist | Add to Wishlist | POST | `/api/wishlist/items` | Yes | No | Yes | Same as above. |
| Wishlist | Remove from Wishlist | DELETE | `/api/wishlist/items/{id}` | Yes | No | Yes | Same as above. |
| Wallet | Get Wallet Balance | GET | `/api/wallet` | Partial | No | Yes | eWallet concept exists in `loyalty` (manifest mentions eWallet). API needed. |
| Wallet | Get Wallet Transactions | GET | `/api/wallet/transactions` | Partial | No | Yes | Loyalty history exists; wallet transactions API is custom. |
| Wallet | Top-up Wallet | POST | `/api/wallet/topup` | No | No | Yes | Wallet top-up requires payment + business rules; not found as standard. |
| Loyalty | Get Loyalty Summary | GET | `/api/loyalty` | Yes | No | Yes | Loyalty programs exist via `loyalty` (`odoo/addons/loyalty`). |
| Loyalty | Get Points History | GET | `/api/loyalty/transactions` | Yes | No | Yes | Loyalty history exists (`loyalty_history`). |
| Loyalty | Redeem Points | POST | `/api/loyalty/redeem` | Yes | No | Yes | Redemption exists via loyalty rules/rewards; API is custom. |
| Loyalty | Get Tier Info | GET | `/api/loyalty/tier` | No | No | Yes | Tier-specific feature not found in scanned `loyalty` addon. |
| Loyalty | Get Referral Info | GET | `/api/loyalty/referral` | No | No | Yes | Referral feature not found in scanned addons. |
| Promotions | Get Available Promotions | GET | `/api/promotions` | Yes | No | Yes | Promotions/coupons exist via `loyalty` + `website_sale_loyalty`. |
| Promotions | Validate Coupon | POST | `/api/promotions/validate-coupon` | Yes | No | Yes | Coupon validation exists in website sale loyalty logic; API is custom. |
| Reviews | Get Product Reviews | GET | `/api/reviews/product/{id}` | Yes | No | Yes | Product supports `rating.mixin` (`odoo/addons/website_sale/models/product_template.py`). |
| Reviews | Add Review | POST | `/api/reviews` | Yes | No | Yes | Ratings can be created; API controller needed. |
| Reviews | Edit Review | PUT | `/api/reviews/{id}` | Partial | No | Yes | Editing user ratings is not a standard exposed operation; may require custom rules/controller. |
| Reviews | Delete Review | DELETE | `/api/reviews/{id}` | Partial | No | Yes | Same as above. |
| Reviews | Check Eligibility | GET | `/api/reviews/eligibility/{productId}` | No | No | Yes | Eligibility rules (e.g., must have purchased) are custom. |
| CMS | Get Home Content | GET | `/api/cms/home` | Yes | No | Yes | Website pages/snippets exist; API assembly for mobile home is custom. |
| CMS | Get Banners | GET | `/api/cms/banners` | Yes | No | Yes | Banners can be managed via website content; API endpoint is custom. |
| CMS | Get Page by Slug | GET | `/api/cms/pages/{slug}` | Yes | No | Yes | Website pages exist (`website`); API endpoint is custom. |
| CMS | Get FAQs | GET | `/api/cms/faqs` | Yes | No | Yes | FAQ snippets exist (see `odoo/addons/website/static/src/snippets/s_faq_*`). API endpoint is custom. |
| System | Get App Config | GET | `/api/system/config` | Partial | No | Yes | Config parameters exist (`ir.config_parameter`) but an app-config endpoint is custom. |
| System | Maintenance mode | GET | `/api/system/maintenance` | No | No | Yes | Maintenance-mode endpoint not found as standard. |
| System | Get feature flags | GET | `/api/system/flags` | No | No | Yes | Feature flags endpoint not found as standard. |
| Notifications | Get Notifications | GET | `/api/notifications` | Partial | No | Yes | Messaging exists (`mail`), but “app notifications API” is custom. |
| Notifications | Mark as Read | POST | `/api/notifications/{id}/read` | Partial | No | Yes | Would require mapping to a chosen model (mail.message / custom notification model). |
| Notifications | Mark All as Read | POST | `/api/notifications/read-all` | Partial | No | Yes | Same as above. |
| Notifications | Delete Notification | DELETE | `/api/notifications/{id}` | Partial | No | Yes | Same as above. |
| Notifications | Get Unread Count | GET | `/api/notifications/unread-count` | Partial | No | Yes | Same as above. |
| Gifts | Get Wrapping Options | GET | `/api/gifts/wrapping-options` | No | Yes | Yes | Gift wrap exists in `Tigro-Live` (`odoo_website_giftwrap`), but only exposes `/shop/cart/giftwrap` route; mobile API contract is missing. |
| Gifts | Get Card Designs | GET | `/api/gifts/card-designs` | No | No | Yes | No evidence of “gift card designs” feature in scanned custom modules. |
| Gifts | Add Gift to Cart | POST | `/api/gifts/add-to-cart` | No | Partial | Yes | Gift wrap add logic exists via `/shop/cart/giftwrap` (custom). Needs dedicated mobile API endpoints. |
| Gifts | Remove Gift from Cart | POST | `/api/gifts/remove-from-cart` | No | No | Yes | No remove endpoint found in `odoo_website_giftwrap`; would need customization. |
| Support | Create Ticket | POST | `/api/support/tickets` | Yes | No | Yes | Helpdesk exists (`odoo/addons/helpdesk`) including portal templates; API endpoints are custom. |
| Support | Get My Tickets | GET | `/api/support/tickets` | Yes | No | Yes | Same as above. |
| Support | Get Ticket Details | GET | `/api/support/tickets/{id}` | Yes | No | Yes | Same as above. |
| Support | Add Ticket Comment | POST | `/api/support/tickets/{id}/comments` | Yes | No | Yes | Ticket chatter exists (`mail`), but API endpoint is custom. |
| Analytics | Log Event | POST | `/api/analytics/events` | No | No | Yes | Frontend event logging endpoint not found as standard. |
| Driver | Driver login | POST | `/api/driver/login` | No | No | Yes | Driver app/user role is not standard Odoo. |
| Driver | Driver Logout | POST | `/api/driver/logout` | No | No | Yes | Same as above. |
| Driver | Get Profile | GET | `/api/driver/profile` | No | No | Yes | Same as above. |
| Driver | Update Driver | PUT | `/api/driver/profile` | No | No | Yes | Same as above. |
| Driver | Pickup Order | PUT | `/api/driver/order/status` | No | No | Yes | Requires custom delivery workflow + driver assignment models. |
| Analytics | Sales data | GET | `/api/analytics/sales` | Yes | No | Yes | Reporting exists in Odoo, but a consolidated analytics API endpoint is custom. |
| Analytics | Product performance | GET | `/api/analytics/product` | Yes | No | Yes | Reporting exists; API endpoint is custom. |
| Analytics | Campaign performance | GET | `/api/analytics/campaign` | Partial | No | Yes | Depends on marketing modules; no dedicated endpoint found. |
| Analytics | Abandoned cart | GET | `/api/analytics/cart` | Yes | No | Yes | Abandoned cart behavior exists in `website_sale` (tests reference it), but API endpoint is custom. |

## Existing Tigro-Live customization that affects the mobile API requirements

- **MyFatoorah payments**
  - Implemented as a payment provider + JSON routes.
  - It can be reused as the payment integration layer, but it does not match the client’s `/api/payments/...` routes.

- **Gift wrap**
  - Existing custom gift wrap logic exists as a website JSON route.
  - A mobile-friendly API layer is still required to:
    - List options, add/remove gifts, and expose card designs (if needed).

- **Brand model**
  - A custom `product.brand` model exists in `Tigro-Live`.
  - A `/api/catalog/brands` endpoint can be implemented against that model.

- **Tigro custom fields**
  - `res.partner`: `block`, `floor`, `apartment` (see `custom_18_part/Tigro-Live/tigro_custom_fields/models/res_partner.py`).
  - `product.template`: `tigro_reference_number`.
  - `sale.order`: `sales_channel`.

## Summary

- The CSV describes a full mobile backend API.
- Odoo standard (plus enterprise modules present in this codebase) provides many underlying business objects (eCommerce, orders, payment, loyalty, helpdesk), but **it does not provide the same `/api/...` contract**.
- The current `Tigro-Live` codebase does **not** contain an implementation for the requested `/api/...` endpoints.
- Therefore, the project needs a **new API / interface layer** (custom controllers) that:
  - Maps the mobile endpoints to Odoo models and existing modules.
  - Reuses existing Tigro-Live pieces where applicable (MyFatoorah, gift wrap, brands, extra fields).
