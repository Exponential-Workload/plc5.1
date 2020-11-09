## PLC - Pure Lua Crypto

A small collection of crpytographic functions, and related utilities, 
implemented  in pure Lua  (version 5.3 or above)

### Recent changes

December 2018

* Added XChaCha20 encryption (in plc/chacha20.lua)

October 2018

* Added SHA2-512 and optimized SHA2-256. Borrowed the core permutation code from the very nice pure_lua_SHA2 project by Egor Skriptunoff - https://github.com/Egor-Skriptunoff/pure_lua_SHA2

April 2018

* Added *Morus*, a finalist (round 4) in the CAESAR competition for authenticated encryption.

* Adding *Gimli*, cryptographic functions based on the Gimli permutation (Dan Bernstein et al., 2017, https://gimli.cr.yp.to/). *Work in progress - at the moment, only the core permutation has been implemented.*

* Added *Base85*, including the ZeroMQ variant of Ascii85 encoding.

December 2017

* Added *SipHash*, a very fast pseudorandom function (or keyed hash) 
optimized for speed on short messages. It can be used as a MAC and has
been extensively used as a robust string hash function, as a defense 
against hash-flooding DoS attacks.

August 2017

* Added *Salsa20* and the NaCl *box() / secret_box()* API, contributed 
by Pierre Chapuis - https://github.com/catwell

June 2017

* Added *MD5*.

### Objective

Collect in one place standalone implementation of well-known, and/or useful,  and/or interesting cryptographic algorithms.

Users should be able to pickup any file and just drop it in their project:

* All the files are written in pure Lua, version 5.3 and above (tested on 5.3.4). Lua 5.3 is required since bit operators and string pack/unpack are extensively used.

* The files should not require any third-party library or C extension beyond the standard Lua 5.3 library. 

* The files should not define any global. When required, they should just return a table with the algorithm's functions and constants.

Contributions, fixes, bug reports and suggestions are welcome.

What this collection is *not*:

* a complete, structured cryptographic library - no promise is made about consistent API structure and documentation. This is not a library - just a collection of hopefully useful snippets of crypto source code. 

* high performance, heavy-duty cryptographic implementations -- after all, this is *pure* Lua...  :-)

*  memory-efficient implementations (see above)

*  memory-safe algorithms  -- Lua immutable strings are used and garbage-collected as needed. No guarantee is made that information, and in particular key material, is properly erased when no longer needed or do not leak.


### Functions

Encryption

* Morus, an *amazingly* fast (see performance below) authenticated encryption algorithm with associated data (AEAD). Morus is a finalist (round 4) in the [CAESAR](http://competitions.cr.yp.to/caesar-submissions.html) competition - see https://personal.ntu.edu.sg/wuhj/research/caesar/caesar.html

* NORX, a *very* fast authenticated encryption algorithm with associated data (AEAD). NORX is a 3rd-round candidate to [CAESAR](http://competitions.cr.yp.to/caesar.html). This Lua code implements the default NORX 64-4-1 variant (state is 16 64-bit words, four rounds, no parallel execution, key and nonce are 256 bits) - see https://norx.io/

* NORX32, a variant of NORX intended for smaller architectures (32-bit and less). Key and nonce are 128 bits. (Note that this NORX32 Lua implementation is half as fast as the default 64-bit NORX. It is included here only for compatibility with other implementations - In Lua, use the default NORX implementation!)

* Rabbit, a fast stream cipher, selected in the eSTREAM portfolio along with Salsa20, and defined in [RFC 4503](https://tools.ietf.org/html/rfc4503) (128-bit key, 64-bit IV - see more information and links in rabbit.lua)

* Chacha20, Poly1305 and authenticated stream encryption, as defined in [RFC 7539](https://tools.ietf.org/html/rfc7539), and XChacha20 stream encryption (Chacha20 with a 24-byte nonce)

* Salsa20, a fast encryption algorithm and the NaCl secretbox() API for authenticated encryption (with Salsa20 and Poly1305 - see box.lua)
Salsa20, Poly1305 and the NaCl library have been designed by Dan Bernstein, Tanja Lange et al.  http://nacl.cr.yp.to/.

* RC4 - for lightweight, low strength encryption. Can also be used as a simple pseudo-random number generator.

Public key

* Elliptic curve cryptography based on curve ec25519 by Dan Bernstein, Tanja Lange et al.,  http://nacl.cr.yp.to/.  File ec25519.lua includes the core scalar multiplication operation. File box.lua includes the NaCl box() API which combines ECDH key exchange and authenticated encryption.

Hash

* Blake2b - Blake was a final round candidate in the NIST SHA-3 selection process.  Blake2b is an improved version of Blake. See https://blake2.net/. It has been specified in [RFC 7693](https://tools.ietf.org/html/rfc7693)

* SHA2 cryptographic hash family (sha256 and sha512)

* SHA3 cryptographic hash family (formerly known as Keccak - 256-bit and 512-bit versions)

* SipHash, a keyed hash function family optimized for speed on short messages, by Jean-Philippe Aumasson and Dan Bernstein. The variant implemented here is the default SipHash-2-4.

* MD5, as specified in [RFC 1321](https://tools.ietf.org/html/rfc1321)

* Non-cryptographic checksums (CRC-32, Adler-32), ...

Some (un)related utilities: 

* Base64, Base58, Base85 (Z85, the ZeroMQ variant of Ascii85)  and Hex encoding/decoding.

### In the future...

Implementations that may come some day:

* Ed25519 signature 

* better documentation in each file :-)

### Performance

These crude numbers give an idea of the relative performance of the algorithms. 
They correspond to the encryption or the hash of a 10 MB string (10 * 1024 * 1024 bytes). 

They have been collected on a laptop with Linux x86_64,  CPU i5 M430 @ 2.27 GHz. Lua version is 5.3.4 (ELF 64 bits) - see file 'test_perf.lua'; uncomment whatever test you want to run at the end. 

```
Plain text size: 10 MBytes. Elapsed time in seconds

Encryption
	- rc4 raw                 7.4  
	- rabbit                  4.7  
	- xtea ctr               11.0  
	- chacha20                7.9  
	- salsa20                 8.0  
	- norx aead               4.5  
	- norx32 aead             9.2  
	- morus aead              1.7

Hash
	- md5                     3.7  
	- sha2-256                9.1  
	- sha2-512                6.4  
	- sha3-256               23.2  
	- sha3-512               43.0  
	- blake2b-512             9.4  
	- blake2b-256             9.3  
	- poly1305 hmac           1.2  

	- adler-32                1.3  
	- crc-32                  1.8  

Elliptic curve (ec25519)
	- scalarmult (100 times) 18.9

```

### Test vectors, tests, and disclaimer

Some simplistic tests can be run (test_all.lua). Individual test files are provided in the 'test' directory. 

The implementations should pass the tests, but beyond that, there is no guarantee that these implementations conform to anything  :-)  -- Use at your own risk!


### License and credits

All the files included here are distributed under the MIT License (see file LICENSE)

The salsa20 and box/secretbox implementations are contributed by Pierre Chapuis - https://github.com/catwell

The sha2-256 and sha2-512 core permutation has been borrowed from Egor Skriptunoff's pure_lua_SHA2 project - https://github.com/Egor-Skriptunoff/pure_lua_SHA2





