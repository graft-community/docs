# How to update to a newer version of graft-ng

Note: this assumes that you already have a working graft-ng and want to update to a new release.

It also assumes that your `graft-ng` is checked out in `~/graft-ng` and that your build directory is `~/supernode`.

## Updating git

The following will fetch new commits from github:

    cd ~/graft-ng
    git checkout alpha3   # Or whatever the current branch is called.
    git pull --recurse-submodules

In some cases, you may need to update the GraftNetwork code embedded inside graft-ng separately.  To do that you would use:

    cd ~/graft-ng/modules/cryptonode
    git checkout rta-alpha3    # change, if needed, to whichever branch you want to use
    git pull

Now rebuild graft-ng from scratch by moving aside the existing build directory and creating a fresh one:

    cd ~
    mv supernode supernode.old
    mkdir supernode
    cd supernode
    cmake -DENABLE_SYSLOG=ON ~/graft-ng
    make -j2    # Adjust as needed: -j1 for a low-powered machine, -j8 for an 8-core monster with at least 16GB ram

That's it; this will give you a new, up-to-date build.


# Trying out the community branch

The community development branch follows the main alpha development branch, but also includes various community-developed
fixes and/or features.  If you want to try it out, the following recipe should get you going:

    cd ~/graft-ng
    git remote add g.c https://github.com/graft-community/graft-ng.git
    git fetch g.c
    git checkout gc-alpha
    git pull --recurse-submodules
    cd ~
    mv supernode supernode.old
    cd supernode
    cmake -DENABLE_SYSLOG=ON ~/graft-ng
    make -j2    # Adjust as needed: -j1 for a low-powered machine, higher for a machine with more cores/ram

Note that the community version builds slightly differently: the `graftnoded` and `graft-wallet-cli` files will end up
in the build directory (`~/supernode`) rather than in a subdirectory (`~/supernode/BUILD/bin`).


# I'm lazy and hate compiling, help!

Yes, you can!  The graft-community version is also available in a .deb, built by Jason (@jagerman).  You can get it by running:

    curl -s https://deb.graft.community/public.gpg | sudo apt-key add -
    echo "deb https://deb.graft.community bionic main" | sudo tee /etc/apt/sources.list.d/graft.community.list
    apt update
    apt install graftnoded-alpha graft-supernode
    cp /usr/share/doc/graft-supernode/config.ini ~/supernode.ini

Then you can launch graftnoded by just running

    graftnoded

and can launch the supernode by running

    graft_server --config-file ~/supernode.ini

In the future, when you want to update, `apt update && apt upgrade` will check for and install an update, if one is available.
