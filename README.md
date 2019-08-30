# sourcejedi.google-chrome #

Install the Google Chrome web browser.


## Status

This role works fine on my systems (Fedora and Debian).  I expect it to break at some point.  See Package Signing Keys below.


## Requirements

Your operating system must be based on "Debian" (`apt` package manager) or "RedHat" (`dnf` or possibly `yum` package manager).

Your CPU architecture must be supported by Google Chrome.  64-bit x86 is supported.  32-bit x86 is no longer supported.

Currently, this role assumes the `gpg` command can be obtained by installing the `gnupg` package.  It's OK if this is still the traditional GnuPG v1, like in Fedora.  It would only break if they removed the `gnupg` package, and expected us to install `gnupg2`.


## Warning for Fedora Linux

Fedora users are reminded not to remove `fedora-workstation-repositories`.  Removing that package can cause security vulnerabilities.  You do not have to install that package if you do not want to.  The problem is if you remove `fedora-workstation-repositories` after installing Chrome, it will remove `google-chrome.repo`.  This will cut off your security updates, leaving you vulnerable to attack.  This applies even if you originally installed Chrome by downloading it from the website.  The same problem would apply to all repositories which `fedora-workstation-repositories` installs.  New repositories may be added to this package at any time.  I am not aware of any script to handle the problem.  (For technical completeness: this Ansible role appears to avoid the problem.  This is not deliberate.  Do not rely on it).


## Role Variables

For yum or dnf, you can download updates from a shared cache by setting `google_chrome__yum_baseurl`.  To set this value, take the baseurl for the repository, and remove the architecture at the end (`/x86-64` / `/$basearch`).  I use this with apt-cacher-ng.

We consider `proxy=` in dnf.conf or yum.conf to be obsolete.  PackageKit deliberately ignores it, and has no direct equivalent.

There is no proxy variable for systems using apt.  You can instead set `Acquire::http::Proxy` in apt.conf.  The setting will be respected both by apt and PackageKit.

There is an older variable `google_chrome__yum_proxy`, to set a HTTP proxy on the repository.  The proxy MUST support HTTPS through the HTTP CONNECT method.  apt-cacher-ng MUST NOT be used.  The yum/dnf repo configuration fetches the signing key over HTTPS.  apt-cacher-ng does not support HTTPS.  I have seen PackageKit fail to fetch the signing key, and then ignore Chrome security updates.  This affects the GNOME Software updater, for example.


## Package Signing Keys

This role verifies the known fingerprints of Google package signing keys, inspired by Dockerfile best practices.  If you like to cross-check the current fingerprints, simply visit the [Google Linux Software Repositories](https://www.google.com/linuxrepositories/) page.

Once Google Chrome has been verified and installed, it should take care of any subsequent key updates as normal.  They will be downloaded over HTTPS.  I believe key pinning is *not* implemented.

However the current version of this role will stop working when the Packages Signing Authority key changes.  Note the "expires" fields below.

```
$ gpg --with-fingerprint files/linux_signing_key.pub
pub  1024D/7FAC5991 2007-03-08 Google, Inc. Linux Package Signing Key <linux-packages-keymaster@google.com>
      Key fingerprint = 4CCA 1EAF 950C EE4A B839  76DC A040 830F 7FAC 5991
sub  2048g/C07CB649 2007-03-08
pub  4096R/D38B4796 2016-04-12 Google Inc. (Linux Packages Signing Authority) <linux-packages-keymaster@google.com>
      Key fingerprint = EB4C 1BFD 4F04 2F6D DDCC  EC91 7721 F63B D38B 4796
sub  4096R/640DB551 2016-04-12 [expires: 2019-04-12]
sub  4096R/997C215E 2017-01-24 [expires: 2020-01-24]
```

(Possible improvement: provide an option which refetches `linux_signing_keys.pub`, for use by developers.  This option will fail *during* a key transition, instead of *after*.  It can use `get_url force=yes` - it's a small file - so I can pick up the transition the next time I run the role.  The option should work in --check mode, if at all possible).


## License

The role itself is licensed GPLv3, please open an issue if this creates any problem.

Terms of Service apply for Google Chrome:

* [Google Chrome site](https://www.google.com/chrome/).  The site will show the terms of service before letting you download Chrome.
* [Google Chrome Terms of Service](https://www.google.com/intl/en/chrome/browser/privacy/eula_text.html]), "Printer-friendly version".

Google Chrome may include executable code from multiple parties, with specific sections in the Terms of Service.  Chrome may also use Google online services.  The terms do not require Google to notify you of any changes.

For open source code, without third-party components, look for the [Chromium Browser](https://www.chromium.org/).
