# Liminal Recovery Runbook — Mobile Backup (EVM, Solana, Tron)

Use this only if you cannot reach your funds through Liminal.

## You need

- Your mobile backup `.zip` file and its backup password.
- The derivation path for each wallet. Liminal gives these; they are in the `walletKeys.path` field of the Retrieve a wallet API.
  - **EVM, new wallets:** `m/44/60/0/{org_id}/{deposit_wallet_index}/0`
  - **EVM, older wallets:** `m/44/60/0/0`
  - **Tron:** `<PATH>`
  - **Solana:** `<PATH>` (the full path to the address)
- Your destination address on each chain. Use addresses you control and have tested.
- **Laptop A (offline):** wiped, Wi-Fi off, never connected during recovery.
- **Laptop B (online):** clean, used only to send funds.
- A USB stick.

## 1. Prepare (Laptop B, online)

1. Get the recovery tool repo from Liminal.
2. Check the binary hash. It must match the hash Liminal sent you.
   - macOS / Linux: `shasum -a 256 bin/<binary>`
   - Windows: `certutil -hashfile bin\<binary> SHA256`
3. In the `utils` folder run `npm install`.
4. Download the Node.js installer for Laptop A.
5. Copy the repo (with `utils/node_modules`) and the Node.js installer to the USB stick.

## 2. Verify the backup (Laptop A, offline)

1. Install Node.js from the USB stick. Copy the repo to Laptop A.
2. Put your backup `.zip` in the repo's `bin` folder.
3. Open a terminal in `bin`. Run the binary for your OS, e.g. `./liminal-key-recovery-manager-mac-arm`.
4. Press **Enter**. Choose **4**.
5. Choose **2** (Mobile backup). Enter the `.zip` file name.
6. At `Enter Recovery key pair passphrase`, type your **backup password**. Nothing shows while you type.
7. If asked to pick a key, pick the one Liminal named. You may be asked twice.
8. You must see `Recovery package verification successful`. If not, stop and contact Liminal.

## 3. Get the keys (Laptop A, offline)

**EVM and Tron**

1. Run the binary again. Press **Enter**. Choose **5**.
2. Enter the `.zip` file name. Choose **2**. Type your backup password.
3. If asked, pick the key. Answer **y**. Copy the key that starts with `xprv`.
4. Open `utils/bip32-address-derivations.html` in a browser.
5. Paste the `xprv` into **BIP32 Root Key**.
6. Set **Coin** to `ETH - Ethereum` (for EVM) or `TRX - Tron`.
7. Open the **BIP32** tab. Set **Client** to `Custom derivation path`.
8. In **BIP32 Derivation Path**, enter the path from Liminal. Leave **Use hardened addresses** unchecked.
9. In the rows below, find the **Address** that matches your wallet. Row 0 is the wallet's main address. To see more rows, use **Show 20 more rows starting from index** at the bottom.
10. Copy that row's **Private Key**.

If your address is not in the rows, the path is wrong. Stop and check the path with Liminal.

**Solana**

1. Run the binary again. Press **Enter**. Choose **6**.
2. Enter the `.zip` file name. Choose **2**. Type your backup password.
3. If asked, pick the key. Enter the full Solana path from Liminal. Use numbers only, with no `'`.
4. Answer **y**. Copy the **EDDSA public backup key** and **EDDSA private backup key**.
5. In `utils` run `node solana-derive.js`. Paste the private key, then the public key.
6. Check that **Wallet Address** matches your Solana address and both checks show ✅.

Write each wallet's private key (and the Solana public key) to the USB stick. Do not save the `xprv`.

## 4. Move the funds (Laptop B, online)

Send a small test amount first on each chain. Each wallet needs its native coin for fees.

- **EVM:** import the private key into MetaMask. Send to your destination address. Repeat on each EVM network you use. The address is the same on all of them.
- **Tron:** import the private key into TronLink. Send to your destination address. Token transfers (e.g. USDT) burn TRX for fees, so keep enough TRX.
- **Solana:** the key cannot be imported into Phantom or other wallets. In `utils/solana-derive.js`:
  1. Remove the `/*` and `*/` around the "SAMPLE TRANSACTION CODE" block.
  2. Set `RPC_URL` to `https://api.mainnet-beta.solana.com`. Set `RECIPIENT` and `AMOUNT_SOL`. Leave a little SOL for the fee.
  3. Run `node solana-derive.js` and paste the keys. Note the transaction ID.

  This script sends SOL only. For SPL tokens (e.g. USDC), contact Liminal.

## 5. Clean up

1. Confirm all funds arrived at the destination addresses.
2. Remove the imported keys from MetaMask and TronLink.
3. Wipe Laptop A, Laptop B and the USB stick.
4. Treat the old wallets as compromised. Never send funds to them again.

## If something goes wrong

- **macOS blocks the binary:** run `xattr -d com.apple.quarantine <binary>` once.
- **Windows will not run the binary:** rename it to end in `.exe`.
- **`Incorrect password` or `Invalid private key`:** the backup password is wrong.
- **Address not found:** do not try other paths at random. Contact Liminal.
