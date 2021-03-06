# URL Interop

(or RFC 3986 vs WHATWG URL Specification vs the real world)

This document is an attempt to describe where and how [RFC
3986](https://tools.ietf.org/html/rfc3986) (86), [RFC
3987](https://tools.ietf.org/html/rfc3987) (87) and [the WHATWG URL
Specification](https://url.spec.whatwg.org/) (TWUS) differ. This might be
useful input when trying to interop with URLs on the modern Internet.

This document focuses on network-using URL schemes (http, https, ftp, etc) as
well as 'file'.

This is a best-effort and describes the situation as we understand it at the
time of writing. Since the TWUS document is "living" and software keep getting
updated, chances are conditions have changed since we last updated this
document!

If you find mistakes or have additional interop issues to suggest, please file
[an issue](https://github.com/bagder/docs/issues) or [a pull
request](https://github.com/bagder/docs/pulls)!

## URL components

Put simply, a URL may consist of the following components - many of them are
optional:

    [scheme][divider][userinfo][hostname][port number][path][fragment]

Each component is separated from the following component with a divider
character.

Which in an example could look like

    http://user:password@www.example.com:80/index.hmtl#top

| Component  | Value           | Known interop issues exist |
|------------|-----------------|----------------------------|
| scheme     | http            | no                         |
| divider    | ://             | YES                        |
| userinfo   | user:password   | YES                        |
| hostname   | www.example.com | YES                        |
| port number| 80              | YES                        |
| path       | index.html      | YES                        |
| fragment   | top             | no                         |

## Scheme

There are no known interop issues.

## Divider

86: specifies this to be [exactly
"://"](https://tools.ietf.org/html/rfc3986#section-3) for all network-using
(hierarchical) schemes.

TWUS: says a parser must accept one to an infinite amount of slashes but a
producer should use two.

Real world: one and three slash URLs occur, possibly a few using even
more. 'file' URLs are notoriously often malformed.

## Userinfo

The userinfo field can be used to set user name and password to pass on to the
server. The use of this field is discouraged since it often means passing
around the password in plain text and is thus a security risk.

86: specifies that ['@' is the
separator](https://tools.ietf.org/html/rfc3986#section-3.2) between the
userinfo field and the host name.

TWUS: accepts '@' as part of the user name if it shows up before a colon

**This is an interop collision**

## Hostname

### Numerical IP addresses

86: mentions how [IPv4 addresses with a
dot-notation](https://tools.ietf.org/html/rfc3986#section-3.2.2) are valid

TWUS: specifies that both 32bit numbers ("12345677") as well as partial
dot-addresses ("127.0") are valid.

Real world: 32 bit numbers occur, and are automagically supported if typical
OS level name resolver funcitons are used since they often support this out of
the box.

### IDNA

Hostnames were traditionally ascii based. When introducing IDN hostnames, it
has caused problems to the specifications and they are lacking.

86: Is written to work without IDN (ascii characters), so basically it works
with already punycoded domain names.

87: Specifies [IDNA 2003 to be used](https://tools.ietf.org/html/rfc3987#section-3.1)

TWUS: Implies that IDNA 2003 should be used

Real world: A total mess. Some TLD domains (`.de` for example) require IDNA
2008, which makes user-agents treating the host name according to IDNA 2003
and IDNA 2008 transitional to fail. Some user-agents use IDNA 2003, some do
IDNA 2008 transitional and some do IDNA 2008 non-transitional.

**This is an interop collision**

## Port number

The port number is a TCP port number between 0 and 65535.

[86](https://tools.ietf.org/html/rfc3986#section-3.2.3) and TWUS agree that
this is a base-10 number that is virtually unbounded in
length. 00000000000000000000000000000000000080 means 80 in both specs.

Real world: at least curl and wget2 ignore "rubbish" entered after the number
all the way to the next component divider (a slash, a pound sign, or a
question mark). That seems to be a bug according go both 86 and TWUS.

## Path

### 8bit

86: says that a URL path is specified as ASCII characters or need to be URL
encoded. That makes 8-bit characters illegal.

TWUS:

Real world: 8bit characters are occasionally seen in URLs in the wild, and
when used in redirects, browsers are known to URL-encode them in the next
outgoing request.

### U+0020, space

86: does not allow spaces (U+0020) to be part of the path. A space instead
ends the URL.

TWUS: allows spaces in URLs and will instead URL-encode it to %20 when sent in
a request. A TWUS URL thus needs other magic to know where a URL ends.

Real world: Spaces are occasionally seen in URLs in the wild, and when used in
redirects, browsers are known to URL-encode them in the next outgoing request.

## Fragment

There are no known interop problems with fragments. The fragment part is not
included in protocol data that goes over the network.
