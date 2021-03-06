DFU Image Generation
====================

Tools to generate and encrypt a binary image in the format used by
the RigDFU transfer tools.


Build
-----

Install dependencies:

Linux:

    sudo apt-get install build-essential libcrypto++-dev

OS X using Homebrew package installer:

    sudo brew install cryptopp

Compile:

    make


Image format
------------

* 12 byte header

    * 32 bit softdevice size, little-endian
    * 32 bit bootloader size, little-endian
    * 32 bit application size, little-endian

* 16 byte encryption IV

* 16 byte encryption tag

* Softdevice, bootloader, and application binary data, in order


Genimage
--------

The `genimage.py` tool takes one or more hex files and generates a
single firmware image containing either:

* Softdevice
* Bootloader
* Softdevice and bootloader
* Application

Which ones to include, and where each is located in the hex file, can
either be manually specified, or is automatically determined using some
heuristics for where these images are expected to be.

The created image is unencrypted and unsigned.

Note the following restrictions:

* Three regions cannot be updated through DFU and are ignored:

    * MBR (`0x0000-0x1000`)
    * Bootloader settings (`0x3fc00-0x40000`)
    * UICR (`0x10001000-0x10001100`)

* The application start address is defined by the size of the
  softdevice that is currently flashed to the chip.

* The bootloader start address is defined by the UICR registers, which
  cannot be changed.

Usage:

    usage: genimage.py [-h] [--output BIN] [--quiet] [--softdevice] [--bootloader]
                       [--application] [--softdevice-addr LOW-HIGH]
                       [--bootloader-addr LOW-HIGH] [--application-addr LOW-HIGH]
                       HEXFILE [HEXFILE ...]

    Generate images for RigDFU bootloader

    positional arguments:
      HEXFILE               Hex file(s) to load

    optional arguments:
      -h, --help            show this help message and exit
      --output BIN, -o BIN  Output file
      --quiet, -q           Print less output

    Images to include:
      If none are specified, images are determined automatically based on the
      hex file contents.

      --softdevice, -s      Include softdevice
      --bootloader, -b      Include bootloader
      --application, -a     Include application

    Image locations in the HEX files:
      If unspecified, locations are guessed heuristically. Format is LOW-HIGH,
      for example 0x1000-0x16000 and 0x16000-0x3b000.

      --softdevice-addr LOW-HIGH, -S LOW-HIGH
                            Softdevice location
      --bootloader-addr LOW-HIGH, -B LOW-HIGH
                            Bootloader location
      --application-addr LOW-HIGH, -A LOW-HIGH
                            Application location

Signimage
---------

The `signimage` tool takes an existing unencrypted image and encrypts
it with a given key.  The encryption is AES-128 in EAX mode:

* IV is generated randomly and stored in the output file.

* Additional authenticated data is the 12-byte header.

* Encryption is a single stream over all of the image data.

* The resulting EAX tag (MAC) is stored in the output file.

Usage:

    usage: ./signimage <input.bin> <output.bin> <key>
      <input.bin> must be an unencrypted image, as generated by genimage
      <key> is 32 hex characters, e.g.: 00112233445566778899aabbccddeeff

Notes
---------

If you decide not to use encryption, then set the key for the bootloader to
either all 0's or all 0xFF.  In this case, you still need to provide a binary
as produces from genimage.  The genimage tool adds some information to the
beginning of the binary that is required for the bootloader.

However, you do not sign the image.  Signing the image will cause the OTA update
to fail.
