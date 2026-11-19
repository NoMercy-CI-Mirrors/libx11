# Reporting Security Issues

Please notify us of any security issues by sending mail to
<xorg-security@lists.x.org>.

See https://www.x.org/wiki/Development/Security/Organization/
for more information about the X.Org security team.

# Learning about Security Fixes

X.Org announces security bugs and bug fix releases on the xorg-announce
mailing list. See the archives at https://lists.x.org/archives/xorg-announce/
and see https://lists.x.org/mailman/listinfo/xorg-announce to subscribe.

Security advisories are also listed on our wiki at
https://www.x.org/wiki/Development/Security/ and mailed to the
https://oss-security.openwall.org/wiki/mailing-lists/oss-security mailing list.

# Covered library components

This policy covers libX11 and the following associated X client libraries:

- [libXcomposite](https://gitlab.freedesktop.org/xorg/lib/libXcomposite/)
- [libXdamage](https://gitlab.freedesktop.org/xorg/lib/libXdamage/)
- [libXext](https://gitlab.freedesktop.org/xorg/lib/libXext/)
- [libXfixes](https://gitlab.freedesktop.org/xorg/lib/libXfixes/)
- [libXft](https://gitlab.freedesktop.org/xorg/lib/libXft/)
- [libXi](https://gitlab.freedesktop.org/xorg/lib/libXi/)
- [libXinerama](https://gitlab.freedesktop.org/xorg/lib/libXinerama/)
- [libICE](https://gitlab.freedesktop.org/xorg/lib/libICE/)
- [libXmu](https://gitlab.freedesktop.org/xorg/lib/libXmu/)
- [libXp](https://gitlab.freedesktop.org/xorg/lib/libXp/) / [libXprintUtil](https://gitlab.freedesktop.org/xorg/lib/libXprintUtil/)
- [libXrandr](https://gitlab.freedesktop.org/xorg/lib/libXrandr/)
- [libXrender](https://gitlab.freedesktop.org/xorg/lib/libXrender/)
- [libXres](https://gitlab.freedesktop.org/xorg/lib/libXres/)
- [libXScrnSaver](https://gitlab.freedesktop.org/xorg/lib/libXScrnSaver/)
- [libSM](https://gitlab.freedesktop.org/xorg/lib/libSM/)
- [libXt](https://gitlab.freedesktop.org/xorg/lib/libXt/)
- [libXtst](https://gitlab.freedesktop.org/xorg/lib/libXtst/)
- [libXv](https://gitlab.freedesktop.org/xorg/lib/libXv/) / [libXvMC](https://gitlab.freedesktop.org/xorg/lib/libXvMC/)
- [libXxf86dga](https://gitlab.freedesktop.org/xorg/lib/libXxf86dga/)
- [libXxf86misc](https://gitlab.freedesktop.org/xorg/lib/libXxf86misc/)
- [libXxf86vm](https://gitlab.freedesktop.org/xorg/lib/libXxf86vm/)

Affiliated libraries that are not covered by this policy are:

- [libXfont](https://gitlab.freedesktop.org/xorg/lib/libXfont/): runs server-side and has a different trust model
- [libfontenc](https://gitlab.freedesktop.org/xorg/lib/libfontenc/): used by libXfont and shares that library's trust model
- [libxtrans](https://gitlab.freedesktop.org/xorg/lib/libxtrans/): runs both client and server-side, pre-authentication

In the above cases, it's the **server-side** usage of the library that puts it
outside of scope for this policy.

# Security model and trust boundaries

The X server is responsible for all user input (keyboard, mouse, ...) and
the display of any image to the user. It *has* to be trusted by the user,
because there is no use-case where interaction with an assumed hostile X server
makes sense. From the perspective of the client-side X libraries, the X server
can thus be assumed to be trusted.

A hostile X server that attempts to exploit a client-side vulnerability can
only do so if the client is running with privileges higher (or other) than the
X server itself. It's been a long-standing policy of the X.Org project
that clients utilizing libX11 and associated libraries must not run
with elevated privileges when they connect to the X server.

Because of the above, we consider the X server and a local XIM server
running at the same privilege level as the client to be inside the
client's trust boundary:

- A malicious or buggy X server that crashes a client is **not** considered
  a security vulnerability (though any crash is still a bug in the X library
  and we welcome regular issue reports for it).
- The same applies to a local XIM server running at the same privilege level
  as the client. The use-case of a remote XIM server using TCP transport
  has been deprecated for decades and has not been used in real-world
  deployments for almost as long.

libX11 also processes data from other local sources, such as locale and
compose files (XLOCALEDIR, XLOCALELIBDIR, XCOMPOSEFILE, XCOMPOSECACHE,
system locale data), the XCMS color-name database (XCMSDB), the keysym
database (XKEYSYMDB), and X resource database files (XENVIRONMENT,
~/.Xdefaults). Bugs in parsing these are evaluated on a case-by-case
basis, with the primary consideration being whether the vulnerable code
path is reachable by an attacker who controls these environment variables
but not the process's effective UID.
