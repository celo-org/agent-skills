# MiniPay Troubleshooting

## Common Errors

### "Cannot read property of undefined"

**Cause:** Accessing `window.ethereum` before checking if it exists.

**Solution:** Always check for browser environment first:
```typescript
if (typeof window !== "undefined" && window.ethereum) {
  // Safe to use window.ethereum
}
```

### Transaction Fails Silently

**Cause:** MiniPay uses legacy transactions, EIP-1559 properties are ignored.

**Solution:** Don't include `maxFeePerGas` or `maxPriorityFeePerGas` in transactions. Viem handles this automatically when using Celo chains.

### "eth_sendTransaction not implemented"

**Cause:** Using HTTP transport instead of injected provider.

**Solution:** Use `custom(window.ethereum)` transport for wallet operations:
```typescript
// Wrong
const client = createWalletClient({
  chain: celo,
  transport: http("https://forno.celo.org"),
});

// Correct
const client = createWalletClient({
  chain: celo,
  transport: custom(window.ethereum),
});
```

### "Permission denied" on a listed app (works in preview, fails in production)

**Cause:** A **listed/registered** MiniPay Mini App is approved against an
**allowlist** of the exact contract address(es), functions, and input-field
types it calls. MiniPay only permits `eth_sendTransaction` for calls matching
that approved set on the **registered domain**. If the contract changed after
approval — a **new deployment address**, a **changed function signature** (new
4-byte selector), or **added/renamed/re-typed input params** — the call is no
longer in the allowlist and MiniPay rejects it with a masked **"Permission
denied"** (surfaces via viem as `ContractFunctionExecutionError`), **before
broadcast**, so nothing reaches the chain.

**Also in scope: URLs, not just contracts.** The URLs, subdomains, and origins
you submitted are part of the same approval. Moving to a new production domain,
adding a CDN host, or calling an analytics or RPC endpoint that was not in your
submitted manifest can fail the same way.

**How to recognize it:** the *identical build* sends fine from an **unregistered**
URL (a fresh `*.vercel.app`, ngrok, or any non-listed page — those get default
in-app-browser permissions) but fails on the **registered** production domain.
"Same commit, works in preview, denied in production" ⇒ it's the allowlist, not
your code. It is **not** the `feeCurrency` (canonical adapters are correct) and
**not** the gas limit — changing those won't help.

**Confirm the exact denied call:** wrap the provider to log requests —

```typescript
const orig = window.ethereum.request.bind(window.ethereum);
window.ethereum.request = async (args) => {
  try { return await orig(args); }
  catch (e) { console.error("MiniPay denied:", args.method, args.params, e); throw e; }
};
// You'll see: eth_sendTransaction { to, feeCurrency, data(sel=0x…) } -> Permission denied
```

**Solution (MiniPay-side, not client-side):** notify the MiniPay team of the
contract update and send updated **sample transactions** for each new/changed
method so they re-approve. **Treat every contract change you ship for MiniPay as
requiring MiniPay re-approval** — approval covers specific functions with
specific input types, so any change must be communicated before it hits the
listed app.

**Prevention:** submit the **production-ready** build with the ABI and the URL
set frozen, not a staging version you intend to change. This is the single most
common way an app that passed review starts failing in production.

### App Not Loading in MiniPay

**Possible causes:**
1. ngrok tunnel expired or restarted
2. Using HTTP instead of HTTPS
3. Mixed content errors

**Solutions:**
- Restart ngrok and get new URL
- Always use the HTTPS URL from ngrok
- Check browser console for mixed content warnings

### Gas Estimation Errors

**Cause:** Insufficient funds or invalid transaction parameters.

**Solutions:**
- Ensure account has sufficient balance for gas
- For fee currency transactions, ensure account has enough USDm
- Verify recipient address is valid
- Check token decimals (USDm uses 18, USDC/USDT use 6)

### Wallet Not Detected

**Cause:** App not running inside MiniPay webview.

**Solution:** Verify you're testing within MiniPay:
```typescript
console.log("Is MiniPay:", window.ethereum?.isMiniPay);
console.log("Provider exists:", !!window.ethereum);
```

### Transaction Pending Forever

**Cause:** Gas price too low or network congestion.

**Solutions:**
- Wait for network conditions to improve
- Increase gas price if setting manually
- Check transaction on Celoscan

### CORS Errors

**Cause:** Cross-origin requests blocked.

**Solution:** Configure your server to allow cross-origin requests or use appropriate headers in your framework configuration.

### Message Signing Not Working

**Note:** Message signing is not currently supported in MiniPay.

### EIP-1559 Errors

**Cause:** MiniPay only supports legacy transactions.

**Solution:** Don't pass `maxFeePerGas` or `maxPriorityFeePerGas`. Viem handles this automatically for Celo chains.

## Debugging Tips

### Check MiniPay Detection

```typescript
console.log("Is MiniPay:", window.ethereum?.isMiniPay);
console.log("Provider:", window.ethereum);
console.log("Accounts:", await window.ethereum?.request({
  method: "eth_accounts"
}));
```

### Remote Debugging with Chrome

1. Connect phone via USB
2. Enable USB debugging on phone
3. Open `chrome://inspect` in Chrome
4. Find your app's webview and click "inspect"

### Monitor ngrok Traffic

ngrok provides a local dashboard at `http://localhost:4040` where you can:
- See all incoming requests
- Inspect request/response headers
- Replay requests for debugging

## Testing Checklist

### Basic Functionality
- [ ] App loads in MiniPay webview
- [ ] Wallet address detected automatically
- [ ] Connect button hidden when in MiniPay
- [ ] Balance reads correctly

### Transactions
- [ ] Transactions submit successfully
- [ ] Transaction confirmation works
- [ ] Error handling displays properly
- [ ] Correct decimals for each token type

### Edge Cases
- [ ] Handle network errors gracefully
- [ ] Handle insufficient balance
- [ ] Handle user rejection
- [ ] Handle invalid addresses
