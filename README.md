# Codesign
Codesigning Scripts for Zcash Foundation releases

## Quickstart
```
# Install pre-requisites
brew install  gsha256sum gnupg      # MacOS
sudo apt install sha256sum gnupg    # Linux

# Generate a full key
gpg --full-generate-key

# Sign your release binaries
./codesign.sh file1.msi file2.deb file3.dmg
# Will produce signatures.zip with the hashes and signatures
```

# Documentation

## Create a gpg private key on your release machine
First, create a GPG private key on the machine you plan to use to sign the release binaries. This is usually your build/CI machine. 
Create the keys by running
```
gpg --full-generate-key
```
The defaults for the crypto parameters are usually sufficient

### NOTE on using a passphrase
If you specify a passphrase for the key, you'll have to enter it interactively when running the script. If you're planning to do the signing as a part of your build process, you might want to not specify a passphrase for the key


## Backup your key
You can backup your private key with 
```
gpg --armor --export > pgp-public-keys.asc
gpg --armor --export-secret-keys > pgp-private-keys.asc
gpg --export-ownertrust > pgp-ownertrust.asc
```
You'll need to backup all 3 files to be able to restore them if you lose access to them.


## Export your public key
You can get your full public key by running
```
./codesign.sh --public
```
It might be a good idea put your public key as a file in your project's root directory, so users can verify the public key is coming from your project.

## Upload your Public Key to Public Keyserves
For users to find your public key, you can optionally upload it to several public key servers. 
```
./codesign.sh --upload KEY_FINGERPRINT
```
Where KEY_FINGERPRINT is your public key's fingerprint. You can get it by running `gpg --check-sigs`. For example:
```
bash:~>gpg --check-sigs
/home/adityapk00/.gnupg/pubring.kbx
---------------------------------
pub   rsa4096 2019-02-04 [SC]
      C23172D0C9569591ECEC8ECB0E1E90279521EBB4
uid           [ unknown] adityapk00 (PGP Key for zec-qt-wallet) <zcash@adityapk.com>
sig!3        0E1E90279521EBB4 2019-02-04  adityapk00 (PGP Key for zec-qt-wallet) <zcash@adityapk.com>
sub   rsa4096 2019-02-04 [E]
sig!         0E1E90279521EBB4 2019-02-04  adityapk00 (PGP Key for zec-qt-wallet) <zcash@adityapk.com>
```
The lines begining with `sig!` and `sig!3` show the fingerprint, in this case `0E1E90279521EBB4`

# Signing your releases
To sign your release binaries, run 
```
./codesign.sh file1 [file2 ...]
```

This will create a single file, `signatures.zip` which will contain the sha256 hashes of all the binaries files and a signature for each of the input files. For example:
```
./codesign.sh zecwallet-Windows-Installer.msi zecwallet-Linux.deb zecwallet-MacOS.dmg
```

# Verifying signatures

#### Import public key
Import the public key via

`gpg --import public_key.asc`

Alternatively, you can import from a keyserver using the following command:

`gpg --keyserver pool.sks-keyservers.net --recv-keys KEY_FINGERPRINT`

#### Verifying signatures

Once the key has been imported, you can run the following command to verify the signature:

`gpg --verify <filename.sig> <downloaded-filename-to-verify>`

Assuming a good signature, you will see a similar output to the below.

```
gpg --verify macOS-zecwallet-v0.6.2.dmg.sig macOS-zecwallet-v0.6.2.dmg
gpg: Signature made Wed 20 Feb 11:06:04 2019 PST
gpg:                using RSA key C23172D0C9569591ECEC8ECB0E1E90279521EBB4
gpg: Good signature from "adityapk00 (PGP Key for zec-qt-wallet) <zcash@adityapk.com>" [full]
```

### Checksums

The file `sha256sum.txt` contains the SHA256 checksums for each download. You can verify that the file you downloaded matches this checksum via the following command:

```
sha256sum macOS-zecwallet-v0.6.2.dmg
b8adcac91a138adb471e729e02696479b5680f98590003fe5bede6862a5e2d38  macOS-zecwallet-v0.6.2.dmg
```
