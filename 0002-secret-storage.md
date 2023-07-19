# Secret Storage

The Matrix spec defines a protocol for storing and retrieving encrypted secrets in the user's Account Data.


## Key Descriptions

According to the Matrix spec, we can store information on each secret storage
key that we derive from a password, and this will enable other clients to
re-construct the key.
For reference, see [11.13.11. Key Storage](https://spec.matrix.org/v1.6/client-server-api/#key-storage).

The shape of the `Key Description` structure depends on the algorithm that was
used to derive the key.
There is only one algorithm in the spec, `m.secret_storage.v1.aes-hmac-sha2`,
and it uses key descriptions with the following format:

```json
{
    "name": "(optional) Name for the key",
    "algorithm": "(required) Name of the algorithm used to derive the key -- Must be m.secret_storage.v1.aes-hmac-sha2",
    "passphrase": {
        "algorithm": "(required) Name of the password hashing algorithm -- Must be m.pbkdf2",
        "salt": "(required) The salt for the password hash",
        "iterations": "(required) The number of iterations for the password hash",
        "bits": 256,
    }
}
```

This object is stored in the user's Account Data under the key `m.secret_storage.key.[key ID]`.

We can use a similar format to describe the keys that we derive using our `bcrypt`-based
password hashing.

```json
{
    "name": "(optional) Name for the key",
    "algorithm": "org.futo.secret_storage.bcrypt"
    "passphrase": {
        "algorithm": "org.futo.bcrypt"
        "salt": "(required) The salt for the password hash",
        "iterations": "(required) The number of iterations for the password hash",
        "bits": 256,
    }
```

As above, we can store this object in the user's Account Data under the key
`m.secret_storage.key.[key ID]`.


## Deriving key ID's

The Matrix spec does not specify how a client should drive the identifier for
a key.
Presumably they intend for the key ID's to be totally random, and for clients
to tell which key is which by looking at the Key Description objects in the
user's Account Data.

However, that is not a great fit for our approach, because we generate our
keys before we obtain access to the user's Account Data.
We need a deterministic identifier that we can compute as a function of the
key itself.

We generate our key ID for a key `k` as follows:

1. Let `h = SHA256(k)`
2. Let `s = base64(h)`
3. Let `id = s[:16]`

In other words, the key ID is the first 16 characters of the SHA256 hash of
the key, when encoded in base64.
Or, put another way, it is the first 12 raw bytes of the SHA256 hash, encoded
into 16 bytes of base64.


### Security Analysis
Normally, it is considered quite risky to store in plaintext any data that
is derived directly from a secret key. 
Indeed, this opens up the possibility that an adversary could launch some
sort of brute force attack, possibly using some sort of time-memory trade-off
such as rainbow tables, to eventually recover the key.

However, in this case we have a reasonable argument for the security of
our scheme:

1. If the SHA256 hash function is preimage resistant, then the adversary
   cannot simply work backwards from the hash to recover the key.
2. A brute-force dictionary attack on the key itself is not feasible
   because the key is long (256 bits) and pseudorandom.
3. The adversary could attempt to dictionary attack the password, using
   the hash in the key ID to tell when he has found the correct password.
   In this case our security depends on the computational difficulty of
   the `bcrypt` hash; our security is no worse than any other system that
   uses bcrypt to hash passwords with unique salts for each user.


# Storing Secret Storage Keys in the Secret Storage

The Matrix spec does not provide a standard way to store the actual bytes
of the secret storage keys themselves.

We can do this easily using the existing Secret Storage facility.
The Matrix Secret Storage is a simple key-value store, where both keys
and values are strings.
Each value is stored encrypted under one or more encryption keys.

To disambiguate between key/value keys and encryption keys, Matrix also
refers to the key/value key as the "type" of the secret.

To store a key with ID `keyID`, we first encode the raw bytes of the
secret key as base64, and then we store the base64 string in Secret
Storage for the type `org.futo.ssss.key.[keyID]`.


