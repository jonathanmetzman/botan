Hash Functions and Checksums
=============================

Hash functions are one-way functions, which map data of arbitrary size to a
fixed output length. Most of the hash functions in Botan are designed to be
cryptographically secure, which means that it is computationally infeasible to
create a collision (finding two inputs with the same hash) or preimages (given a
hash output, generating an arbitrary input with the same hash). But note that
not all such hash functions meet their goals, in particular MD4 and MD5 are
trivially broken. However they are still included due to their wide adoption in
various protocols.

The class :cpp:class:`HashFunction` is defined in `botan/hash.h`.

Using a hash function is typically split into three stages: initialization,
update, and finalization (often referred to as a IUF interface). The
initialization stage is implicit: after creating a hash function object, it is
ready to process data. Then update is called one or more times. Calling update
several times is equivalent to calling it once with all of the arguments
concatenated. After completing a hash computation (eg using ``final``), the
internal state is reset to begin hashing a new message.

.. cpp:class:: HashFunction

  .. cpp:function:: static std::unique_ptr<HashFunction> create(const std::string& name)

    Return a newly allocated hash function object, or nullptr if the
    name is not recognized.

  .. cpp:function:: static std::unique_ptr<HashFunction> create_or_throw(const std::string& name)

    Like ``create`` except that it will throw an exception instead of
    returning nullptr.

  .. cpp:function:: size_t output_length()

    Return the size (in *bytes*) of the output of this function.

  .. cpp:function:: void update(const uint8_t* input, size_t length)

    Updates the computation with *input*.

  .. cpp:function:: void update(uint8_t input)

    Updates the computation with *input*.

  .. cpp:function:: void update(const std::vector<uint8_t>& input)

    Updates the computation with *input*.

  .. cpp:function:: void update(const std::string& input)

    Updates the computation with *input*.

  .. cpp:function:: void final(uint8_t* out)

    Finalize the calculation and place the result into ``out``.
    For the argument taking an array, exactly ``output_length`` bytes will
    be written. After you call ``final``, the algorithm is reset to
    its initial state, so it may be reused immediately.

  .. cpp:function:: secure_vector<uint8_t> final()

    Similar to the other function of the same name, except it returns
    the result in a newly allocated vector.

  .. cpp:function:: secure_vector<uint8_t> process(const uint8_t in[], size_t length)

     Equivalent to calling ``update`` followed by ``final``.

  .. cpp:function:: secure_vector<uint8_t> process(const std::string& in)

     Equivalent to calling ``update`` followed by ``final``.

  .. cpp:function:: std::unique_ptr<HashFunction> new_object()

     Return a newly allocated HashFunction object of the same type as this one.

  .. cpp:function:: std::unique_ptr<HashFunction> copy_state()

     Return a newly allocated HashFunction object of the same type as this one,
     whose internal state matches the current state of this.

Code Example
------------

Assume we want to calculate the SHA-256, SHA-384, and SHA-3 hash digests of the STDIN stream using the Botan library.

.. literalinclude:: /../src/examples/hash.cpp
   :language: cpp

Available Hash Functions
------------------------------

The following cryptographic hash functions are implemented. If in doubt,
any of SHA-384, SHA-3, or BLAKE2b are fine choices.

BLAKE2b
^^^^^^^^^

Available if ``BOTAN_HAS_BLAKE2B`` is defined.

A recently designed hash function. Very fast on 64-bit processors. Can output a
hash of any length between 1 and 64 bytes, this is specified by passing a value
to the constructor with the desired length.

Named like "Blake2b" which selects default 512-bit output, or as
"Blake2b(256)" to select 256 bits of output.

GOST-34.11
^^^^^^^^^^^^^^^

.. deprecated:: 2.11

Available if ``BOTAN_HAS_GOST_34_11`` is defined.

Russian national standard hash. It is old, slow, and has some weaknesses. Avoid
it unless you must.

.. warning::
   As this hash function is no longer approved by the latest Russian standards,
   support for GOST 34.11 hash is deprecated and will be removed in a future
   major release.

Keccak-1600
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_KECCAK`` is defined.

An older (and incompatible) variant of SHA-3, but sometimes used. Prefer SHA-3 in
new code.

MD4
^^^^^^^^^

An old and now broken hash function. Available if ``BOTAN_HAS_MD4`` is defined.

.. warning::
   MD4 collisions can be easily created. There is no safe cryptographic use
   for this function.

.. warning::
   Support for MD4 is deprecated and will be removed in a future major release.

MD5
^^^^^^^^^

An old and now broken hash function. Available if ``BOTAN_HAS_MD5`` is defined.

.. warning::
   MD5 collisions can be easily created. MD5 should never be used for signatures.

RIPEMD-160
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_RIPEMD160`` is defined.

A 160 bit hash function, quite old but still thought to be secure (up to the
limit of 2**80 computation required for a collision which is possible with any
160 bit hash function). Somewhat deprecated these days. Prefer SHA-2 or SHA-3
in new code.

SHA-1
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_SHA1`` is defined.

Widely adopted NSA designed hash function. Use SHA-2 or SHA-3 in new code.

.. warning::

   SHA-1 collisions can now be created by moderately resourced attackers. It
   must never be used for signatures.

SHA-256
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_SHA2_32`` is defined.

Relatively fast 256 bit hash function, thought to be secure.

Also includes the variant SHA-224. There is no real reason to use SHA-224.

SHA-512
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_SHA2_64`` is defined.

SHA-512 is faster than SHA-256 on 64-bit processors. Also includes the
truncated variants SHA-384 and SHA-512/256, which have the advantage
of avoiding message extension attacks.

SHA-3
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_SHA3`` is defined.

The new NIST standard hash. Fairly slow.

Supports 224, 256, 384 or 512 bit outputs. SHA-3 is faster with
smaller outputs.  Use as "SHA-3(256)" or "SHA-3(512)". Plain "SHA-3"
selects default 512 bit output.

SHAKE (SHAKE-128, SHAKE-256)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_SHAKE`` is defined.

These are actually XOFs (extensible output functions) based on SHA-3, which can
output a value of any byte length. For example "SHAKE-128(1024)" will produce
1024 bits of output. The specified length must be a multiple of 8.

SM3
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_SM3`` is defined.

Chinese national hash function, 256 bit output. Widely used in industry there.
Fast and seemingly secure, but no reason to prefer it over SHA-2 or SHA-3 unless
required.

Skein-512
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_SKEIN_512`` is defined.

A contender for the NIST SHA-3 competition. Very fast on 64-bit systems.  Can
output a hash of any length between 1 and 64 bytes. It also accepts an optional
"personalization string" which can create variants of the hash. This is useful
for domain separation.

To set a personalization string set the second param to any value,
typically ASCII strings are used. Examples "Skein-512(256)" or
"Skein-512(384,personalization_string)".

Streebog (Streebog-256, Streebog-512)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_STREEBOG`` is defined.

Newly designed Russian national hash function. Due to use of input-dependent
table lookups, it is vulnerable to side channels. There is no reason to use it
unless compatibility is needed.

.. warning::
   The Streebog Sbox has recently been revealed to have a hidden structure which
   interacts with its linear layer in a way which may provide a backdoor when
   used in certain ways. Avoid Streebog if at all possible.

Whirlpool
^^^^^^^^^^^^^^^

Available if ``BOTAN_HAS_WHIRLPOOL`` is defined.

A 512-bit hash function standardized by ISO and NESSIE. Relatively slow, and due
to the table based implementation it is potentially vulnerable to cache based
side channels.

Hash Function Combiners
---------------------------

These are functions which combine multiple hash functions to create a new hash
function. They are typically only used in specialized applications.

Parallel
^^^^^^^^^^^^^

Available if ``BOTAN_HAS_PARALLEL_HASH`` is defined.

Parallel simply concatenates multiple hash functions. For example
"Parallel(SHA-256,SHA-512)" outputs a 256+512 bit hash created by hashing the
input with both SHA-256 and SHA-512 and concatenating the outputs.

Note that due to the "multicollision attack" it turns out that generating a
collision for multiple parallel hash functions is no harder than generating a
collision for the strongest hash function.

Comp4P
^^^^^^^^^^^^^

Available if ``BOTAN_HAS_COMB4P`` is defined.

This combines two cryptographic hashes in such a way that preimage and collision
attacks are provably at least as hard as a preimage or collision attack on the
strongest hash.

Checksums
----------------

.. note:: Checksums are not suitable for cryptographic use, but can be used for
          error checking purposes.

Adler32
^^^^^^^^^^^

Available if ``BOTAN_HAS_ADLER32`` is defined.

The Adler32 checksum is used in the zlib format. 32 bit output.

CRC24
^^^^^^^^^^^

Available if ``BOTAN_HAS_CRC24`` is defined.

This is the CRC function used in OpenPGP. 24 bit output.

CRC32
^^^^^^^^^^^

Available if ``BOTAN_HAS_CRC32`` is defined.

This is the 32-bit CRC used in protocols such as Ethernet, gzip, PNG, etc.
