PAM-PKCS\#11 Login Tools
========================

Notes from Levitator1
----------------------

THIS IS BROKEN. GO AWAY.

Ok, so the funadmental problem to address here is an issue of certificate private key validation.
The upstream code attempts to accomplish this by signing random data using the Cryptoki/PKCS11 library.
And then it tries to validate that signature using OpenSSL, but on Debian, using ECDSA/P-384 certificates,
OpenSSL wigs out and issues some nonsensical complaint about a missing PEM header. I assumed that this was
some sort of incompatibility between Cryptoki and OpenSSL, so I set about implementing the signature generation
using OpenSSL, so that the formats would match, but OpenSSL now complains about the same thing attempting to
sign. Furthermore, it seems like it's going to be a pain figuring out how to get OpenSSL to reference the 
hardware PKCS11 private key, which should be possible...

- Openssl keeps balking and complaining about PEM format still, similar to what it does when attempting to validate the signature.
- Looks like maybe OpenSSL is being initialized wrong, or there is some incompatibility with the Debian Buster environment
- Have given up messing with this for now and will just use the pam-p11 package from Debian instead. It lacks CA verification, but it's not important
	and it works out-of-the-box

The other branch I added (encryption_test, or something) attempts to get around this problem by using round-trip encryption instead of signing.
I didn't finish that, and suspect it will probably end up suffering from the same problems.


Description
-----------

This Linux-PAM login module allows a X.509 certificate based user login.
The certificate and its dedicated private key are thereby accessed by
means of an appropriate PKCS\#11 module. For the verification of the
users' certificates, locally stored CA certificates as well as either
online or locally accessible CRLs are used.

Detailed information about the Linux-PAM system can be found in [The
Linux-PAM System Administrators'
Guide](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_SAG.html),
[The Linux-PAM Module Writers'
Guide](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_MWG.html)
and [The Linux-PAM Application Developers'
Guide](http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_ADG.html)
The specification of the Cryptographic Token Interface Standard
(PKCS\#11) is available at [PKCS\#11 - Cryptographic Token Interface
Standard](https://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html).

PAM-PKCS\#11 package provides:

* A PAM module able to:
  * Use certificates to get user credentials
  * Deduce a login based on provided certificate
* Several tools:
  * Standalone cert-to-login finder tool
  * Certificate contents viewer
  * Card Event status monitor, to trigger actions on card insert/removal

You can read the online [PAM-PKCS\#11 User
Manual](http://opensc.github.io/pam_pkcs11/doc/pam_pkcs11.html) to know
how to install, configure and use this software.

### PKCS\#11 Module Requirements

The PKCS\#11 modules must fulfill the requirements given by the RSA
Asymmetric Client Signing Profile, which has been specified in the
 [PKCS\#11: Conformance Profile
Specification](http://www.rsa.com/rsalabs/node.asp?id=2133) by RSA
Laboratories.

### User Matching

To map the ownership of a certificate into a user login, pam-pkcs11 uses
the concept of *mapper* that is, a list of configurable, stackable
list of dynamic modules, each one trying to do a specific cert-to-login
maping. Several mappers are provided:

* the common name of the subject matches the login name
* the unique identifier of the subject matches the login name
* the user part of an e-mail subject alternative name extension matches the login name
* the Microsoft universal principal name extension matches the login name
* etc...(see documentation on provided mappers)

Many mappers may use also a *mapfile* to translate Certificate
contents to a login name.

Download
--------

* [pam\_pkcs11-x.y.z.tar.gz](http://sourceforge.net/projects/opensc/files/pam_pkcs11/)

Packages for [various Linux
distributions](https://repology.org/metapackage/pam-pkcs11) are
available through the their standard package management system.

Installation
------------

Unpack the archive, configure, compile and install it:

```sh
tar xvzf pkcs11_login-X.Y.Z.tar.gz
cd pkcs11_login-X.Y.Z
./configure
make
sudo make install
```

If you want to use [cURL](http://curl.haxx.se/libcurl/) instead of
our native URI-functions for downloading CRLs, use `./configure --with-curl`

However, up to now cURL is not able to handle binary LDAP replies and
thus CRL download might not work for all LDAP URIs.

Next, you have to create the needed openssl-hash-links.

```sh
make_hash_link.sh ${path to the directory with the CA certificates}
make_hash_link.sh ${path to the directory with the CRLs}
```

Configuration
-------------

See [PAM-PKCS\#11 User
Manual](http://opensc.github.io/pam_pkcs11/doc/pam_pkcs11.html) to
configure and set up pam\_pkcs11.

See [PAM-PKCS\#11 Mappers
API](http://opensc.github.io/pam_pkcs11/doc/mappers_api.html) to get
advanced information on mappers (mainly for developers).

Documentation
-------------

* Online Manuals
* [PAM-PKCS\#11 User Manual](http://opensc.github.io/pam_pkcs11/doc/pam_pkcs11.html)
* [PAM-PKCS\#11 Mappers API Reference](http://opensc.github.io/pam_pkcs11/doc/mappers_api.html)
* [TODO](https://raw.github.com/OpenSC/pam_pkcs11/master/TODO) file (outdated)
* Man pages
  * [`pam_pkcs11(8)`](https://linux.die.net/man/8/pam_pkcs11)
  * [`card_eventmgr(1)`](https://linux.die.net/man/1/card_eventmgr)
  * [`pkcs11_eventmgr(1)`](https://linux.die.net/man/1/pkcs11_eventmgr)
  * [`pklogin_finder(1)`](https://linux.die.net/man/1/pklogin_finder)
  * [`pkcs11_inspect(1)`](https://linux.die.net/man/1/pkcs11_inspect)

Contact
-------

[Get involved](https://github.com/OpenSC/pam_pkcs11/issues)
in development! All comments, suggestions and bug reports are welcome.
