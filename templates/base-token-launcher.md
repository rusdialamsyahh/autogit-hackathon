---
title: Base Token Launcher
app_type: base-token-launcher
wallet: 0xc335172422ABEB483E97a370Ae6AF03dc29c29C7
---

You are an expert Web3 full-stack developer. Build a complete token launcher app on Base L2 where users can create and deploy their own ERC-20 token in one click — no coding required.

## Core Features

### 1. Token Creator Form
- Token name, symbol, total supply, decimals (default 18)
- Optional: token logo upload (stored on IPFS via Pinata)
- Optional: mint to multiple addresses at launch
- Live preview card showing how the token will look on Basescan

### 2. One-Click Deploy
- User connects wallet (MetaMask / Coinbase Wallet / WalletConnect)
- Estimated gas cost shown before deploy
- Deploy ERC-20 contract directly to Base mainnet using wagmi + viem
- Use a minimal ERC-20 factory contract (CREATE2 for deterministic address)
- Show live tx status with progress bar

### 3. Token Dashboard (post-deploy)
- Deployed contract address with Basescan link
- Total supply, holder count (from Basescan API)
- Add to MetaMask button (wallet_watchAsset)
- Share card: tweet-ready image with token name, symbol, contract address
- QR code linking to Basescan token page

### 4. My Tokens History
- List of tokens the user has deployed (stored in localStorage)
- Quick stats: holders, transfers, liquidity (via Basescan API)

## Technical Requirements

### Stack
- React 18 + TypeScript + Vite
- Tailwind CSS v3 + shadcn/ui components
- wagmi v2 + viem for wallet connection and contract interaction
- @tanstack/react-query for async state
- RainbowKit for wallet modal UI

### Contract Interaction
- Use wagmi `useDeployContract` for deploying the ERC-20 bytecode
- ABI and bytecode for a standard OpenZeppelin ERC-20 should be inlined (no backend needed)
- Chain: Base mainnet (chainId 8453) and Base Sepolia testnet (chainId 84532) with network switcher

### Design
- Clean, modern UI with dark mode support
- Main color: blue (#2222FF) matching Base brand
- Step-by-step wizard layout: Configure → Preview → Deploy → Done
- Confetti animation on successful deployment
- Mobile-responsive

### No Backend Required
- Everything runs client-side
- Token metadata stored in localStorage
- Basescan API called directly from frontend for token stats

## Page Structure

```
/ (Home)
  - Hero: "Launch your token on Base in 60 seconds"
  - Token creator form (step wizard)
  - Recent launches (from localStorage)

/token/:address (Token Detail)
  - Token info card
  - Basescan embed link
  - Add to wallet button
  - Share options
```

## Sample UX Flow

1. User lands on page, sees "Launch Token" CTA
2. Fills in: Name="CoolToken", Symbol="COOL", Supply=1,000,000
3. Clicks "Preview" → sees token card with logo placeholder
4. Clicks "Deploy to Base" → wallet popup opens
5. Confirms tx → loading screen with Base logo spinning
6. Success screen: contract address, Basescan link, "Add to MetaMask" button
7. Confetti rains down

## Error Handling
- Insufficient ETH for gas: show "Get ETH on Base" link to bridge
- Wallet not connected: prompt connect modal
- Network mismatch: prompt switch to Base
- Tx rejected: friendly error with retry button
