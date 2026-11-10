# Liminal Recovery Runbook — Mobile Backup (EVM, Tron, Solana, BTC)

Use this only if you cannot reach your funds through Liminal.

## You need

- Your mobile backup `.zip` file and its backup password.
- The derivation path for each wallet. Get the exact path from the `walletKeys.path` field of the Retrieve a wallet API. The strategy is in [Liminal wallets derivation path strategy](https://docs.lmnl.app/docs/liminal-wallets-derivation-path-strategy).

  | Chain | New wallets | Legacy wallets |
  |---|---|---|
  | EVM | `m/44/60/0/{org_id}/{deposit_wallet_index}/0` | `m/44/60/0/0` |
  | Tron | `m/44/195/0/{org_id}/{deposit_wallet_index}/0` | `m/44/195/0/0` |
  | BTC | `m/49/1/0/{org_id}/{deposit_wallet_index}/0` | `m/49/1/0/0` |
  | Solana | `m/44/501/0/{org_id}/{deposit_wallet_index}/0/{address_index}` | `m/44/501/0/0/{address_index}` |

  - New wallets have `{org_id}` in the path. Wallets created before the new strategy use the legacy path.
  - `{address_index}` 0 is the wallet's main address. The tools add it for EVM, Tron and BTC. For Solana, enter the full path.
  - Liminal uses coin type `1` for BTC, not `0`.
  - Warm wallets use `m/44/{coin}/2/{wallet_id}` (BTC: `m/49/1/2/{wallet_id}`).
- Your destination address on each chain. Use addresses you control and have tested.
- A laptop that is wiped, with Wi-Fi off. It stays offline while you recover the keys.
- A USB stick.

## 1. Prepare

1. Get the recovery tool repo from Liminal.
2. Check the binary hash. It must match the hash Liminal sent you.
   - macOS / Linux: `shasum -a 256 bin/<binary>`
   - Windows: `certutil -hashfile bin\<binary> SHA256`
3. In the `utils` folder run `npm install`.
4. Download the Node.js installer.
5. Copy the repo (with `utils/node_modules`) and the Node.js installer to the USB stick.

## 2. Verify the backup (offline laptop)

1. Install Node.js from the USB stick. Copy the repo to the laptop.
2. Put your backup `.zip` in the repo's `bin` folder.
3. Open a terminal in `bin`. Run the binary for your OS, e.g. `./liminal-key-recovery-manager-mac-arm`.
4. Press **Enter**. Choose **4**.
5. Choose **2** (Mobile backup). Enter the `.zip` file name.
6. At `Enter Recovery key pair passphrase`, type your **backup password**. Nothing shows while you type.
7. If asked to pick a key, pick the one Liminal named. You may be asked twice.
8. You must see `Recovery package verification successful`. If not, stop and contact Liminal.

## 3. Get the keys (offline laptop)

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

**BTC**

1. Get the `xprv` as in EVM and Tron, items 1–3.
2. In `utils` run `node btc-derive.js`.
3. Paste the `xprv`. Enter the BTC path from Liminal as the base path. Enter `0` as the start index and `20` as the count.
4. Find the **Address** that matches your wallet. Copy its **Private Key (WIF)**.

**Solana**

1. Run the binary again. Press **Enter**. Choose **6**.
2. Enter the `.zip` file name. Choose **2**. Type your backup password.
3. If asked, pick the key. Enter the full Solana path from Liminal. Use numbers only, with no `'`.
4. Answer **y**. Copy the **EDDSA public backup key** and **EDDSA private backup key**.
5. In `utils` run `node solana-derive.js`. Paste the private key, then the public key.
6. Check that **Wallet Address** matches your Solana address and both checks show ✅.

Write each wallet's private key (and the Solana public key) to the USB stick. Do not save the `xprv`.

## 4. Move the funds

Send a small test amount first on each chain. Each wallet needs its native coin for fees.

- **EVM:** the private key can be imported into wallets such as MetaMask to sign transactions. The address is the same on every EVM network.
- **Tron:** the private key can be imported into wallets such as TronLink to sign transactions. Token transfers (e.g. USDT) burn TRX for fees, so keep enough TRX.
- **BTC:** the WIF private key from `btc-derive.js` can be imported into a BTC wallet that accepts WIF keys to sign transactions.
- **Solana:** the key cannot be imported into Phantom or other wallets. Use `utils/solana-derive.js` to sign and send:
  1. Remove the `/*` and `*/` around the "SAMPLE TRANSACTION CODE" block.
  2. Set `RPC_URL` to `https://api.mainnet-beta.solana.com`. Set `RECIPIENT` and `AMOUNT_SOL`. Leave a little SOL for the fee.
  3. Run `node solana-derive.js` and paste the keys. Note the transaction ID.

  This script sends SOL only. For SPL tokens (e.g. USDC), contact Liminal.

## 5. Clean up

1. Confirm all funds arrived at the destination addresses.
2. Remove the imported keys from every wallet you used.
3. Delete the files `liminal-ecdsa-private-backup-key` and `liminal-eddsa-private-backup-key` from `bin`.
4. Wipe the laptop and the USB stick.
5. Treat the old wallets as compromised. Never send funds to them again.

## If something goes wrong

- **macOS blocks the binary:** run `xattr -d com.apple.quarantine <binary>` once.
- **Windows will not run the binary:** rename it to end in `.exe`.
- **`Incorrect password` or `Invalid private key`:** the backup password is wrong.
- **Address not found:** do not try other paths at random. Contact Liminal.
