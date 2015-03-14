# Git and repo setup #

Before you download the source, you will need [git](http://git-scm.com/) and [repo](http://source.android.com/source/git-repo.html) tools.

On an Ubuntu LTS (10.04) host, you can do the following:

Getting _git_:
```
$ sudo apt-get install git-core
```

Getting other tools:
```
$ sudo apt-get install flex bison gperf build-essential zip curl python zlib1g-dev libncurses5-dev
```

Additionally on 64-bit hosts you should run:
```
$ sudo apt-get install lib32ncurses5-dev ia32-libs lib32readline5-dev lib32z-dev
```

Getting _repo_:
```
$ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo >~/bin/repo
$ chmod a+x ~/bin/repo
```

You can find detailed repo installing instructions [here](http://source.android.com/source/git-repo.html).

# PPCDroid source code download #
Use the following commands to get source code:

```
$ mkdir ppcdroid
$ cd ppcdroid
$ repo init -u git://gitorious.org/ppcdroid/manifest.git -m manifest_name
$ repo sync -j10
```

_manifest\_name_ can be the following:
  * ppcdroid-donut.xml - for Android 1.6 (Donut)
  * ppcdroid-gingerbread.xml - for Android 2.3 (Gingerbread)