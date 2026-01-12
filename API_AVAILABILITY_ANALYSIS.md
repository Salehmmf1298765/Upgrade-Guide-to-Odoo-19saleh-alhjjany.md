# Tigro Mobile App API - Availability (Odoo 18) + Estimate

## 1) Summary (Available / Not Available)

### Available in Odoo (no business customization)

- Authentication (session-based) and portal customer access.
- Customer profile and addresses (`res.partner` + child addresses).
- Countries/states and (if installed) cities.
- eCommerce catalog: categories, products, variants, images, SEO fields.
- Related/upsell/cross-sell products.
- Cart + checkout flow (website sale order in session).
- Orders/invoices (portal).
- Payment providers framework.
- Shipping fee calculation framework (`delivery` + checkout integration).
- Coupons/promotions/loyalty framework (`loyalty`, `sale_loyalty`, `website_sale_loyalty`).
- Wishlist (`website_sale_wishlist`).
- Helpdesk tickets (portal support) (`helpdesk`).
- Ratings framework (`rating` + product uses `rating.mixin`).
- CMS pages and FAQ snippets (website).

### Available via existing Tigro-Live customization

- MyFatoorah payment integration (payment provider + JSON routes under `/payment/myfatoorah/...`).
- Gift wrap add-to-cart logic (JSON route `/shop/cart/giftwrap`).
- Brands: custom model `product.brand`.
- Extra fields:
  - `res.partner`: `block`, `floor`, `apartment`.
  - `product.template`: `tigro_reference_number`.
  - `sale.order`: `sales_channel`.

### Not available (requires new business customization in addition to the API layer)

- Guest login concept (dedicated guest session/account endpoint).
- Refresh token flow (if required by the mobile contract).
- OTP-based verification/resend.
- Geographic “areas/zones” model (if required by the app).
- Flash sale concept.
- Delivery zones, ETA, delivery slots, live tracking (requires logistics integration).
- Confirm delivery and return requests (customer self-service flows).
- Wallet top-up flow.
- Loyalty tier / referral program.
- Mobile notifications model/contract (beyond standard messaging).
- Driver role/app endpoints and driver order workflow.
- Aggregated analytics endpoints (sales/product/campaign/abandoned cart) as a mobile API contract.

## 2) New Odoo addons to be developed (API layer)

| Addon (New) | Scope (Endpoints) | Key Dependencies | Estimate (Hours) |
|---|---|---|---:|
| `tigro_mobile_api_core` | API base, response format, error handling, auth/session/token strategy | `web`, `portal` | 60-80 |
| `tigro_mobile_api_customer` | Profile, preferences, addresses, location | `base`, `portal` | 40-60 |
| `tigro_mobile_api_catalog` | Categories, brands, products, search, availability | `website_sale`, `stock` | 60-85 |
| `tigro_mobile_api_shop` | Cart, checkout, orders, invoices, payment methods | `website_sale`, `sale`, `account`, `payment`, `delivery` | 100-140 |
| `tigro_mobile_api_loyalty` | Coupons, promotions, loyalty, wallet endpoints | `loyalty`, `sale_loyalty`, `website_sale_loyalty` | 70-110 |
| `tigro_mobile_api_support` | Helpdesk tickets + comments | `helpdesk`, `mail`, `portal` | 35-55 |
| `tigro_mobile_api_content` | CMS (home/banners/pages/faq) + system config/flags | `website` | 40-65 |
| `tigro_mobile_api_engagement` | Reviews + notifications endpoints | `rating`, `mail` | 70-110 |
| `tigro_mobile_api_logistics` | Delivery zones/ETA/slots/live + driver workflow | `delivery`, `stock_delivery` + external integration | 200-320 |
| `tigro_mobile_api_analytics` | Analytics endpoints (sales/product/campaign/cart) | `sale`, `website_sale` (+ marketing modules if needed) | 60-100 |

## 3) Effort estimation (Hours)

### Phase 1 (Core Mobile API + eCommerce)

| Workstream | Estimate (Hours) |
|---|---:|
| API base + authentication strategy | 60-80 |
| Customer/profile/addresses/location | 40-60 |
| Catalog/products/brands/search/availability | 60-85 |
| Cart/checkout/orders/invoices | 100-140 |
| Payments integration (incl. MyFatoorah mapping to mobile contract) | 40-70 |
| Loyalty/promotions/wallet (as per CSV contract) | 70-110 |
| Wishlist | 16-24 |
| Reviews | 24-40 |
| CMS + system config/flags | 40-65 |
| Support tickets (helpdesk) | 35-55 |
| QA + test cycles + handover | 50-80 |
| **Phase 1 Total** | **535-809** |

### Phase 2 (Advanced Logistics + Driver + Analytics)

| Workstream | Estimate (Hours) |
|---|---:|
| Delivery zones/ETA/slots/live tracking (integration-dependent) | 80-140 |
| Driver app endpoints + driver order status workflow | 120-180 |
| Loyalty tier + referral (if required by app) | 30-60 |
| Returns + confirm delivery flows | 40-70 |
| Analytics API endpoints (aggregation + access rules) | 60-100 |
| QA + performance + security hardening | 40-70 |
| **Phase 2 Total** | **370-620** |

### Assumptions (for estimation)

- REST endpoints are implemented as Odoo HTTP controllers returning JSON.
- Access control and record rules are enforced per endpoint.
- External logistics/live tracking integration is treated as a separate dependency impacting Phase 2.

## 4) Endpoint matrix

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
