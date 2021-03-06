jot
===

jot is a Python implementation of the *draft* [JSON Web Token (JWT)](http://tools.ietf.org/html/draft-jones-json-web-token-07) specification.

It supports signing through the *draft* [JWS](http://tools.ietf.org/html/draft-jones-json-web-signature-04) specification.  Only HMAC SHA-256/384/512 are
supported as of yet.

It will eventually support encryption through JWE, following a similar API.


Plaintext JWT
-------------

Plaintext JWTs are neither signed nor encrypted and take a JSON-compatible
object as the sole argument.

    >>> from jot import jwt
    >>> msg = jwt.encode({'status': 'ready'})
    >>> msg
    'eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdGF0dXMiOiJyZWFkeSJ9.'
    >>> jwt.decode(msg)
    {'headers': {u'alg': u'none', u'typ': u'JWT'}, 'valid': True, 'payload':
    {u'status': u'ready'}}


Signed JWT (JWS)
----------------

For encoding, you need to provide an object representing your desired
algorithm along with a key and, optionally, a key id for the header.

    >>> from jot import jwt, jws
    >>> msg = jwt.encode({'status': 'ready'}, signer=jws.HmacSha(
    bits=256, key='verysecret', key_id='client1'))
    >>> msg
    'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImNsaWVudDEifQ.eyJzdGF0dXM
    iOiJyZWFkeSJ9.DcKKQXXUjGP7pape8BgQ3AcQSPH8toWFLY2woIVUZ-w'

To decode and verify, you must pass a signer object for every possible
expected algorithm.  This may only be one.  You can pass a key directly to
the signer object if you expect only a particular one:

    >>> jwt.decode(msg, signers=[jws.HmacSha(bits=256, key='verysecret')])
    {'headers': {u'alg': u'HS256', u'typ': u'JWT', u'kid': u'client1'},
    'valid': True, 'payload': {u'status': u'ready'}}

If you expect any of several keys, you can pass a dictionary of key_id -> key
mappings.  decode() will use the 'kid' (key id) header to choose the correct
one.

    >>> jwt.decode(msg, signers=[jws.HmacSha(bits=256, keydict={'client1':
    'verysecret', 'client2': 'evensecreter'})])
    {'headers': {u'alg': u'HS256', u'typ': u'JWT', u'kid': u'client1'},
    'valid': True, 'payload': {u'status': u'ready'}}

An invalid key, or a key id not being found in the key dictionary, will flip
the 'valid' attribute to False:

    >>> jwt.decode(msg, signers=[jws.HmacSha(bits=256, keydict={'client1':
    'notverysecret', 'client2': 'evensecreter'})])
    {'headers': {u'alg': u'HS256', u'typ': u'JWT', u'kid': u'client1'},
    'valid': False, 'payload': {u'status': u'ready'}}
    >>> jwt.decode(msg, signers=[jws.HmacSha(bits=256, keydict={'client10':
    'verysecret', 'client2': 'evensecreter'})])
    {'headers': {u'alg': u'HS256', u'typ': u'JWT', u'kid': u'client1'},
    'valid': False, 'payload': {u'status': u'ready'}}

The headers and payload should not be trusted if valid is False, but they are
provided for informational purposes.
