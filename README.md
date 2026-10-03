# web3-support-runbooks
Standard Operating Procedures (SOPs) and diagnostic workflows for troubleshooting Web3 user friction points.
# SOP: Stuck Transactions & Mempool Congestion

## Context
When a user buys or transfers crypto, high network activity can lead to transactions lingering in the mempool if the base gas or priority fee spikes above what was estimated at order broadcast.

## Diagnostic Flowchart

1. **Transaction Hash Validation**
   - Obtain the user’s order ID and locate the internal transaction hash (`tx_hash`).
   - Query the hash on the native chain explorer:
     - Bitcoin: `mempool.space/tx/`
     - Ethereum: `etherscan.io/tx/`
     - Solana: `solscan.io/tx/`
     - Polygon: `polygonscan.com/tx/`

2. **Mempool Status Verification**
   - **Status: Dropped / Not Found:** The transaction was rejected by nodes (e.g., nonce gap or underpriced gas limit). Escalate to internal payment operations to rebroadcast with updated base fee.
   - **Status: Pending / In Mempool:** The transaction is broadcasted but waiting for block inclusion. Check the current median gas price on the network against the gas fee allocated on the transaction.

3. **Resolution Actions**
   - **User Education:** If the transaction is safely queued and gas prices are actively adjusting down, explain mempool dynamics calmly and share the live explorer link.
   - **Gas Acceleration:** If the payment engine supports `Replace-By-Fee` (RBF) or speed-up transactions via the payout wallet, notify Tier 2 operations to replace the transaction using the identical nonce with a 20%+ higher priority fee.

## Customer Response Template

> Hi [Customer Name],
>
> Your order has been processed by our payment gateway, and the transaction was broadcast to the [Network Name] blockchain under transaction hash: `[tx_hash]`.
>
> Currently, the [Network Name] network is experiencing elevated traffic, which has temporarily increased average confirmation times. Your transaction is securely queued in the public mempool waiting to be written to the next available block.
>
> You can monitor real-time block confirmation status directly on the explorer here: `[Explorer Link]`.
>
> # SOP: Cross-Chain Routing & Network Mismatches

## Context
A customer purchases a multi-chain token (e.g., USDT, USDC) or native asset, receives confirmation that delivery succeeded, but reports zero balance in their self-custody wallet (Trust Wallet, MetaMask, Brave Wallet) because their wallet is defaulted to the wrong network.

## Triage Workflow

1. **Verify Receiving Public Key**
   - Check the destination address submitted during checkout.
   - For EVM transactions (starts with `0x`), check the address across:
     - Ethereum Mainnet (`etherscan.io`)
     - Arbitrum One (`arbiscan.io`)
     - Optimism (`optimistic.etherscan.io`)
     - Polygon PoS (`polygonscan.com`)
     - BNB Smart Chain (`bscscan.com`)

2. **Locate Contract Balance**
   - Verify which specific chain processed the token mint or transfer event.
   - Confirm whether the tokens are present in the wallet address on that specific chain.

3. **Client Configuration Check**
   - If funds exist on-chain: The issue is local wallet visibility, not a delivery failure.
   - Provide explicit network switching steps or custom token import parameters (Contract Address, Symbol, Decimals).

## Customer Response Template

> Hi [Customer Name],
>
> We confirmed that your funds were successfully delivered to your wallet address `[Wallet Address]` on the **[Network Name]** network.
>
> You can verify the on-chain confirmation here: `[Explorer Link]`.
>
> Because [Wallet Provider, e.g., Trust Wallet / MetaMask] defaults to Ethereum Mainnet, you simply need to switch networks in your wallet app to see your balance:
>
> 1. Open your wallet and tap the **Network Selector** at the top.
> 2. Select **[Network Name]**.
> 3. If the token balance does not populate automatically, tap **Import / Add Custom Token** and paste this contract address:
>    - **Contract Address:** `[Token Contract]`
>    - **Symbol:** `[e.g., USDT]`
>    - **Decimals:** `[e.g., 6 or 18]`
>
> Once added, your balance will appear immediately.
>
> # SOP: WalletConnect & Injected Provider Failures

## Context
Users attempting to connect a mobile non-custodial wallet via WalletConnect or a desktop extension (Brave Wallet, MetaMask) fail to bind their wallet or execute signature handshakes.

## Common Root Causes & Diagnostics

| Symptom | Root Cause | Fix |
| :--- | :--- | :--- |
| **Deeplink does not open app on iOS/Android** | Broken Universal Link or browser pop-up block | Instruct user to copy the raw WalletConnect URI and manually paste it into their wallet’s internal dApp browser. |
| **"Session Request Disconnected"** | Stale pairing cached in local storage | Clear previous pairing sessions within the wallet settings. |
| **Wallet extension does not trigger on Brave** | Brave Shields blocking provider injection | Set Shields down for the checkout domain, or toggle default wallet provider under `brave://settings/web3`. |

## Escalation Checklist for Frontend Bugs
When an issue is reproducible across multiple users on the same device/browser combo:
- Device Model & Operating System Version.
- Browser & Version (e.g., Brave v1.65.12).
- Wallet Name & App Version (e.g., Trust Wallet v8.12).
- Browser Console Logs (F12 > Console tab) capturing RPC handshake errors.
- Network HAR capture if API endpoints fail during handshake.

- # SOP: Fiat Gateway Drop-offs & Payment Rejections

## Context
A customer attempts to complete a card or bank transfer on-ramp purchase, but the payment gateway halts the transaction prior to crypto settlement.

## Error Codes & Resolution

### 1. `3DS_AUTHENTICATION_FAILED` / `CARD_NOT_ENROLLED`
- **Cause:** Customer failed two-factor SMS/OTP verification with their issuing bank, or the issuing bank does not participate in 3D Secure 2.0.
- **Support Action:** Advise customer to contact their bank to authorize international card payments or switch to an alternate payment method (e.g., Instant SEPA / Apple Pay).

### 2. `DO_NOT_HONOR` / `DECLINED_BY_ISSUER`
- **Cause:** Bank-level automated fraud filters blocking MCC 6051 (Quasi-Cash / Cryptocurrency Merchants).
- **Support Action:** Explain that the block originates on the bank's fraud detection system. The user must call their bank's fraud desk to whitelist transactions from the payment entity.

### 3. `VELOCITY_LIMIT_EXCEEDED`
- **Cause:** Internal fraud system triggered due to multiple card attempts within a rolling window.
- **Support Action:** Check user's KYC tier. If verified, request a risk tier override from Risk/Fraud operations; otherwise, inform the user to retry after a 24-hour cooldown.

## Customer Response Template

> Hi [Customer Name],
>
> We looked into your recent payment attempt of [Amount]. The payment processor received a decline response directly from your card-issuing bank with the status: **[Error Description]**.
>
> Because crypto purchases are classified as digital asset transactions, many banks require direct phone authorization before approving the charge.
>
> **Recommended Next Steps:**
> 1. Call your bank’s card support line (number on the back of your card) and ask them to approve pending transactions from our payment gateway.
> 2. Alternatively, try using **[Apple Pay / Instant Bank Transfer / Debit Card]**, which typically experience significantly higher acceptance rates.
>
> No further action is required on your side; your funds will reflect in your wallet balance automatically as soon as the network confirms the block.
