# Stream Receipts (PROD-01)

## Feasibility

Soroban has no native NFT primitive, and there is no ratified SEP for
non-fungible tokens on Stellar yet. A separate NFT contract would add a
cross-contract call to every `withdraw` and tie PayStream to a draft interface.

**Decision:** implement receipts natively inside the stream contract. Each
stream has exactly one receipt, keyed by stream ID (`DataKey::ReceiptOwner`).
The interface (`receipt_owner`, `transfer_receipt`) mirrors `owner_of` /
`transfer` so it can be wrapped by a standard NFT contract later without
changing stream storage.

## Design

| Action | Behaviour |
|---|---|
| `create_stream`, `create_streams_batch` | Mint the receipt to `employee` |
| `receipt_owner(stream_id)` | Current holder; falls back to `stream.employee` for streams created before receipts existed |
| `transfer_receipt(from, to, stream_id)` | Holder moves the receipt; `to` is added to the employee index; emits `receipt` event |
| `withdraw` | Only the receipt holder may withdraw (E016 otherwise) |
| `withdraw_all` | Skips streams whose receipt the caller no longer holds |

Unchanged: `stream.employee` keeps the original employee for audit and events;
employer controls (pause, resume, cancel, top-up, rate updates) are unaffected.
On `cancel_stream` / `cancel_streams_batch`, accrued earnings are paid to the
receipt holder and the unearned remainder is refunded to the employer.

Receipts carry no metadata on-chain; indexers derive it from `get_stream`.

## Legal and compliance considerations

> **Disclaimer:** This section is guidance for integrators, not legal advice.
> It has not replaced review by qualified counsel in your jurisdiction. Before
> enabling receipt transfers in production, have it reviewed by a qualified
> legal reviewer. *Legal review status: pending.*

### General considerations

- **Wage assignment law:** many jurisdictions restrict or prohibit assigning
  future wages. Integrators must confirm transfers are lawful for their
  employees and may need to disable transfers in the UI.
- **Securities classification:** a transferable claim on future income could be
  treated as a security or financial instrument. Secondary markets built on
  receipts need their own legal review.
- **KYC/AML:** transferring a receipt moves value to an unverified address;
  regulated employers should restrict transfers to verified accounts off-chain.
- **Tax and payroll reporting:** the employer's payroll obligations attach to
  the employee, not the receipt holder. Withdrawals by a third party do not
  change the employer's reporting duties.
- **Employer risk:** the employer can still cancel a stream; buyers of a
  receipt bear that counterparty risk and should be informed.

### United States

- **Wage assignments:** most states regulate or limit assignments of future
  wages (e.g. written consent, spousal consent, percentage caps, revocability,
  filing with the employer). Some states void assignments of unearned wages
  entirely. Check the law of the state where the employee works.
- **Federal pay rules:** the FLSA requires wages be paid "free and clear";
  transfers must not be used to route wages back to the employer or to push
  pay below minimum wage.
- **Securities:** apply the *Howey* test (investment of money, common
  enterprise, expectation of profit from others' efforts). Pooled or
  marketed receipts are higher risk; consult SEC guidance on digital assets.
- **AML:** a platform facilitating receipt transfers for value may be a money
  services business under FinCEN rules and state money-transmitter laws;
  OFAC sanctions screening applies to counterparties.

### European Union

- **Wage protection:** member-state labour codes commonly restrict the
  assignment or attachment of wages (e.g. protected minimum amounts); rules
  differ per country, so check the employee's country of employment.
- **Securities / crypto-assets:** assess whether a receipt is a financial
  instrument under MiFID II or a crypto-asset under MiCA (including whether
  the NFT exemption applies — fractionalised or series issuance generally
  does not qualify).
- **AML:** the AML Directives and the Transfer of Funds Regulation (travel
  rule) apply to crypto-asset service providers handling transfers.
- **Data protection:** linking receipt holders to employees is personal data
  under GDPR; keep that mapping off-chain and minimise it.

### Compliance checklist

Complete before enabling receipt transfers for any deployment:

- [ ] **Wage-assignment law check:** confirmed for each jurisdiction where
      employees work whether assigning future wages is permitted and under
      what conditions (consent, caps, revocation). Owner: ______
- [ ] **Securities classification review:** written assessment (Howey / MiFID
      II / MiCA) of whether receipts, or any marketplace for them, are
      securities or regulated crypto-assets. Owner: ______
- [ ] **KYC/AML considerations:** transfers restricted to verified accounts;
      sanctions screening in place; money-transmitter / CASP licensing
      assessed. Owner: ______
- [ ] **Tax/payroll:** confirmed reporting continues to reference the
      original employee. Owner: ______
- [ ] **Disclosure:** employees and any receipt buyers informed of employer
      cancellation risk. Owner: ______
- [ ] **Legal sign-off:** reviewed by a qualified legal reviewer. Owner: ______

## Technical integration: disabling receipt transfers in the UI

The contract always exposes `transfer_receipt`; there is no on-chain switch.
Integrators who must block transfers should do so in their application layer:

1. **Hide the action:** do not render any "transfer receipt" control, and
   remove `transfer_receipt` from any generated client bindings you ship.
2. **Gate by config:** use a feature flag (e.g. `RECEIPT_TRANSFERS_ENABLED=false`)
   per jurisdiction, defaulting to disabled.
3. **Refuse to sign:** in custodial or wallet-integrated flows, reject any
   transaction whose invocation is `transfer_receipt` on the stream contract.
4. **Monitor:** watch for `receipt` events from your streams; if one appears
   (e.g. sent via the CLI directly), alert compliance and treat the new holder
   per your policy.
5. **Display the holder:** show `receipt_owner(stream_id)` alongside
   `stream.employee` so mismatches are visible to support staff.

PayStream provides the mechanism only and does not endorse any secondary
market. This document is guidance, not legal advice.
