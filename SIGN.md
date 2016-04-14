# File Signing/Timestamping Documentation

This documentation is for the File Signing/Timestamping webpage `sign.html`. It describes its functionality, usage, technical details, and code.

## Functionality

The webpage allows the user to timestamp a file on the blockchain, using either the hash of the file or the hash of a PGP signature of the file (which can be created locally in the browser using the webpage). Users without a PGP keypair can generate one (also client-side) using the webpage.

By timestamping a file's hash on the blockchain, anyone can later verify that the file's hash (and therefore the file) existed at the point it time when it was timestamped. Since the only thing stored on the blockchain is the hash of the file, the actual contents of the file are not revealed. If the PGP signature of the file is used instead of the file itself as input for the hash function, then it is not only verifiable that the file existed at the time that it was timestamped, but also that owner of the private PGP key signed the file at the time that it was timestamped. This can then potentially be used to prove ownership/creation of the file. For more information with regard to the timestamping part of this tool, please see [the Proof of Existence about page](https://proofofexistence.com/about).

Files are signed and hashed within the browser. The only data that goes outside of your computer is the hash of the file/signature, which is then posted to the blockchain as metadata in a small Bitcoin transaction, paid for by the user. The data is sent to [Proof of Existence](https://proofofexistence.com), which does the work required to place the file's hash on the blockchain.

In addition, note that all generated signatures and keypairs can be easily downloaded to your computer as a file with the click of a link.

## Usage

It is very simple to use this webpage. To begin, simply click the "Choose file" button and select a file. The "PGP Signing" and "Blockchain Timestamping" sections will be enabled. To sign the selected file, input your private PGP key and its passphrase if necessary. You can also generate a new PGP private/public keypair if you do not have one already by clicking "Generate a new private/public keypair to use" and then filling out the fields required to create your new key. Then choose either *Raw* or *Base64* to encode the contents. Choosing *Raw* to use the raw contents of the file for the PGP signature is usually best, but in order to preserve the contents of the message next to the signature as Base64 characters and not raw data, choose *Base64*. The signature will be generated in a textarea a few seconds after the user presses the "Generate Signature" button. To timestamp a file or the PGP signature of the file, choose the source for the hash and click "Generate Timestamp". The generated hash, in addition to a Bitcoin address and its QR code should be presented to you. Pay the specified amount of satoshi so Proof of Existence can generate the timestamp on the blockchain. After the specified amount has been paid, hit "Check Payment Status". If the correct amount has been paid, you should be presented with a message confirming that your file has been timestamped on the blockchain (or that it will be shortly when it is confirmed).

## Warnings

Don't be worried if your computer freezes for a little while when generating a PGP keypair or signature! These processes both take anywhere from a few seconds to a few minutes to do the cryptography magic.

## Technical Explanation

Note that much of the following information is based on text from [the Proof of Existence about page](https://proofofexistence.com/about). For more technical information, you can check there.

The document or PGP signature of the document is certified by embedding its SHA256 digest in the bitcoin blockchain. This is done by generating a special bitcoin transaction that encodes/contains the hash via an OP_RETURN script. This is a bitcoin scripting opcode that marks the transaction output as provably unspendable and allows a small amount of data to be inserted. In this case, this information is the document's hash prefixed by a marker to identify all of Proof of Existence's transactions.

Once the transaction is confirmed, the document or its PGP signature (and therefore also the document) is permanently certified and proven to exist at least as early as the time the transaction was confirmed. If the document hadn't existed at the time the transaction entered the blockchain, it would have been virtually impossible to embed its digest (or the digest of the PGP signature) in the transaction (This is because of the hash function's property of being [second pre-image resistant](https://en.wikipedia.org/wiki/Cryptographic_hash_function#Properties)). Embedding some hash and then adapting a future document to match the hash is also virtually impossible (due to the pre-image resistance of hash functions). This is why once the bitcoin blockchain confirms the transaction generated for the document, its existence is proven, permanently, with no trust required.

When the PGP signature is used as the input for the hash function, one can prove that the owner of the private PGP key signed (and therefore had possesion of) the contents of the file sometime before the generated timestamp. If this is the earliest or only proof of ownership of the file that can be found, one can infer that the private key's owner did in fact create/own the file.

PGP signatures are unique, so when the PGP singature is revealed and someone verifies that the file was signed by the private key that corresponds to the public key he/she verified it with, it is proven that the owner of the private key signed and therefore was in possession of the document at the timestamp recorded on the blockchain.

## Code

The webpage is simply an HTML document that includes all necessary CSS and Javascript within the file. All necessary CSS and JS libraries are also included in the same file. All implemented libraries are as follows:

- [jQuery](https://jquery.com)
- [Bootstrap](http://getbootstrap.com)
- [OpenPGP.js](https://openpgpjs.org)
- [SJCL](https://github.com/bitwiseshiftleft/sjcl)