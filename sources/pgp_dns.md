<!--
REF: https://www.gushi.org/make-dns-cert/HOWTO.html
-->
<!-- Indent
<div style="padding-left: 20px;">
TEXT1
</br></br>
TEXT2
</br></br></div>
-->
## The complete guide to publishing PGP keys in DNS

### Introduction

Publishing PGP keys is a pain. There are many disjoint keyservers, three or four networks of which, which do (or don't) share information with each other. Some are corporate, some are private. And it's a crapshoot as to whose key is going to be on which, or worse, which will have the latest copy of a person's key.

For a long time, GPG has had a way to publish keys in DNS, but it hasn't been well documented. This document hopes to change that.

* After reading this, you should:

* Know the three ways to publish a key

* Have at least a couple tools to do so

* Have learned a bit more about DNS

The target audience for this guide is a technical one. It's expected you understand what DNS is, and what an RFC and a resource record is.

There are three ways to publish a PGP key in DNS. Most modern versions of GPG can retrieve from all three, although it's not enabled by default.

There are no compile-time options you need to enable it, and it's simple to turn on.

Of the three key-publishing methods, there are two that you probably shouldn't use at the same time, and there are advantages and disadvantages to each, which I hope to outline below, both in general and for each method.

### Advantages to DNS publishing of your keys

It's universal. Your DNS is your own, and you don't have to worry about which network of vastly-disconnected keyservers is caching your key.

Using DNS does not stop you from publishing via other means.

If you run an organization, you can easily publish all your employee-keys via this method, and in the same step, define a signing-policy, such that a person need only assign trust to your organization's "keysigning key" (or the CEO's key, or the CTO's), without the trouble of running a keyserver.

DNSSEC can be (somewhat) used as an additional trust-path vector. More on this in the notes at the bottom.

You do not have to be searching DNS for keys in order to publish. On the same note, you do not have to be publishing in this manner to search for others there.

### Disadvantages to DNS publishing

If you don't control your own DNS (or have a good relationship with your DNS admin), this isn't going to be as easy or even possible. Ideally, you want to be running BIND.

With two of the three methods listed here, you're going to need to be able to put a CERT record into your DNS. Most web-enabled DNS tools probably will not give you this ability. The third uses TXT records, which SPF has caused to be fairly universal in web-interfaces. However, it's also the least standards-defined of the three.

Using at least some of these methods, it's not always a "set it and forget it" procedure. You may need to periodically re-export your key and re-publish it, especially if you gain new signatures.

Using some of these methods, you're going to be putting some pretty large, pretty unwiedly lines in your DNS zones.
Not everyone will easily be able to retrieve them, but again, you can still publish other ways.

Using some of these methods, DNS is just a means to an end: you still need to publish your key elsewhere, like a webpage, and the DNS records just point at it.

Initial verifications of most of these seem to imply that only DSA keys are supported, although I welcome feedback. It seems the community is trying to get RSA keys to make a comeback. They're the only type supported by the gpg2.0 card, and they are the default keytype. There was a while where they weren't, though. Since writing this document, I've discovered that "new" RSA keys work, but ancient RSA keys with no subkeys tend to misbehave.

# Turning on key-fetching via DNS
<hr>

Inside your GPG "options" file, find the "auto-key-locate" line, and add "cert" and/or "pka" to the options.

auto-key-locate cert pka (as well as other methods, like keyserver URLs)
Don't be surprised if a lot of people don't use this method.

Note that you can also turn on two options during signature verification. They are specified in a "verify-options" clause in your config file, or on the command line, and they are (right from the GPG manpage):

pka-lookups
    Enable PKA lookups to verify sender addresses. Note that
    PKA is based on DNS, and so enabling this option may dis-
    close  information  on when and what signatures are veri-
    fied or to whom data is encrypted. This is similar to the
    "web bug" described for the auto-key-retrieve feature.
And:

pka-trust-increase
    Raise  the  trust in a signature to full if the signature
    passes PKA validation. This option is only meaningful  if
    pka-lookups is set.
You can also use the same options on the command line (as you'll see in this document).

# Types of PGP Key Records
<hr>
## DNS PKA Records

Relevant RFCs: None that I can find.

Other Docs: The GPG source and mailing lists.

### Advantages

It's a TXT record. Easy to put in a zonefile with most management software.

No special tools required to generate, just three simple pieces of data.

Since it uses a special subzone, you can manage the _pka namespace in a separate zonefile.

GPG has an option, when verifying a signature, to look up these records (--verify-options pka-lookups), so it's doubly useful, both from a distribution and a verification point.

### Disadvantages

As with IPGP certs, you're at the mercy of the URL. This doesn't put your key in DNS, just the location of it, and the fingerprint. Some clients may not be able to support https or http 1.1.

Not RFC standard.

## Howto

Figure out which key you want to export:

```
$ gpg --list-keys randymcmillan
```
```
pub   rsa4096 2017-06-29 [SC] [expires: 2033-06-25]
      EE29B7B017F19DDA7C565A55AB2017A2CA7FD790
uid           [ultimate] randy mcmillan <randy.lee.mcmillan@gmail.com>
sub   rsa4096 2017-06-29 [E] [expires: 2033-06-25]

pub   rsa4096 2019-03-13 [SC]
      E616FA7221A1613E5B99206297966C06BB06757B
uid           [ultimate] Randy McMillan (github.com/randymcmillan) <randy.lee.mcmillan@gmail.com>
uid           [ultimate] admin@bitcoincore.dev <admin@bitcoincore.dev>
uid           [ultimate] randymcmillan@protonmail.com <randymcmillan@protonmail.com>
uid           [ultimate] randymcmillan@pm.me <randymcmillan@pm.me>
uid           [ultimate] [jpeg image of size 6741]
uid           [ultimate] admin@timechain.academy <admin@timechain.academy>
sub   rsa4096 2020-05-25 [E]
sub   elg4096 2022-01-12 [E]
sub   rsa4096 2022-01-12 [S]
sub   dsa3072 2022-01-12 [S]
sub   rsa4096 2022-01-21 [E]

pub   rsa4096 2022-03-21 [SC]
      D861F4DC1F4A61A2F694E9FAC1F7CD09865B2381
uid           [ultimate] Randy McMillan (0x20bf.org) <randy.lee.mcmillan@gmail.com>
uid           [ultimate] admin@0x20bf.org <admin@0x20bf.org>
uid           [ultimate] [jpeg image of size 6741]
sub   rsa4096 2022-03-21 [E]
sub   elg4096 2022-03-21 [E]
sub   rsa4096 2022-03-21 [S]
sub   dsa3072 2022-03-21 [S]
```

### Export the key to a file (I use BB06757B.pub.asc, but it can be anything)

```
gpg --export --armor BB06757B > BB06757B.pub.asc
```

### Get the fingerprint for your key:

```
gpg --list-keys --fingerprint BB06757B
```
```
pub   rsa4096 2019-03-13 [SC]
      E616 FA72 21A1 613E 5B99  2062 9796 6C06 BB06 757B
uid           [ultimate] Randy McMillan (github.com/randymcmillan) <randy.lee.mcmillan@gmail.com>
uid           [ultimate] admin@bitcoincore.dev <admin@bitcoincore.dev>
uid           [ultimate] randymcmillan@protonmail.com <randymcmillan@protonmail.com>
uid           [ultimate] randymcmillan@pm.me <randymcmillan@pm.me>
uid           [ultimate] [jpeg image of size 6741]
uid           [ultimate] admin@timechain.academy <admin@timechain.academy>
sub   rsa4096 2020-05-25 [E]
sub   elg4096 2022-01-12 [E]
sub   rsa4096 2022-01-12 [S]
sub   dsa3072 2022-01-12 [S]
sub   rsa4096 2022-01-21 [E]
```

Copy the file somewhere, like your webspace. It need not live on the same server. It needs to be accessable by the url you create in the next step.

```
cp BB06757B.pub.asc BB06757B.pub.asc.txt
```

Make up your text record. The format is:

DOMAIN: [https://pgp.guide](https://pgp.guide)  

```
@ 10800 IN OPENPGPKEY BB06757B
@ 10800 IN OPENPGPKEY E616FA7221A1613E5B99206297966C06BB06757B
@ 10800 IN TXT "E616FA7221A1613E5B99206297966C06BB06757B"
_pka 10800 IN TXT "v=pka1;fpr=BB06757B.pub.asc.txt\;uri=https://pgp.guide/BB06757B.pub.asc.txt"
admin._pka 10800 IN OPENPGPKEY E616FA7221A1613E5B99206297966C06 BB06757B
admin._pka 10800 IN TXT "v=pka1;fpr=BB06757B.pub.asc.txt\;uri=https://pgp.guide/BB06757B.pub.asc.txt"
```

We'll take this in several parts.

The record label is simply the email address with "._pka." replacing the "@".

```
admin@pgp.guide
```

###### becomes

```
admin._pka.pgp.guide
```

<center><span style="color:red">
*Don't forget the trailing dot, if you're using the fully qualified name.*
</br>
*I recommend sticking with fully-qualified, for simplicity.*
</span></center>


The body of the record is also simple. The v portion is just a version. There's only one version as far as I can tell, 'pka1'. The fpr is the fingerprint, with all whitespace stripped, and in uppercase. The uri is the location a key can be retrieved from. All the "names" are lowercase, separated by semicolons.

Publish the above record in your DNS. Bump your serial number and reload your nameserver. If you're using DNSSEC, re-sign your zone.

#### Testing

Most of the tests we're going to do for these are essentially the same activity. See if our DNS server is handing out an answer, and then see if GPG can retrieve it.

A simple dig:

```
dig +noall +answer +multiline pgp.guide any
dig +noall +answer +multiline  _pka.pgp.guide any 
dig +noall +answer +multiline  admin._pka.pgp.guide any
```
```
dig +short pgp.guide any
dig +short _pka.pgp.guide any
dig +short admin._pka.pgp.guide any


```

```
dig +short admin._pka.pgp.guide. TXT "v=pka1\;fpr=BB06757B\;uri=https://pgp.guide/BB06757B.pub.asc.txt"
```
(The backslashes before the semicolons are normal). Other than that, it seems to make sense and match what I put in.)

Test it with GPG. Rather than messing around with, and adding-from and deleting from live keyrings, you can do:

```
echo "bitkarrot" | gpg --no-default-keyring --keyring /tmp/gpg-$$ --encrypt --armor --auto-key-locate pka -r admin@pgp.guide
echo "bitkarrot" | gpg  --encrypt --armor --auto-key-locate pka -r admin@pgp.guide

echo "This is an encrypted message using the public key using the pka-lookups option in gnupg/openpgp..." | \
gpg  --encrypt --armor --auto-key-locate pka -r admin@pgp.guide > pka-lookups-message.gpg && \
open pka-lookups-message.gpg && cat pka-lookups-message

TIME=$(date +%s) && echo "This is an encrypted message using the public key using the pka-lookups option in gnupg/openpgp at Time:$TIME" | gpg  --encrypt --armor --auto-key-locate pka -r randy.lee.mcmillan@gmail.com > pka-lookups-message.$TIME.txt.gpg && gpg -d pka-lookups-message.$TIME.txt.gpg && echo

TIME=$(date +%s) && echo "This is an encrypted message using the public key using the pka-lookups option in gnupg/openpgp at Time:$TIME" | gpg  --encrypt --armor --auto-key-locate pka -r admin@pgp.guide > pka-lookups-message.$TIME.txt.gpg && gpg -d pka-lookups-message.$TIME.txt.gpg && echo


```
(where you@you.com is the address of your primary key.)

The /tmp/gpg-$$ creates a random file named after your PID. What you should see, and what I see, is something like this:

```
gpg: WARNING: using insecure memory!
gpg: please see http://www.gnupg.org/faq.html for more information
gpg: keyring `/tmp/gpg-39996' created
gpg: requesting key 624BB249 from http server prime.gushi.org
gpg: key 624BB249: public key "Daniel P. Mahoney <danm@prime.gushi.org>" imported
gpg: public key of ultimately trusted key CF45887D not found
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: Total number processed: 1
gpg:               imported: 1
gpg: automatically retrieved `danm@prime.gushi.org' via PKA
gpg: DE20C529: There is no assurance this key belongs to the named user

pub  2048g/DE20C529 2000-10-02 Daniel P. Mahoney <danm@prime.gushi.org>
Primary key fingerprint: C206 3054 5492 95F3 3490  37FF FBBE 5A30 624B B249
     Subkey fingerprint: CE40 B786 81E2 5CB9 F7D3  1318 9488 EB58 DE20 C529

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N) y
```

```
-----BEGIN PGP MESSAGE-----
Version: GnuPG v1.4.10 (FreeBSD)


hQIOA5SI61jeIMUpEAf/UotgWP8VQC9VTY36HaZeXO1CTFk90x0qlPrAhJk9YaoA
2eHNKZSoHKqaLjzTbaWnWHnNZu0IllIS+qrAwNeIAhswfzDoc8Q9+/4sGSR3LmxA
8SEwrJIvLmGVbqJEtnH8TTHIEao/lpL/d+ul4nLfbXRn0NW+MsaCAi8UsjbLlJeV
n4p0GQlpDoZCE55DTwMzfWMT84YVwuXTesuN+i7sSyJn2hT1rXuK1BCVcsgTcKdy
QhIo3EfKBlfFp74yiU7QCmlAujD6U6a93mmxezPIHVx/WGXgPExVRGgEzfT/tUcI
IQ2xMDUv4BF05hgm04GPGCbBY431j4UkdWWI6bvMLwgA2i01NmflH/6Z8+ss6J1M
e3RWnR7TPl5lDkXFBtLGAzO+HrsC5A32SbkTw+WsljCQLifJ2EalfoJ1QGY4Sp3v
H2YunwZLVPTc+D2JnrXfqNmi5zYZio8by3c8L0CgWdMwZ7PPxZpTOLN77/MIjBkJ
EBb8Z6SZCgzTIhN5z56ZgWFvmSKf1vKkeUcrgxMs+DnA+XqBMJ9w520JwoTLjJza
syrlYVhd+ktY21DYB9OJ5MZx2HMAtkUDRAzW1zoLcehk1kdZNzhpjU5hqSjT8/GN
trKFeqkmKemrq2GvMNyJyrEOB8e7KgbmXa95YKH0Wh2D4SWpXukegyCspmY4tDE+
uckaFSao+48g8D6vs1irGSxBRjyhD/jPDblrgpo=
=NbgW
-----END PGP MESSAGE-----
```


The "insecure memory" warning is a silly warning that the only way to turn off is to run GPG setuid root.

You can see in the output that the key comes from PKA.

The "it is NOT certain" warning has nothing to do with the fact that it came from DNS. You will get that warning every time you use that key (or any gpg key) until you have edited it and assigned ownertrust to it, or until the key is signed with a trusted signature, either from your personal web of trust, or from a signing service like the pgp.com directory.

Ask other people to run it for you and send you the resulting blob. You should be able to decrypt it with your private key.

### PGP CERT Records

Also known as: The "big" CERT record.
Relevant RFCs: RFC 2538, RFC 4398, specifically sections 2.1 and 3.3

#### Advantages

DNS is all you need. You don't have to host the key elsewhere. As a DNS nerd, this strikes me as very cool.

Suprisingly easy to verify with dig, if you have a base64 converter handy (openssl includes one)

#### Disadvantages

These records can get big. Really big. Especially if you have photo-ids on your keys. You can play with export-options to shrink it somewhat. Big dns packets may require EDNS, or dns-over-tcp, which not everyone supports, but support is becoming more widespread as a result of DNSSEC awareness.

Requires the make-dns-cert tool, which isn't built by default.

Requires you to have some control over your actual zonefile. Most control panels won't cut it.

Make-dns-cert currently generates a very ugly record for this.

#### How to

As before, the first step is to figure out which key we want.

```
gpg --list-keys danm@prime.gushi.org
Warning: using insecure memory!
pub   1024D/624BB249 2000-10-02  <-- I'm going to use this one.
uid                  Daniel P. Mahoney <danm@prime.gushi.org>
uid                  Daniel Mahoney (Secondary Email) <gushi@gushi.org>
sub   2048g/DE20C529 2000-10-02


pub   1024R/309C17C5 1997-05-08
uid                  Daniel P. Mahoney <danm@prime.gushi.org>
We export the key, but this time, it needs to be binary.
```
```
gpg --export 624BB249 > 624BB249.pub.bin
Warning: using insecure memory!
```
We run make-dns-cert on it. make-dns-cert comes with no manual or docs, but running with -h gives you all the clue you need.

```
make-dns-cert
    -f      fingerprint
    -u      URL
    -k      key file
    -n      DNS name
So then, make-dns-cert -n danm.prime.gushi.org. -k 624BB249.pub.bin
```

```
make-dns-cert -n danm.prime.gushi.org. -k 624BB249.pub.bin
```

```
danm.prime.gushi.org.   TYPE37  \# 1298 0003 0000 00 9901A20439D8DAF1110400F770EC6AA006076334BEC6DB6FBB237DC194BC0AB8
302C8953F04C28FC2085235D4F10EFA027234FBD63D142CCADD5213AD2B79A22C89ED9B4138370D8220D0F987F993A5364A4A7AC3D42F3765C384
71DDD0FF3372E4AE6F7BEE1E18EF464A0BEB5BBE860A08238891455EBE7CB53D567E981F78ADBD263206B0493ADCB74DD00A0FF0E9A1CD245415E
CEF59435162AFCE4CDD14BC70400EA38FF501256E773DEA299404854D99F4EDB2757AA911A9C77C68AB8D6622E517A556C43D21F0523C568F016C
D0DB89EF435F0D53B4E07434213F899E6578955DC2C147931E7B6901C9FD8A02705417D69A879B3CC196D2AC2EAEF311192EE89ABAF5A60942167
B4625735FCBDFB5DE0E3AC1236A53FA4D7CDD7D75F5DE85AF50400867D9546B28B79AF10541053CF4AB06A6171BFD21458BFD12AF1AE2B2401CAD
8851661F8AF6602F80EDAC99C79616BE1F910F4156242003779C68D7A079A8B18F89DD293E1B247E7420471300A4A0730AA61DE281CCC211FC405
A0A8A79877999FF9042AD892AB927DA371E8883BBB370AB7A97841408C3486BB18598CF2559BB42844616E69656C20502E204D61686F6E6579203
C64616E6D407072696D652E67757368692E6F72673E884E04101102000E050239D8DAF1040B030102021901000A0910FBBE5A30624BB249FA2E00
9B057503ED498695AE5ED73CA1B98EBAEE13F717E500A0921E0D92724459100266FEBBC29E911C8B0F530BB43244616E69656C204D61686F6E657
920285365636F6E6461727920456D61696C29203C67757368694067757368692E6F72673E8860041311020020050245D49FD7021B23060B090807
030204150208030416020301021E01021780000A0910FBBE5A30624BB249158400A082C8AF43DA8B85F740D6B1A6E9FF0B4490520B8C00A08F77D
21FBF86C842963E8090DC0646D1DD7F95C9B9020D0439D8DAF4100800F64257B7087F081772A2BAD6A942F305E8F95311394FB6F16EB94B3820DA
01A756A314E98F4055F3D007C6CB43A994ADF74C648649F80C83BD65E917D4A1D350F8F5595FDC76524F3D3D8DDBCE99E1579259CDFDB8AE744FC
5FC76BC83C5473061CE7CC966FF15F9BBFD915EC701AAD35B9E8DA0A5723AD41AF0BF4600582BE5F488FD584E49DBCD20B49DE49107366B336C38
0D451D0F7C88B31C7C5B2D8EF6F3C923C043F0A55B188D8EBB558CB85D38D334FD7C175743A31D186CDE33212CB52AFF3CE1B1294018118D7C84A
70A72D686C40319C807297ACA950CD9969FABD00A509B0246D3083D66A45D419F9C7CBD894B221926BAABA25EC355E9320B3B00020207FF5E1A3C
C5DA00E1E94EC8EF6C7FE9B49D944C71D8BBC817DD8E64A7344B9E48392E0B833B3B1DB7E6D5A38BE2826DEF0060F78C6417871EAF1CFBCBC47D2
7E93718D975E0A3A36D868C021D6B771740CE2918307D69D614BBF0632DC31932EA31397A7F3B04618C9A76C2F38265C7037E303EDD8AEF03D069
208E3FE9C4EA77D83E6311ED36C013D58C54E914B263A459E22D463A0288510C4752B99C163EEA0A55686979691AB0D9F9AA0C06C834446D7A723
EC534D819301382621ACF8930C74E9FD28C8797718AEC2C30CF601E24194B799234104A3D6239657B1D4AD545BDAA637F61541435CB51B4D138FB
F55E1A9FD2EED860E4459D6795B6FCCA23155A8846041811020006050239D8DAF4000A0910FBBE5A30624BB249415A009E37BCFDC64E76CBF6A86
82B85EA161BD1DFB793DF00A0C471BC7B9723535CD855D8FF1EB93F01E251B698
```

The program prints that all on one line.

Immediately, we notice a few things.

The record type isn't "CERT", it's "TYPE37". This confused me for a while until I discovered RFC3597 Basically, it's a way that a DNS server can handle a resource
record it doesn't know about, by giving it some special fields like the "#", as well as a length (which is the 1298 you see there).

The rest of the record is on one line. I wrapped it for the purposes of brevity. If I were using this in a zonefile, I would need to be careful that I wrapped it on a byte-boundary (every two characters is a byte). If I miss the boundary, named will refuse to load it, dnssec-signzone won't touch it, etc.

So the thing is ugly and you don't want to touch it. The easiest way to work with it is to drop all that into a file:

```
make-dns-cert -n danm.prime.gushi.org. -k 624BB249.pub.bin > 624BB249.big.cert
```

And then either read it into your editor, or tack it on like this:

```
cat 624BB249.big.cert >> your.zonefile
```

Be sure to make a backup first. Either way, you never have to copy/paste the raw hex and worry about newlines being inserted where you don't want them.

Before you reload your zone, you might want to use named-checkzone on it first:

prime# named-checkzone gushi.org gushi.org.hosts
zone gushi.org/IN: loaded serial 2009102909
OK
prime#
Voice of experience: You may want to dial the TTL (which controls how long servers will cache your data) way down on the record above. It's not hard, just put a number before the TYPE37, with a space, i.e:

danm.prime.gushi.org. 30 TYPE37
This way if it all goes terribly wrong, or you need to make changes, it won't be cached for very long.

If it looks okay, bump your serial number and reload.

#### Testing

As above, you can dig, but you won't be able to easily read the results:

```
dig +short danm.prime.gushi.org CERT
```

```
PGP 0 0 mQGiBDnY2vERBAD3cOxqoAYHYzS+xttvuyN9wZS8CrgwLIlT8Ewo/CCF I11PEO+gJyNPvWPRQsyt1SE60reaIsie2bQTg3DYIg0PmH+ZOlNkpKes
PULzdlw4Rx3dD/M3Lkrm977h4Y70ZKC+tbvoYKCCOIkUVevny1PVZ+mB 94rb0mMgawSTrct03QCg/w6aHNJFQV7O9ZQ1Fir85M3RS8cEAOo4/1AS
Vudz3qKZQEhU2Z9O2ydXqpEanHfGirjWYi5RelVsQ9IfBSPFaPAWzQ24 nvQ18NU7TgdDQhP4meZXiVXcLBR5Mee2kByf2KAnBUF9aah5s8wZbSrC
6u8xEZLuiauvWmCUIWe0Ylc1/L37XeDjrBI2pT+k183X119d6Fr1BACG fZVGsot5rxBUEFPPSrBqYXG/0hRYv9Eq8a4rJAHK2IUWYfivZgL4DtrJ
nHlha+H5EPQVYkIAN3nGjXoHmosY+J3Sk+GyR+dCBHEwCkoHMKph3igc zCEfxAWgqKeYd5mf+QQq2JKrkn2jceiIO7s3CrepeEFAjDSGuxhZjPJV
m7QoRGFuaWVsIFAuIE1haG9uZXkgPGRhbm1AcHJpbWUuZ3VzaGkub3Jn PohOBBARAgAOBQI52NrxBAsDAQICGQEACgkQ+75aMGJLskn6LgCbBXUD
7UmGla5e1zyhuY667hP3F+UAoJIeDZJyRFkQAmb+u8KekRyLD1MLtDJE YW5pZWwgTWFob25leSAoU2Vjb25kYXJ5IEVtYWlsKSA8Z3VzaGlAZ3Vz
aGkub3JnPohgBBMRAgAgBQJF1J/XAhsjBgsJCAcDAgQVAggDBBYCAwEC HgECF4AACgkQ+75aMGJLskkVhACggsivQ9qLhfdA1rGm6f8LRJBSC4wA
oI930h+/hshClj6AkNwGRtHdf5XJuQINBDnY2vQQCAD2Qle3CH8IF3Ki utapQvMF6PlTETlPtvFuuUs4INoBp1ajFOmPQFXz0AfGy0OplK33TGSG
SfgMg71l6RfUodNQ+PVZX9x2Uk89PY3bzpnhV5JZzf24rnRPxfx2vIPF RzBhznzJZv8V+bv9kV7HAarTW56NoKVyOtQa8L9GAFgr5fSI/VhOSdvN
ILSd5JEHNmszbDgNRR0PfIizHHxbLY7288kjwEPwpVsYjY67VYy4XTjT NP18F1dDox0YbN4zISy1Kv884bEpQBgRjXyEpwpy1obEAxnIByl6ypUM
2Zafq9AKUJsCRtMIPWakXUGfnHy9iUsiGSa6q6Jew1XpMgs7AAICB/9e GjzF2gDh6U7I72x/6bSdlExx2LvIF92OZKc0S55IOS4Lgzs7Hbfm1aOL
4oJt7wBg94xkF4cerxz7y8R9J+k3GNl14KOjbYaMAh1rdxdAzikYMH1p 1hS78GMtwxky6jE5en87BGGMmnbC84JlxwN+MD7diu8D0Gkgjj/pxOp3
2D5jEe02wBPVjFTpFLJjpFniLUY6AohRDEdSuZwWPuoKVWhpeWkasNn5 qgwGyDREbXpyPsU02BkwE4JiGs+JMMdOn9KMh5dxiuwsMM9gHiQZS3mS
NBBKPWI5ZXsdStVFvapjf2FUFDXLUbTROPv1Xhqf0u7YYORFnWeVtvzK IxVaiEYEGBECAAYFAjnY2vQACgkQ+75aMGJLsklBWgCeN7z9xk52y/ao
aCuF6hYb0d+3k98AoMRxvHuXI1Nc2FXY/x65PwHiUbaY
```

It's still ugly, but it's not AS ugly because it's base64, which includes spaces, at least, and is easier to search for a pattern. Base64 can also be easily wrapped on any boundary, which is nice.

You can run your existing exported key through a base64 converter, like the one built into the openssl binary, if you want to compare:

```
cat 624BB249.pub.bin | openssl enc -base64
```
```
mQGiBDnY2vERBAD3cOxqoAYHYzS+xttvuyN9wZS8CrgwLIlT8Ewo/CCFI11PEO+g
JyNPvWPRQsyt1SE60reaIsie2bQTg3DYIg0PmH+ZOlNkpKesPULzdlw4Rx3dD/M3
Lkrm977h4Y70ZKC+tbvoYKCCOIkUVevny1PVZ+mB94rb0mMgawSTrct03QCg/w6a
(...etc...)
OPv1Xhqf0u7YYORFnWeVtvzKIxVaiEYEGBECAAYFAjnY2vQACgkQ+75aMGJLsklB
WgCeN7z9xk52y/aoaCuF6hYb0d+3k98AoMRxvHuXI1Nc2FXY/x65PwHiUbaY
```

Now, while you could compare things byte-by-byte here, what I've done as a "casual check" is just pick random strings in the text and see if they match up. For example, you can see that "reaIsie2" is present in both. They both start with and end with similar strings on every line. The real test, of course, is to see if GPG recognizes it as a valid key.

By the way, since I use DNSSEC, dnssec-signzone rewrites this record into the proper "presentation format" for me, which is base64. If you want a similar function, you can use named-compilezone to get some of the same effects, or you can use the shell script I provide later in this document, with which you don't even need make-dns-cert.

Testing with gpg

As above, the command to test this is remarkably simple:

```
rm /tmp/gpg-*
```
```
echo "foo" | gpg --no-default-keyring --keyring /tmp/gpg-$$ --encrypt --armor --auto-key-locate cert -r danm@prime.gushi.org
```

```
gpg: keyring `/tmp/gpg-39996' created
gpg: key 624BB249: public key "Daniel P. Mahoney <danm@prime.gushi.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: automatically retrieved `danm@prime.gushi.org' via DNS CERT
gpg: DE20C529: There is no assurance this key belongs to the named user


pub  2048g/DE20C529 2000-10-02 Daniel P. Mahoney <danm@prime.gushi.org>
Primary key fingerprint: C206 3054 5492 95F3 3490  37FF FBBE 5A30 624B B249
     Subkey fingerprint: CE40 B786 81E2 5CB9 F7D3  1318 9488 EB58 DE20 C529
```

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.


Use this key anyway? (y/N) y

```
-----BEGIN PGP MESSAGE-----
Version: GnuPG v1.4.10 (FreeBSD)


hQIOA5SI61jeIMUpEAf/Sx7MKWm+e9EpUTSrDaBp4nJfDcBeqbYJulPRbDZz7eVW
2+ol6sG0jWjuirbG1YppZccEr9mgqaQujdSXb/bleD8POS0TEWuf3aPswFQvHf90
NLEzHt6BnfLoeobXXxyCflNaGX8zW+XgJtwZqAc2+jietuz8MOUhrf5m17CsW/wZ
IuEqwaek+K1irJp+w3rhaE08Jzb/S4CCifeW9J3mK57chQoPOu7Nz3rY666YKp/3
9T9StOgmFiNpvtFPNy4N7hHMHvbQwRsKlnkl+a7n0Aq2+OF4d1+/k2EE4uSGgcz0
oHvee8DnuOx3P92mO4Jz5/0O0lwBD7I51iOjzUurTAgAiIM5sHV8/QFCVzH9Ule+
gd8Wo5momcphkU/AXpce5Xgi/Vm4oGQ0x0queii8afUrzkpeN5SuwgQfAdOPiXW5
2bo527jBllxOxjeBasfky82XheTnLzbAQNvQNTEM9zE7zCl1LQJUZEJ1hVzcOevI
s+cm/AaGII9VkrAtSt3aLSRZuRJHFmhGvYd2Hz5WzcV1YFjXXP1eLwfetDBlaeB9
/K5v4hZBkIZPbHX0DcLVrP96mCIT4wCBYSJw+I6n0E6Fz3IfybQG2HMfqWp966/c
00ijx/aRDh42Dr/fTropuzzFzQr7weYDa1JnN3Zoftv6Zb/n+NcrmMiDCH8jJV6E
uMkaeeB5Mv7ssDQ9kPhO989CHFcznrE1lgOxjX8=
=NTLY
-----END PGP MESSAGE-----
```

Okay, as above, try to decrypt that with your private key.

### IPGP CERT Records

Also known as: The "little" or "short" CERT record. (These terms are purely my own).
Relevant RFCs: RFC 2538, RFC 4398, specifically sections 2.1 and 3.3

IPGP certs are interesting. It's basically the same pieces of infomation that are in the PKA record, as above, except that it's supported by an RFC. Despite the RFC compliance, I am not sure if any non-gpg client knows to look for them. However, because it's a DNS cert, make-dns-cert encodes the information in binary, and your DNS server will see it in base64. So verifying it visually is harder than verifying either of the above.

#### Advantages

Small, easy-to-transmit records.

Can use the same uri as the PKA record.

#### Disadvantages

Relies on the URI scheme. I haven't yet been able to get a definitive list of what uri schemes are supported, although I've seen http and finger. I've also seen reports that unless gpg is compiled against curl, http 1.1 is not supported (what this actually means is that any host that supports SSL will probably work, because of some of the nuances of SSL).

With PGP certs and IPGP certs, GPG will only parse the first key it gets, so if you publish both, and one doesn't work, there's no failover. I've argued that this should be fixed.

Requires make-dns-cert, which is not built in GPG by default. (But see "A Better Way" below)

Requires publication in your main DNS zone.

Despite being RFC compliant, GPG has additional trust vectors for PKA but not this, despite the fact that they share basically the same information.

Harder to verify with dig.

## How to

<H3>
<span style="color:red">
Note that some of these steps are redundant. If you're already doing a PKA key, skip to step 5.
</span>
</H3>



##### Dig:

```
gpg --list-keys danm@prime.gushi.org
Warning: using insecure memory!
pub   1024D/624BB249 2000-10-02  <-- I'm going to use this one.
uid                  Daniel P. Mahoney <danm@prime.gushi.org>
uid                  Daniel Mahoney (Secondary Email) <gushi@gushi.org>
sub   2048g/DE20C529 2000-10-02


pub   1024R/309C17C5 1997-05-08
uid                  Daniel P. Mahoney <danm@prime.gushi.org>
```

##### Export the key to a file (I use keyid.pub.asc, but it can be anything)

```
gpg --export --armor 624BB249 > 624BB249.pub.asc
Warning: using insecure memory!
```


##### Get the fingerprint for your key:

```
gpg --list-keys --fingerprint 624BB249

gpg: WARNING: using insecure memory!
gpg: please see http://www.gnupg.org/faq.html for more information
pub   1024D/624BB249 2000-10-02
      Key fingerprint = C206 3054 5492 95F3 3490  37FF FBBE 5A30 624B B249 <-- That bit is your fingerprint.
uid                  Daniel P. Mahoney <danm@prime.gushi.org>
uid                  Daniel Mahoney (Secondary Email) <gushi@gushi.org>
sub   2048g/DE20C529 2000-10-02
```

##### As above, run make-dns-cert. This time we use the -n, -f, and -u options:

```
make-dns-cert -n danm.prime.gushi.org. -f C2063054549295F3349037FFFBBE5A30624BB249 -u http://prime.gushi.org/danm.pubkey.txt
danm.prime.gushi.org.   TYPE37  \# 64 0006 0000 00 14 C2063054549295F3349037FFFBBE5A30624BB249
687474703A2F2F7072696D652E67757368692E6F72672F64616E6D2E7075626B65792E747874

```

##### Put the above in DNS. All on one line. Optionally add a TTL.


#### IMPORTANT: make sure you don't have any other CERT records with the same label (i.e. a "big" cert, as above). While it won't break things, you have no control over which (of multiple) people will get.



#### Reload your zone, and test. Testing will probably look VERY MUCH like the above, but here are the steps anyway:

### Testing


##### Dig:

```
dig +short danm.prime.gushi.org CERT
```
```
6 0 0 FMIGMFRUkpXzNJA3//u+WjBiS7JJaHR0cDovL3ByaW1lLmd1c2hpLm9y Zy9kYW5tLnB1YmtleS50eHQ=
Sadly, I haven't come across an easy way to decipher it yet, but there's always gpg.
```

##### GPG:

Since we're fetching the same kind of record, the command is exactly the same as before:

```
echo "foo" | gpg --no-default-keyring --keyring /tmp/gpg-$$ --encrypt --armor --auto-key-locate cert -r  danm@prime.gushi.org
gpg: WARNING: using insecure memory!
gpg: please see http://www.gnupg.org/faq.html for more information
gpg: keyring `/tmp/gpg-39996' created
gpg: requesting key 624BB249 from http server prime.gushi.org
gpg: key 624BB249: public key "Daniel P. Mahoney <danm@prime.gushi.org>" imported
gpg: public key of ultimately trusted key CF45887D not found
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: Total number processed: 1
gpg:               imported: 1
gpg: automatically retrieved `danm@prime.gushi.org' via DNS CERT
gpg: DE20C529: There is no assurance this key belongs to the named user


pub  2048g/DE20C529 2000-10-02 Daniel P. Mahoney <danm@prime.gushi.org>
Primary key fingerprint: C206 3054 5492 95F3 3490  37FF FBBE 5A30 624B B249
     Subkey fingerprint: CE40 B786 81E2 5CB9 F7D3  1318 9488 EB58 DE20 C529


It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.


Use this key anyway? (y/N) y
```
```
-----BEGIN PGP MESSAGE-----
Version: GnuPG v1.4.10 (FreeBSD)


hQIOA5SI61jeIMUpEAgApZurJi3hZmDaUFjB2j93eX/lTl96xq6T//sz6nT6jcTx
IPnq1RN8IrIQPjDBByHdqOZBT5hhblr9xi7NKIIv3W4q4L0z0fJx7NERPZNvn/H0
DkTwfDgAvCRxcKjenpLSwKZFwLjyfS7wjlDr3HFX7Tila0hbzplHslvgTE0QMcd7
7oNmEyOL3z+yZr/afQGp2wpzDv4YB9zOiNHcHcenqX0yrtiqKozZ9VAldi53rb/q
f38lwInbveyAcEQkE2iFwhRsbMR4VLcsBoxY6D9brsBprt23ey8Rnv+bQ9IAR0VN
/WYzU4zUUqb8HmpNFXQLEgH8A2BENw+bxkVYHjSfWQf/cBSGAzfBQQVJ7qp4tN0Z
FRVe51dokbU4NM9tGBdCzFHWARVkQX/Ulekd4F3sxBR/sum1UOT2xl2THVBz7/Pq
UCrTRPA0uH4dIbL5JpfGZhqsJ079+wmUWUtJIiO2wXi7ePEA/DrBC6p7jlmjyYN/
AeSKcPoTeLX+zryV5bECx4RO6S56EEcy0Ns0pASGMsgUnKL6Adrv3Y6ea3ZAOQMn
H9Uo28BKTKNUvUaBpN8cV8jIbKYPPW9i04kvEQRqs5rdamERCY1vVTqYTrcLsNqz
fF3KopX+V82X1oE2QuGdFfd8mK57ZXJL3VRUrfohQjhfYNKzougiP46rQQv79MYT
j8kazWyJUuufm6NVco1/35Zdp1UhHu8qTgXxrjo=
=zY9G
-----END PGP MESSAGE-----
```
Strangely, the output doesn't say what PKA does (a PKA retrieval has a line about fetching via HTTP), however, by checking my webserver logs, I can see it retrieved it from there:

```
tail -200 /usr/local/apache/logs/prime.gushi.org.log | grep pubkey | tail -1
```
```
prime.gushi.org 72.9.101.130 - - [28/Oct/2009:23:50:43 -0400] "GET /danm.pubkey.txt HTTP/1.1" 200 4337 "-" "-"
```
As usual, test decryption, etc. You're done.

## Further Steps

Figure out which of these are useful to you, and use them.

When someone asks for your public key, tell them to run the above command instead of mailing them your key or sending them a keyserver URL.

Consider using the pka-related verify-options.

Look into embracing DNSSEC. With a signed root, there's a good trust-path vector here. Who knows, maybe some day GPG will be dnssec-aware so it will give more credit to a secure DNS transaction. Without a signed root, there are still ways to have those who care about security use it, through services such as ISC's DLV registry.

On DNSSEC: At present, GPG cannot see the difference between an insecure response (one from an unsigned zone) and a correctly validated one from a signed zone. (In a signed zone, an unsigned or malformed will simply get a SERVFAIL dns response). Look into sponsoring development of GPG to make it as an application more aware of this.

### A better way to generate records

In reading over a lot of these commands, I've come across a few problems with the tools involved. They either require you to assemble large records by hand, or manipulate huge files.

DNS has also come a long way since these tools were written, and RFCs have solidified that have determined the "presentation format" (i.e. the "master file format") of what CERT records should look like.

On top of everything, the make-dns-cert tool is not built by default, and is not present in most binary distributions (RPM's, deb packages, FreeBSD's ports).

Thus, I took it upon myself to rewrite make-dns-cert as a shell script.

### Advantages

Extracts your key for you (takes a keyid as the argument).

Formats all three record types for you, you can pipe it right into your zone file.

Takes email address as an argument, generates record label.

No compiling needed.

Should work with most systems. Requires openssl and sed, a few other standard utilities.

Generates base64-ified CERT records, split into easy, manageable pieces.

Generates DNS-friendly comments, so repeating tasks are easy to reference.

(Eventually) available as a tarball, or as a paste-and-go script.

Arguments are in logical DNS record order emailaddress keyid [url].

Will generate an IPGP CERT record without a URI (this is legal per RFC4398).

You can see sample output here, and you can view the script itself here. Depending on your MIME settings, you can probably get a download link if you go here. If you see the script rather than getting a download prompt, you can just save-as.

README, Changelog, TODO coming soon.

### Other notes

I'm not 100 percent sure (mainly because I haven't tried), but with IPGP cert, and PKA, I believe I could in theory point at a keyserver directly, for example, specify a uri of http://pgp.mit.edu:11371/pks/lookup?op=get&search=0xB0307039309C17C5. I'm a bit dubious about the question marks and equals-signs, or if I might have to uri-encode things. It's something to be tried.

I'm trying to convince the GPG people that this would be much better adopted if the make-dns-cert tool was built/included by default, or if its function were included in gpg rather than a third-party tool. This is analagous as to how dnssec-keygen is used to generate SSHFP DNS records.

It doesn't do any actual cryptography, just some binary conversion, so in theory it could be rewritten in pure-perl, so there's nothing to compile.

I've made the argument to the GPG developers that if multiple CERT records are available, all should be tried if one fails. So far, if multiple exist, only the first received is parsed, and of course, DNS round-robins the answers by default.

It took me quite a lot of trial and error to realize that there's a difference between "modern" RSA keys, like this:
```
gpg --list-keys --fingerprint gushi@prime.gushi.org
pub   2048R/CF45887D 2009-10-29
      Key fingerprint = FCB0 485E 050D DDFA 83C6  76E3 E722 3C05 CF45 887D
uid                  Gushi Test <gushi@prime.gushi.org>
sub   2048R/C9761244 2009-10-29
and ancient RSA keys like this pgp2.6.2 monster:
```
```
gpg --list-keys --fingerprint danm@prime.gushi.org
pub   1024R/309C17C5 1997-05-08
       Key fingerprint = 04 4B 1A 2E C4 62 95 73  73 A4 EA D0 08 A4 45 76
uid                  Daniel P. Mahoney <danm@prime.gushi.org>
```

Note the lack of a subkey there. Note the weird fingerprint. I have not been able to get this key to properly export with gpg. If someone knows the Deep Magic, let me know.

### References

#### Blog posts and list threads

While researching this I came across little more than a few blog posts, and a few short discussions on the gpg-devel mailing list.

A blog entry that seems to have things mostly right.

GPG Mailing List Discussion which seems to date to when these features were first added.

My own thread on the gnupg-users mailing list that led up to this doc.

A slideshow of a talk given on PKA (really the only doc I could find with regard to PKA). Note that this is a postscript doc, for reasons I cannot fathom.

#### RFCs

RFC 3597 defines the odd format of the records that make-dns-cert generates, if it confuses you.

RFC 2538, which was superseded by RFC 4398, defines the format for a CERT record.

#### Todo

At least one GPG enthusiast has suggested to me that any tools I write to handle keys should simply be able to insert them using nsupdate. I don't disagree, but there's a complicated metric there as some of these require manipulation of a site's main zone, or at the very least, many subzones. In doing this I'd also like to find out a bit about how to do nsupdate with sig(0) and KEY records, which with the right policies would mean I could do this without touching named.conf. That may be the subject of a whole other howto.

(Done) I need to get the shell script cleaned up a bit more, and generate proper docs, and start tracking it with version control.

I should probably get the gumption up to formally license all this stuff. For right now, I declare it under the ISC License.

I'd like to track down the full list of supported URI types for PKA/IPGP CERT records. There doesn't seem to be a defined standard for it.

#### Epilogue

##### About the author

Dan Mahoney is a Systems Admin in the Bay Area, California. In his spare time he enjoys thinking for those brief fleeting moments what he would do if he had more free time. Keyid 624BB249, or email address danm@prime.gushi.org.

##### About this Document

This document was written in gnu nano, and HTML was generated using Markdown.

Markdown rocks.

Originally published on my livejournal at [link](http://gushi.livejournal.com/524199.html), its main home is at [link](http://www.gushi.org/make-dns-cert/HOWTO.html), which is where later versions will be published.

Free to use, comments to the above email address are welcome.
