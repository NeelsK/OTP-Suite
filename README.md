# One-Time Pad Suite

A fully offline, browser-based one-time pad (OTP) encryption tool. Generate cryptographically secure pads, encrypt and decrypt messages, hide pads or ciphertext inside images using steganography, and export/import pad libraries — all without any server, account, or internet connection.

Open `otp-suite.html` in any modern browser to get started. Everything runs client-side in JavaScript; nothing is ever transmitted.

---

## Why One-Time Pads?

A one-time pad is the only encryption scheme that is **mathematically proven to be unbreakable**, provided three rules are followed: the pad is truly random, the pad is at least as long as the message, and no portion of the pad is ever reused. This tool enforces the first two rules automatically. The third is up to you.

---

## Features

The suite is organised into seven tabs.

### Generate

Creates a new one-time pad — a sequence of cryptographically random digits produced by `crypto.getRandomValues()`. You choose the length (10–100,000 digits) and an optional label. Each pad gets a unique hex ID and is added to your in-memory pad library. The interface shows an estimate of how many characters of message text the pad can cover (one character uses two pad digits).

### Encrypt

Select a pad from the library, type or paste your plaintext, and click **Encrypt**. The tool converts each character to a two-digit code using a fixed 56-character alphabet (A–Z, space, 0–9, and common punctuation), then adds each code digit to the next unused pad digit modulo 10. Characters not in the alphabet are silently replaced with spaces — you'll see a warning showing exactly which characters were substituted.

The output is a string of ciphertext digits plus a pad reference (pad ID and offset). After encryption you can optionally **hide the ciphertext inside an image** directly from this tab — no passphrase needed since the OTP encryption is already applied.

### Decrypt

Paste ciphertext digits (or extract them from a stego image), select the matching pad, set the offset if needed, and click **Decrypt**. The tool reverses the modular addition to recover the plaintext.

A convenience feature auto-detects a pad reference at the end of pasted ciphertext (format `PADID-OFFSET`) and pre-fills the pad selection and offset for you.

### Pad Library

Lists all pads currently in memory with their status: **Unused**, **Partial** (some digits consumed), or **Exhausted**. Select a pad to inspect its digits — used digits are shown in red with a strikethrough, available digits in green. You can **Destroy** a pad here, which zeros out its digits in memory before removing it.

### Steganography

**Hide Pad in Image** — Select a cover image (PNG or JPEG), choose one or more pads, enter a passphrase, and the tool encrypts the pad data with AES-256-GCM (PBKDF2 key derivation, 600,000 iterations) then embeds the ciphertext into the image's least significant bits (2 bits per colour channel). The output image looks identical to the original. It must be saved and shared as PNG — lossy formats like JPEG will destroy the hidden data.

**Extract Pad from Image** — Load a stego image, enter the passphrase, and the pads are decrypted and loaded into your library.

The capacity display shows how much data the cover image can hold based on its pixel dimensions.

### Export / Import

**Export** — Encrypts your entire pad library with AES-256-GCM (passphrase-derived key, 600,000 PBKDF2 iterations) and downloads it as a single binary file. The filename is randomised with an innocuous extension (`.bak`, `.dat`, `.tmp`, etc.) so it doesn't reveal its purpose.

**Import** — Load an exported file and enter the passphrase to decrypt and add the pads to your library. Duplicates (same pad ID) are skipped.

**Save Updated Pads** — After importing and then using pads to encrypt messages, this button re-exports all pads using the same passphrase and filename from your last import. Use this to save the updated pad state (with used digits marked) so both parties stay in sync.

### Reference

Shows the full character encoding table — the two-digit code assigned to each of the 56 supported characters. Also includes a brief explanation of how OTP encryption and decryption work, with a worked example.

---

## Supported Characters

The alphabet covers 56 characters, each mapped to a two-digit code (00–55):

`A B C D E F G H I J K L M N O P Q R S T U V W X Y Z [space] 0 1 2 3 4 5 6 7 8 9 . , ? ! : ; ' " - ( ) @ # & + = / \`

Anything outside this set is replaced with a space during encryption. The tool warns you when this happens and shows which characters were substituted.

---

## Typical Workflow

1. **Generate** a pad (e.g. 1,000 digits).
2. **Export** or use **steganography** to share the pad with your contact through a separate secure channel — ideally in person on a USB stick, or hidden in an image sent via a different medium than your messages.
3. **Encrypt** your message. Send the ciphertext digits (or a stego image containing them) to your contact.
4. Your contact **imports** the same pad, **decrypts** the ciphertext, and reads the message.
5. After use, **Save Updated Pads** on both sides so the used digits are tracked and never reused.
6. When a pad is exhausted, **Destroy** it and generate a new one.

---

## Security Notes

**What this tool guarantees:**

- Pads are generated with `crypto.getRandomValues()`, a cryptographically secure pseudorandom number generator provided by the browser. Rejection sampling eliminates modulo bias.
- Pad digits are consumed sequentially and never reused within a session. Used digits are visually marked and tracked in the pad state.
- All cryptographic operations (AES-GCM for export/stego, PBKDF2 for key derivation) use the Web Crypto API, not custom implementations.

**What you are responsible for:**

- **Never reuse pad digits.** If you encrypt a message, those digits are consumed. If both parties don't keep their pad state synchronised (using export/save-back), you risk reuse.
- **Pad exchange must happen through a secure, separate channel.** The security of OTP depends entirely on the secrecy of the pad. Sending the pad over the same channel as the ciphertext defeats the purpose.
- **Pads exist only in memory.** If you close the browser tab without exporting, all pads are lost. Export or embed in an image before closing.
- **Steganography is not encryption.** Hiding data in an image obscures its existence but an adversary who suspects steganography can attempt extraction. The pad data embedded via the Steganography tab is AES-encrypted, so extraction without the passphrase yields nothing. The message stego on the Encrypt tab is not additionally encrypted (the OTP ciphertext is already encrypted), so anyone who extracts the image data gets the ciphertext — but without the pad, it is meaningless.
- **Use strong passphrases** for export and steganography. The tool shows a strength meter but the minimum is 8 characters. Longer is better.
- **PNG only for stego images.** JPEG and other lossy formats alter pixel values and will corrupt the hidden data.

---

## Running the Tool

Open `otp-suite.html` in any modern browser (Chrome, Firefox, Edge, Safari). No installation, no dependencies, no build step. The file is fully self-contained — it works from a local filesystem, a USB drive, or a web server.

**Requirements:** A browser that supports the Web Crypto API (`crypto.subtle` and `crypto.getRandomValues`). All browsers released after 2015 qualify.

---

## Privacy

No data leaves your browser. There are no analytics, no tracking, no network requests (beyond loading the Tabler Icons font from a CDN for the UI icons). If you want to go fully offline, download the Tabler Icons CSS and woff2 file and update the stylesheet link to a local path.

---

## License

MIT
