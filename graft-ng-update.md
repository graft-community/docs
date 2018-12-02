# How to update to a newer version of graft-ng

Note: this assumes that you already have a working graft-ng and want to update to a new release.

It also assumes that your `graft-ng` is checked out in `~/graft-ng` and that your build directory is `~/supernode`.

## Updating git (staying on the official dev branch)

The following will fetch new commits from github:

    cd ~/graft-ng
    git fetch --all
    git checkout --recurse-submodules alpha3

In some cases, you may need to update the GraftNetwork code embedded inside graft-ng separately.  To do that you would continue
from the above using:

    cd ~/graft-ng/modules/cryptonode
    git fetch --all
    git checkout rta-alpha3

Now rebuild graft-ng from scratch by moving aside the existing build directory and creating a fresh one:

    cd ~
    mv supernode supernode.old
    mkdir supernode
    cd supernode
    cmake -DENABLE_SYSLOG=ON ~/graft-ng
    make -j2    # Adjust as needed: -j1 for a low-powered machine, -j8 for an 8-core monster with 16GB of ram

That's it; this will give you a new, up-to-date build.


# Trying out the community branch

## Updating

If you already have a community branch checkout, you simply need to update it using:

    cd ~/graft-ng
    git pull --recurse-submodules

and then rebuild using:

    cd ~/supernode
    make -j2    # Adjust as needed: -j1 for a low-powered machine, -j8 for an 8-core monster with 16GB of ram

## Switching from the dev branch

The community development branch follows the main alpha development branch, but also includes various community-developed
fixes and/or features.  If you want to try it out, the following recipe should get you going.

Assuming you already have a `graft-ng` repository cloned from the official graft-project branch, you
can add the community branch as a second `remote` using:

    cd ~/graft-ng
    git remote add g.c https://github.com/graft-community/graft-ng.git
    git fetch g.c
    git checkout gc-alpha

If you don't have `graft-ng` at all, you can obtain the community version directly using:

    cd ~
    git clone --recurse-submodules -b gc-alpha https://github.com/graft-community/graft-ng.git

Using either of these approaches, you can then build the supernode using:

    cd ~
    mv supernode supernode.old   # (only if you already have a supernode build directory)
    mkdir supernode
    cd supernode
    cmake -DENABLE_SYSLOG=ON ~/graft-ng
    make -j2    # Adjust as needed: -j1 for a low-powered machine, -j for an 8-core monster with 16GB of ram

Note that the community version builds slightly differently: the `graftnoded` and `graft-wallet-cli` files will end up
in the build directory (`~/supernode`) rather than in a subdirectory (`~/supernode/BUILD/bin`).


# I'm lazy and hate compiling, help!

Yes, you can!  The graft-community version is also available in a .deb, built by Jason (@jagerman).  You can get it by running:

    curl -s https://deb.graft.community/public.gpg | sudo apt-key add -
    echo "deb https://deb.graft.community bionic main" | sudo tee /etc/apt/sources.list.d/graft.community.list
    apt update
    apt install graftnoded-alpha graft-supernode
    cp /usr/share/doc/graft-supernode/config.ini ~/supernode.ini

Then you can launch graftnoded by just running (see the other guides for details on running this in
screen or as a service):

    graftnoded

or can launch the supernode by running:

    # Only need this the first time:
    cp /usr/share/doc/graft-supernode/config.ini ~/supernode.ini

    graft_server --config-file ~/supernode.ini

In the future, when you want to update, `apt update && apt upgrade` will check for and install an update, if one is available.
