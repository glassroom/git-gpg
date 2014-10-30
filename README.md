# git-gpg

git-gpg allows you to store private files in a GPG-encrypted remote repository.

How it works:

When you pull changes, git-gpg...

1. Runs rsync to download encrypted .git database files from a remote server to a local directory.
2. Decrypts the .git database files to form a "staging" repository.
3. Pulls from the staging repository into your local repo.

When you push changes, git-gpg...

1. Pushes your local changes into the staging repository.
2. Encrypts the .git database files in the staging repository.
3. Runs rsync to upload the encrypted .git database files to the remote server.

Simple.

## Requirements

To try git-gpg you will need:

+ A recent version of git. Tested with 1.9.1.
+ A recent version of gpg. Tested with 1.4.16.
+ A recent version of rsync. Tested with 2.6.9.
+ A gpg private key. For help, see the [gpg cheat sheet](http://irtfweb.ifa.hawaii.edu/~lockhart/gpg/gpg-cs.html).
+ A git repository.

## Installation

Copy the `git-gpg` script to a directory in your path.

## Usage

### Setup

Add a GPG-encrypted remote repository:

    git gpg add myremote user@server:~/files/project.gpg

Add one or more GPG recipients:

    git gpg add-recipient myremote you@host1
    git gpg add-recipient myremote friend@host2
    ...

Show information about a GPG-encrypted repository, including recipients and branches:

    git gpg show myremote

### Push

Push to the GPG-encrypted repository:

    git gpg push myremote master

You can use the standard git command line options when pushing.

    git gpg push --verbose myremote master

For example, specify the local branch and the remote branch.

    git gpg push myremote local-branch:remote-branch

Delete a branch.

    git gpg push myremote :branch-to-delete

### Pull

Pull from the GPG-encrypted repository:

    git gpg pull myremote

You can use the standard git command line options when pulling.

    git gpg pull --rebase myremote

For example, you can specify the remote branch and local branch.

    git gpg pull myremote remote-branch:local-branch


## Other Approaches / Prior Art

I experimented with a number of other solutions before arriving at this
version of git-gpg, including the following:

#### Git Clean / Smudge Filters

Git provides a "clean filter" that will run right before a file is committed,
and a "smudge filter" that will run right after a file is checked out. One of
the intended use cases is to transform files from Unix line endings to and
Windows line endings and vice versa.

Some clever folks have hacked this so that the client filter decrypts a file,
and the smudge filter encrypts the file. While this is clever, it is not
recommended:

> If your "clean" encrypts and "smudge" decrypts, it means you are refusing
> all the benifit git offers.  You are making a pair of similar "smudged"
> contents totally dissimilar in their "clean" counterparts.  That is simply
> backwards.
>
> - Junio C Hamano - Maintainer of Git since 2005.
> - <http://thread.gmane.org/gmane.comp.version-control.git/113124/focus=113221>

#### The git-remote-gcrypt Project

Another approach involves a [remote
helper](https://www.kernel.org/pub/software/scm/git/docs/git-remote-
helpers.html). The code is difficult to follow, but as far as I can tell, it
creates a git repository inside of another git repository. The inner git
repository stores encrypted [packfiles](http://git-scm.com/book/en/Git-
Internals-Packfiles). The outer repository is used simply for a transport
mechanism. That may be totally wrong, as the code is quite complex.

> There are two ways of constructing a software design: One way is to make it so
> simple that there are obviously no deficiencies, and the other way is to make
> it so complicated that there are no obvious deficiencies.
>
> Tony Hoare - winner of the 1980 Turing Award.
> <http://en.wikiquote.org/wiki/C._A._R._Hoare>

I don't trust software that is too complex. The lead developer seems to share
some of this sentiment:

> I hope it interests someone, and maybe it even reaches the target of
> being both usable and secure. It also helps me if you point out
> if/how/why it is broken(!).
>
> Ulrik Sverdrup - Author of git-remote-gcrypt
> https://groups.google.com/forum/#!topic/git-users/0nm3lP122K0

#### An Encrypted Remote FileSystem

Another approach is to store the remote git repo on an encrypted filesystem,
and then mount it when pushing or pulling code. An intruder must time their
attack to happen when the code is decrypted in memory. This raises the
difficulty level, but still leaves a significant weak surface. If you are
*truly* paranoid about your code, then it's dangerous to *ever* decrypt it
anywhere except for on your own computer.

#### Encrypted Patches

A previous version of this utility attempted transmit and store encrypted
patches rather treating the entire repository as a single blob. The was more
bandwidth efficient than the current version, but it had two main disadvantages:

+ Patches don't preserve the commit hash, so different consumers of the repo saw different hashes.
+ It is possible to have a valid git repository that cannot be re-created by exporting and re-applying all of its constituent patches in order.

The second disadvantage was obviously a show-stopper.

# Licence

git-gpg is released under the MIT License.
