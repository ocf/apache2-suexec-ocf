# How to build this package

This package likely won't need any updates except if new Debian releases come
around. When that happens however, it's a bit involved to build.

First, check out the apache version for the release (for example,
https://packages.debian.org/buster/apache2 would lead to `2.4.38-3+deb10u4` at
the time of writing).

Clone this repo, add the Debian apache2 package repo as a remote, and create a
new branch for the appropriate tag from that repo:
```bash
$ git clone git@github.com:ocf/apache2-suexec-ocf.git
$ cd apache2-suexec-ocf
$ git remote add debian https://salsa.debian.org/apache-team/apache2.git
$ git fetch debian
$ git checkout -b ocf/2.4.38-3+deb10u4 debian/2.4.38-3+deb10u4
```

Next, cherry pick on top any OCF commits from the previous version's branch
(https://github.com/ocf/apache2-suexec-ocf/commits/ocf/2.4.25-3 in this case),
and resolve any conflicts that come up.

Then mount it all into a docker container for the correct distribution you're
building for:
```bash
$ docker run -ti -v $PWD:/mnt:rw docker.ocf.berkeley.edu/theocf/debian:buster /bin/bash
```

Generally follow the instructions for building a package as in
https://www.ocf.berkeley.edu/docs/staff/procedures/backporting-packages/ to
install dependencies. Installing `bison` and `flex` are good to do as they
don't seem to be in build dependencies but will still error during the build if
they aren't found.

Download the apache package from the `.dsc` file on
https://packages.debian.org/buster/apache2 (`dget -x
http://deb.debian.org/debian/pool/main/a/apache2/apache2_2.4.38-3+deb10u4.dsc`
in `/` of the container). Once `dpkg-checkbuilddeps` shows no output and
`dquilt push -a` applies all the patches successfully on a repo copy with no
local modifications, then you should be good to build the package with
`dpkg-buildpackage -us -uc -sa`.
