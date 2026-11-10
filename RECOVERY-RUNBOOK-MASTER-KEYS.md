# Liminal Recovery Runbook — Master Keys Only (Mobile Backup)

This runbook recovers your two master private keys from your mobile backup. 

- **ECDSA master key (`xprv`)** — controls your EVM, Tron and BTC wallets.
- **EdDSA master key + chain code** — controls your Solana wallets.

**Whoever holds these keys controls all your wallets.** Protect them like the backup itself.

## You need

- Your mobile backup `.zip` file and its backup password.
- A laptop that is wiped, with Wi-Fi off. It stays offline for the whole runbook.
- The recovery tool repo from Liminal. Check the binary hash matches the one Liminal sent you (`shasum -a 256 <binary>`, or `certutil -hashfile <binary> SHA256` on Windows).
- Secure storage for the keys: paper in a safe, or an encrypted USB stick.

## 1. Set up

1. Copy the repo to the offline laptop.
2. Put your backup `.zip` in the repo's `bin` folder.
3. Disconnect all networks.
4. Open a terminal in `bin`.

## 2. Verify the backup

1. Run the binary for your OS, e.g. `./liminal-key-recovery-manager-mac-arm`.
2. Press **Enter**. Choose **4**.
3. Choose **2** (Mobile backup). Enter the `.zip` file name.
4. At `Enter Recovery key pair passphrase`, type your **backup password**. Nothing shows while you type.
5. If asked to pick a key, pick the one Liminal named. You may be asked twice.
6. You must see `Recovery package verification successful`. If not, stop and contact Liminal.

## 3. Recover the ECDSA master key (EVM, Tron, BTC)

1. Run the binary again. Press **Enter**. Choose **5**.
2. Enter the `.zip` file name. Choose **2**. Type your backup password.
3. If asked to pick a key, pick the one Liminal named.
4. Answer **y** to reveal.
5. Record the key that starts with `xprv`.

## 4. Recover the EdDSA master key (Solana)

1. Run the binary again. Press **Enter**. Choose **6**.
2. Enter the `.zip` file name. Choose **2**. Type your backup password.
3. If asked to pick a key, pick the one Liminal named.
4. When asked for the BIP 32 path, enter only: `m`
5. Answer **y** to reveal.
6. Record all three values: **Master chain code**, **EDDSA public backup key**, **EDDSA private backup key**.

## 5. Store and clean up

1. Check each recorded value character by character against the screen.
2. Put the records in secure storage.
3. Delete the files `liminal-ecdsa-private-backup-key` and `liminal-eddsa-private-backup-key` from `bin`.
4. Close the terminal. Wipe the laptop.

## Next step

To move funds, get the derivation path for each wallet (see [Liminal wallets derivation path strategy](https://docs.lmnl.app/docs/liminal-wallets-derivation-path-strategy) and the `walletKeys.path` field of the Retrieve a wallet API). Then follow `RECOVERY-RUNBOOK.md`:

- **EVM, Tron, BTC:** in step 3, skip the items that get the `xprv`. Paste the `xprv` you recorded here.
- **Solana:** run step 3 in full. The tool must derive the key from your backup, so keep the backup `.zip` and password.
