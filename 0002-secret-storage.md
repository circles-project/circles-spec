# Secret Storage

The Matrix spec defines a protocol for storing and retrieving encrypted secrets in the user's Account Data.


## Getting the Default Key ID

The user may have more than one key that they have used to encrypt secrets in
their secret storage.



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

We can use a similar format to describe the keys that we derive using our `BS-SPEKE`-based
password hashing.

```json
{
    "name": "(optional) Name for the key",
    "algorithm": "(required) Must be m.secret_storage.v1.aes-hmac-sha2",
    "passphrase": {
        "algorithm": "(required) Must be org.futo.bsspeke-ecc"
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
We need a deterministic identifier that we can compute deterministically,
so we get the same key id every time we re-generate the same key.

To do this, we can use the same facility that we use to generate the key itself.

1. Get 32 bytes by calling the BS-SPEKE client's `generate_hashed_key()` function,
   with the label `matrix_ssss_key_id`.
2. Take the first 16 bytes, and hex encode them into a string of 32 hex characters.
   This 32-character string is our key id.


## Storing Secret Storage Keys in the Secret Storage

The Matrix spec does not provide a standard way to store the actual bytes
of the secret storage keys themselves.

We can do this easily using the existing Secret Storage facility.
The Matrix Secret Storage is a simple key-value store, where keys
and are strings and values can be anything representable as JSON.
Each value is stored encrypted under one or more encryption keys.

To disambiguate between key/value keys and encryption keys, Matrix also
refers to the key/value key as the "type" of the secret.

A key with ID `keyID` is stored in Secret Storage with the type
`org.futo.ssss.key.[keyID]`.

## Upgrading Secret Storage Keys

When the user changes their BS-SPEKE password, we should check whether
the user's current default secret storage key was derived from a BS-SPEKE
password.
If so, then we should also replace the old secret storage key with a new
secret storage key derived from the new BS-SPEKE password.

The sequence of steps goes like this:

1. Generate the new key and key id from the BS-SPEKE client
2. Register the new key in secret storage, by saving a Key Description
   object for it in the user's (unencrypted) Account Data.
3. Save the bytes of the new key, encrypted under the old key, in the
   Secret Storage.
4. Set the default key id to be the new key id.
5. Save the bytes of old key, encrypted under the new key, in the
   Secret Storage.
