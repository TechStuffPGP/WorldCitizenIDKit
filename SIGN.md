# File Signing/Timestamping Documentation

This documentation is for the File Signing/Timestamping webpage `sign.html`. It describes its functionality, usage, technical details, and code.

## Functionality

The webpage allows the user to timestamp a file on the blockchain, using either the hash of the file itself or the hash of a PGP signature of the file (which can be created locally in the browser using the webpage). Users without a PGP keypair can generate one (also client-side) using the webpage.

By timestamping a file's hash on the blockchain, anyone can later verify that the file's hash (and therefore the file) existed at the point it time when it was timestamped. Since the only thing stored on the blockchain is the hash of the file, the actual contents of the file are not revealed. If the PGP signature of the file is used instead of the file itself as input for the hash function, then it is not only verifiable that the file existed at the time that it was timestamped, but also that owner of the private PGP key signed the file at the time that it was timestamped. This can then potentially be used to prove ownership/creation of the file.

Files are signed and hashed within the browser. The only data that goes outside of your computer is the hash of the file/signature, which is then posted to the blockchain as the output address in an OP_RETURN script in a small Bitcoin transaction, paid for by the user. The hash is sent to our own backend API (currently hosted by David Lucid) after it is generated so that our server can do all the work required to place the hash on the blockchain.

In addition, note that all generated signatures, keypairs, and timestamp reports can be easily downloaded to your computer as text files with the click of a button.

## Usage

It is very simple to use this webpage. To begin, simply click the "Choose file" button and select a file. The "PGP Signing" and "Blockchain Timestamping" sections will be enabled.

To sign the selected file, input your private PGP key and its passphrase if necessary. You can also generate a new PGP private/public keypair if you do not have one already by clicking "Generate a new private/public keypair to use" and then filling out the fields required to create your new key. Anyway, the signature will be generated and displayed in textareas (both the raw signature encoded in Base64 as well as the ASCII-armored signature) roughly a few seconds after the user presses the "Generate Signature" button.

To timestamp a file or the PGP signature of the file, choose the source for the hash and click "Generate Timestamp". The generated hash, in addition to a Bitcoin address and its QR code should be presented to you. Pay the specified amount of satoshi so our API can generate the timestamp on the blockchain. After the specified amount has been paid, hit "Check Payment Status". If the correct amount has been paid, you should be presented with a message confirming that your file has been timestamped on the blockchain (or that it will be shortly when it is confirmed).

## Warnings

Don't be worried if your computer freezes for a little while when generating a PGP signature or keypair! These processes both take anywhere from a few seconds to a few minutes to do the cryptography magic.

Also, don't get the file/signature hashes and the Bitcoin transaction/block hashes confused. They look the same since they are all SHA256 hashes encoded as strings of 64 hexadecimal characters, but they mean very different things. The hash of your file or signature is a shorter representation of the data in the file or the signed file. The Bitcoin transaction hash identifies the transaction (on the Bitcoin blockchain) containing the hash of your file or signature. The block hash, on the other hand, identifies the block (on the Bitcoin blockchain) containing the transaction.

## Technical Explanation

The document or PGP signature of the document is certified by embedding its SHA256 hash in the Bitcoin blockchain. This is done by generating a special Bitcoin transaction that encodes/contains the hash as the output of an OP_RETURN script. This is a Bitcoin scripting opcode that marks the transaction output as provably unspendable and allows a small amount of data to be inserted instead of a Bitcoin address. In this case, this 40-byte piece of data is made up of the raw 32 bytes of the document's hash prefixed by an 8-byte marker ("WCITIZEN" encoded as ASCII) to identify all of these transactions.

Once the transaction is confirmed, the document or its PGP signature (and therefore also the document) is permanently certified and proven to exist at least as early as the time the transaction was confirmed (the timestamp of the block containing the transaction). If the document hadn't existed at the time the transaction entered the blockchain, it would have been virtually impossible to embed its hash (or the hash of the PGP signature) in the transaction (This is because of the hash function's property of [second pre-image resistance](https://en.wikipedia.org/wiki/Cryptographic_hash_function#Properties)). Embedding any random 256 bits of data and then adapting a future document to match the hash is also virtually impossible due to SHA256's property of pre-image resistance. This is why once the Bitcoin network confirms the transaction generated for the document, its existence is permanently proven with no trust required.

When the PGP signature is used as the input for the hash function, one can prove not only that the document existed at the time of the transaction's confirmation, but also that the owner of the private PGP key signed the contents of the file sometime before the generated timestamp.

Both PGP signatures and hashes are second pre-image resistant. Given the PGP signature (the hash of which is now in the blockchain), the original file, and the PGP public key, someone can cryptographically verify the signature as a valid signature of the original file by the owner of the public key's corresponding private key. In other words, the owner of the private key signed and therefore was in possession of the document at the timestamp recorded on the blockchain. If this is the earliest or only proof of ownership of the file that can be found, one can assume that the private key's owner did in fact create/own the document.

## PGP Signing

Keep the following in mind that when signing a file before timestamping it using this tool. The PGP signature is generated directly from the raw bytes of the file, and presented to you as in the textareas as both the raw signature encoded in Base64 as well as the ASCII-armored signature. When you hash the PGP signature in order to timestamp it on the blockchain, the hash is generated directly from the bytes of the raw PGP signature.

## Code

The webpage is simply an HTML document that includes all necessary CSS and Javascript within the file. All necessary CSS and JS libraries are also included in the same file. All implemented libraries are listed as follows:

- [jQuery](https://jquery.com)
- [Bootstrap](http://getbootstrap.com)
- [kbpgp](https://keybase.io/kbpgp)
- [SJCL](https://github.com/bitwiseshiftleft/sjcl)
- [QRCode.js](https://github.com/davidshimjs/qrcodejs)