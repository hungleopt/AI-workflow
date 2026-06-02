# 003 - FE - Frontend GTM currency

## CONTEXT

- Ticket: FMI-944
- Branch: `feat/FMI-944-eur-currency-ordering`
- Review: `docs/src/tickets/FMI-944-review.md`
- Depends on: `001-add-currency-infrastructure`

## FILES IN SCOPE

- `firstmile.widgets/src/helpers/gtm.ts`
- `firstmile.widgets/src/blocks/cart/Cart.tsx`

## TASK

1. **gtm.ts** — Modify `gtmPushEventAddToCart` to accept a `currencyCode` parameter (default `"GBP"`):
   ```typescript
   export const gtmPushEventAddToCart = (item: { name: string; variant: string; price: string; quantity: number; id: string }, currencyCode: string = "GBP") => {
     window.dataLayer = window.dataLayer || [];
     window.dataLayer.push({
       event: "addToCart",
       ecommerce: {
         currencyCode, // was hardcoded "GBP"
         add: { products: [item] },
       },
     });
   };
   ```

2. **Cart.tsx** — In `handlePushDataLayer` (~source line 130 in original):
   - Change `currencyCode: 'GBP'` → use a prop/context value for currency
   - The Cart component receives props from the backend (via `CartPageController`). Add a `currencyCode` prop and pass it through.

3. Ensure the backend `CartPageController` passes `currencyCode` in the JSON data sent to the React component.

## DONE WHEN

- [ ] `gtmPushEventAddToCart` accepts dynamic `currencyCode`
- [ ] Cart component reads currency from backend props
- [ ] DataLayer push events use dynamic currency
- [ ] Compiles without errors (frontend build)
- [ ] No files outside CONTEXT modified
- [ ] No claim made about existing code without citing file:line
