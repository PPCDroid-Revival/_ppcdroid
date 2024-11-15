# Git and repo setup

Before you download the source, you will need [http://git-scm.com/ git] and [http://source.android.com/source/git-repo.html repo] tools.

On an Ubuntu LTS (12.04) x86 host (the recommended one), you can do the following:

Getting `git`:
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

Getting `repo`:
You'll need an old version of repo for this to work, the latest version is too new for such an old copy of Ubuntu.  Here's how you do that (to get the latest 1.x version):
```
curl -L 'https://gerrit.googlesource.com/git-repo/+/refs/tags/v1.13.11/repo?format=TEXT'  | base64 -d > repo
chmod +x repo
```

You'll need to add this to your path.

## PPCDroid source code download
Use the following commands to get source code:

```
$ mkdir ppcdroid
$ cd ppcdroid
$ repo init -u https://github.com/PPCDroid-Revival/manifest -m <manifest_name>
$ repo sync -j10
```

`<manifest_name>` can be the following:
   * ppcdroid-donut.xml - for Android 1.6 (Donut)
   * ppcdroid-gingerbread.xml - for Android 2.3 (Gingerbread)
