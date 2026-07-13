# Instacart Marketplace — external review handoff

Run this branch **only when I explicitly ask for Instacart for this trip**. Instacart is
not a saved household fulfillment preference, so availability alone never reroutes a
generic “place the order,” Kroger trip, satellite fill, or store walk. Conversely, an
explicit Instacart request must not fall through into `place_order` or a walk even when
my standing primary store is Kroger or Offline.

1. **Create the page once.** Call `create_instacart_handoff()` with no arguments. It uses
   the same current derived unchecked to-buy set as the opening review. Do not call
   `place_order`, build a walk, choose a retailer, or try to translate lines into Kroger
   products.
2. **Keep incomplete plans visible.** If either the opening `read_to_buy` or the handoff
   result reports `underived`, name those recipes and warn that their ingredients are
   absent from the Marketplace page. Offer to capture their ingredients explicitly only
   with my approval; never imply the page is complete.
3. **Report the discriminated result exactly.** On `ready`, give me the URL and say it is
   an Instacart Marketplace shopping-list page ready for **my review**: I choose a
   retailer, review matches, add items, and check out there. On `empty`, say there is
   nothing to hand off. On `not_configured` or a structured error, state the safe reason
   and invite an explicit retry only when `retryable` is true.
4. **Stop at the handoff boundary.** Creating, reusing, or opening the URL proves none of
   product matching, carting, checkout, ordering, or purchase. Never say “sent to cart”
   or “order placed”; never advance rows, stamp `sent_in`, write a send/spend event,
   restock pantry, or infer later lifecycle from the URL. A later purchase/receipt claim
   must come from the member through an existing explicit fulfillment flow, not from this
   handoff.
