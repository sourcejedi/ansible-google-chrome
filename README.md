# sourcejedi.google_chrome #

Install the Google Chrome web browser.


## Status

This role works fine on my systems (Fedora and Debian).  I expect it to break at some point.  See Package Signing Keys below.

Ansible `--check` mode is supported.  This checks that each task has been applied, and has not been reversed, without changing the managed system(s).  If tasks need to be (re)applied, you are likely to see failures following the "changed" tasks.  This is because some tasks depend on earlier tasks having been applied.


## Requirements

Your operating system must be based on "Debian" (`apt` package manager) or "RedHat" (`dnf` or possibly `yum` package manager).

Your CPU architecture must be supported by Google Chrome.  64-bit x86 is supported.  32-bit x86 is no longer supported.

This role now assumes the `gpg` command can be obtained by installing the `gnupg2` package.


## Warning for Fedora Linux

Fedora users are reminded not to remove `fedora-workstation-repositories`.  Removing that package can cause security vulnerabilities.  You do not have to install that package if you do not want to.  The problem is if you remove `fedora-workstation-repositories` after installing Chrome, it will remove `google-chrome.repo`.  This will cut off your security updates, leaving you vulnerable to attack.  This applies even if you originally installed Chrome by downloading it from the website.  The same problem would apply to all repositories which `fedora-workstation-repositories` installs.  New repositories may be added to this package at any time.  I am not aware of any script to handle the problem.  (For technical completeness: this Ansible role appears to avoid the problem.  This is not deliberate.  Do not rely on it).


## Role Variables

For yum or dnf, you can download updates from a shared cache by setting `google_chrome__yum_baseurl`.  To set this value, take the baseurl for the repository, and remove the architecture at the end (`/x86-64` / `/$basearch`).  I use this with apt-cacher-ng.

We consider `proxy=` in dnf.conf or yum.conf to be obsolete.  PackageKit deliberately ignores it, and has no direct equivalent.

There is no proxy variable for systems using apt.  You can instead set `Acquire::http::Proxy` in apt.conf.  The setting will be respected both by apt and PackageKit.

There is an older variable `google_chrome__yum_proxy`, to set a HTTP proxy on the repository.  The proxy MUST support HTTPS through the HTTP CONNECT method.  apt-cacher-ng MUST NOT be used.  The yum/dnf repo configuration fetches the signing key over HTTPS.  apt-cacher-ng does not support HTTPS.  I have seen PackageKit fail to fetch the signing key, and then ignore Chrome security updates.  This affects the GNOME Software updater, for example.


## Package Signing Keys

This role verifies the known fingerprints of Google package signing keys, inspired by Dockerfile best practices.  If you like to cross-check the current fingerprints, simply visit the [Google Linux Software Repositories](https://www.google.com/linuxrepositories/) page.

Once Google Chrome has been verified and installed, it should take care of any subsequent key updates.  They will be downloaded over HTTPS.  I believe key pinning is *not* implemented.

This role could stop working if the necessary keys are changed.  I do not know how Google schedule the changes to their keys.  I also have not worked out which key or subkey is needed to verify packages.  At least on Fedora, the role appears to have kept working when "expires" dates shown below have been passed (in all the "sub"-keys of the Signing Authority).

```
$ gpg2 --with-fingerprint --import-options import-show --dry-run --import < files/linux_signing_key.pub
pub   dsa1024 2007-03-08 [SC]
      4CCA 1EAF 950C EE4A B839  76DC A040 830F 7FAC 5991
uid                      Google, Inc. Linux Package Signing Key <linux-packages-keymaster@google.com>
sub   elg2048 2007-03-08 [E]

gpg: key 7721F63BD38B4796: 2 signatures not checked due to missing keys
pub   rsa4096 2016-04-12 [SC]
      EB4C 1BFD 4F04 2F6D DDCC  EC91 7721 F63B D38B 4796
uid                      Google Inc. (Linux Packages Signing Authority) <linux-packages-keymaster@google.com>
sub   rsa4096 2019-07-22 [S] [expires: 2022-07-21]

gpg: Total number processed: 2
```

(Possible improvement: provide an option which refetches `linux_signing_keys.pub`, for use by developers.  This option will fail *during* a key transition, instead of *after*.  It can use `get_url force=yes` - it's a small file - so I can pick up the transition the next time I run the role.  The option should work in --check mode, if at all possible).


## License

The role itself is licensed GPLv3, please open an issue if this creates any problem.

Terms of Service apply for Google Chrome:

* [Google Chrome site](https://www.google.com/chrome/).  The site will show the terms of service before letting you download Chrome.
* [Google Chrome Terms of Service](https://www.google.com/intl/en/chrome/browser/privacy/eula_text.html]), "Printer-friendly version".

Google Chrome may include executable code from multiple parties, with specific sections in the Terms of Service.  Chrome may also use Google online services.  The terms do not require Google to notify you of any changes.

For open source code, without third-party components, look for the [Chromium Browser](https://www.chromium.org/).
