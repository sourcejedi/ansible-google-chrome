# sourcejedi.google_chrome #

Install the Google Chrome web browser.


## Status

This role works fine on my systems.

If you use `apt-cacher-ng`, please read the corresponding section below.

After installing Chrome on a Debian-based system, if you leave it too long without updating, Chrome will fail to update.  This is because your system has missed the transition between package signing keys.  To fix your system security, run this role to refresh the keys.  You should then install all the missed updates before using Chrome (or any other web browser :-).

Ansible `--check` mode is supported.  This checks that each task has been applied, and has not been reversed, without changing the managed system(s).  If tasks need to be (re)applied, you may see failures following the "changed" tasks.  This is because some tasks depend on earlier tasks having been applied.


## Requirements

Your operating system must be based on "Debian" (`apt` package manager) or "RedHat" (`dnf` or possibly `yum` package manager).

Your CPU must be supported by Google Chrome.  Chrome supports 64-bit x86.  Chrome does not support 32-bit x86.

This role now assumes the `gpg` command can be obtained by installing the `gnupg2` package.


## Warning for Fedora Linux

Fedora users are reminded not to remove `fedora-workstation-repositories`.  Removing that package can cause security vulnerabilities.  You do not have to install that package if you do not want to.  The problem is if you remove `fedora-workstation-repositories` after installing Chrome, it will remove `google-chrome.repo`.  This will cut off your security updates, leaving you vulnerable to attack.  This applies even if you originally installed Chrome by downloading it from the website.  The same problem would apply to all repositories which `fedora-workstation-repositories` installs.  New repositories may be added to this package at any time.  I am not aware of any script to handle the problem.  (For technical completeness: this Ansible role appears to avoid the problem.  This is not deliberate.  Do not rely on it).


## apt-cacher-ng

Google packages are downloaded over HTTPS.  To use a shared cache like apt-cacher-ng, the specific HTTPS repo must be configured in both DNF or APT, and on the proxy server.

I have not implemented this for APT.  If you just set a proxy server in APT, then Google packages will not be cached.

This role will show an error if you use the suggested "Basic Configuration" of apt-cacher-ng.  See: [Not getting Google Chrome updates, after setting up apt-cacher-ng](https://unix.stackexchange.com/questions/746532/not-getting-google-chrome-updates-after-setting-up-apt-cacher-ng).


## Role variables

For yum or dnf, you may download updates from a shared cache by setting `google_chrome__yum_baseurl`.  To set this value, take the baseurl for the repository, and remove the architecture at the end (`/x86-64` / `/$basearch`).  I used this with apt-cacher-ng.

We consider `proxy=` in dnf.conf or yum.conf to be obsolete.  PackageKit deliberately ignores it, and does not provide a direct equivalent.

There is an older variable `google_chrome__yum_proxy`, to set a HTTP proxy on the repository.  This proxy MUST support HTTPS through the HTTP CONNECT method, as warned in earlier versions.  It is also not able to cache updates :-).


## Package Signing Keys

This role verifies the known fingerprints of Google package signing keys, inspired by Dockerfile best practices.  If you like to cross-check the current fingerprints, simply visit the [Google Linux Software Repositories](https://www.google.com/linuxrepositories/) page.

Once the Google Chrome package has been verified and installed, it takes responsibility for any subsequent key updates.  On Fedora, I believe updated keys are downloaded over HTTPS.


## License

The role itself is licensed GPLv3, please open an issue if this creates any problem.

Terms of Service apply for Google Chrome:

 * [Google Chrome website](https://www.google.com/chrome/).  This website will show the terms of service if you try to download Chrome.
 * [Google Terms of Service](https://policies.google.com/terms)
 * [Google Chrome and ChromeOS Additional Terms of Service](https://www.google.com/intl/en/chrome/terms/).

Google Chrome may include executable code from multiple parties, with specific sections in the Additional Terms of Service.  Chrome may also use Google online services.  The terms do not require Google to notify you of any changes.

For open source code, without third-party components, look for the [Chromium Browser](https://www.chromium.org/).
