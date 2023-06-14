# Installation<!-- omit from toc -->

- [Ubuntu 22.04](#ubuntu-2204)

## Ubuntu 22.04

Date: 06/14/2023

1. [Download latest binary releases](https://www.swift.org/download/) for Ubuntu 22.04 (and optionally the signiture file). [Link to download](https://download.swift.org/swift-5.8.1-release/ubuntu2004/swift-5.8.1-RELEASE/swift-5.8.1-RELEASE-ubuntu20.04.tar.gz)
2. Install dependencies.

    ```bash
    # Note we need `libpython3.10` instead of `libpython3.8`
    apt-get install \
            binutils \
            git \
            gnupg2 \
            libc6-dev \
            libcurl4-openssl-dev \
            libedit2 \
            libgcc-9-dev \
            libpython3.10 \
            libsqlite3-0 \
            libstdc++-9-dev \
            libxml2-dev \
            libz3-dev \
            pkg-config \
            tzdata \
            unzip \
            zlib1g-dev
    ```

3. Verify the file (need the signiture file)
   1. If you are downloading Swift packages for the first time, import the PGP keys into your keyring.
   2. Skip this step if you have imported the keys in the past.

        ```bash
        gpg --keyserver hkp://keyserver.ubuntu.com \
            --recv-keys \
            '7463 A81A 4B2E EA1B 551F  FBCF D441 C977 412B 37AD' \
            '1BE1 E29A 084C B305 F397  D62A 9F59 7F4D 21A5 6D5F' \
            'A3BA FD35 56A5 9079 C068  94BD 63BC 1CFE 91D3 06C6' \
            '5E4D F843 FB06 5D7F 7E24  FBA2 EF54 30F0 71E1 B235' \
            '8513 444E 2DA3 6B7C 1659  AF4D 7638 F1FB 2B2B 08C4' \
            'A62A E125 BBBF BB96 A6E0  42EC 925C C1CC ED3D 1561' \
            '8A74 9566 2C3C D4AE 18D9  5637 FAF6 989E 1BC1 6FEA' \
            'E813 C892 820A 6FA1 3755  B268 F167 DF1A CF9C E069'
        ```

        or:

        ```bash
        wget -q -O - https://swift.org/keys/all-keys.asc | \
        gpg --import -
        ```

   3. Refresh the keys to download new key revocation certificates, if any are available:

        ```bash
        gpg --keyserver hkp://keyserver.ubuntu.com --refresh-keys Swift
        ```

   4. Then, use the signature file to verify that the archive is intact:

        ```bash
        gpg --verify swift-<VERSION>-<PLATFORM>.tar.gz.sig
        ...
        gpg: Good signature from "Swift Automatic Signing Key #4 <swift-infrastructure@forums.swift.org>"
        ```

      You might see a warning:

        ```bash
        gpg: WARNING: This key is not certified with a trusted signature!
        gpg:          There is no indication that the signature belongs to the owner.
        ```

      This warning means that there is no path in the Web of Trust between this key and you. The warning is harmless as long as you have followed the steps above to retrieve the key from a trusted source.

4. Extract the archive with the following command:

    ```bash
    tar xzf swift-<VERSION>-<PLATFORM>.tar.gz
    ```

    This creates a `usr/` directory in the location of the archive.

5. Add the Swift toolchain to your path

    ```bash
    sudo mv /path/to/swift-<VERSION>-<PLATFORM> /opt/swift 
    echo "export PATH=/opt/swift/usr/bin:$PATH" >> ~/.bashrc
    source ~/.bashrc
    ```

6. Test if Swift is working propoerly

    ```bash
    swift --version
    swift repl
    ```
