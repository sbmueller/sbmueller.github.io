---
title: "I have nothing to hide anyway"
date: 2017-09-28T14:31:13+02:00
tags: ["opinion"]
draft: false
---

This is what I often hear when I talk to people about encrypted communication.
Generally, the complexity of communicating securely is extremely overrated, and
people seem to avoid going through the supposed hassle because they don't
exchange sensitive data over the internet.

With this article, I want to arise awareness of secure communication, and in
particular of PGP. Generally speaking, in cryptography the strenght of an
encryption is directly proportional with the mean time an attacker would need
to crack the cipher. This time should be *greater* than the timeframe in which
the information could be valuable for an attacker. In most cases of everyday
communication, people correctly assume their information is of *no* value for
an attacker and therefore conclude that no encryption should be used. But there
are two arguments against this logic: Firstly, encrypting is *easy*! With
technology like PGP (which still requires one initial configuration), everybody
can communicate securely. Second, it's *free*. Nobody charges you for
encryption technology since most of it is developed in a free software context.
Third, everybody should protest governmental surveillance and increasing
privacy penetration.

## PG-What?  Pretty good privacy (PGP) is a very proven technology to encrypt
emails. It generally works with asynchronous encryption, making sure only the
intended receipient of a message is able to decrypt the message. To enable
this, every user of PGP publishes a _public_ key which is used by the sender to
encrypt a message. After that, the message can only be decrypted with the
_private_ key of the receipient, which is *NOT* publicly available. For more
details, [Wikipedia](https://en.wikipedia.org/wiki/Pretty_Good_Privacy) is a
good resource.

For further steps, there are good resources for [Mac](https://gpgtools.org) and
[Mozilla Thunderbird](https://www.enigmail.net/index.php/en/). And of course,
my public key can be found
[here](http://pgp.mit.edu/pks/lookup?op=vindex&search=0x9FFBD55DDC2AA3EE).
