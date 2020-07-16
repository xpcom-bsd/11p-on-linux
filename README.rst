.. image:: https://rail2x.berlin/img/logo.svg
  :width: 100em
  :align: right
  :target: https://rail2x.berlin/

==============
|11p| on Linux
==============

This repository contains documentation to make Linux talk
`IEEE 802.11p <https://standards.ieee.org/findstds/standard/802.11p-2010.html>`__,
an amendment to the popular *IEEE 802.11* standard (aka WiFi) aiming at
*wireless access in vehicular environments* (WAVE).

The project originated in the context of the
`Rail2X project <https://rail2x.berlin/>`__
at the `Operating Systems and Middleware Group <https://osm.hpi.de>`__
of the `Hasso Plattner Institute Potsdam <https://hpi.de/>`__.

A
`paper <https://scholarspace.manoa.hawaii.edu/bitstream/10125/60190/1/0750.pdf>`__
has been published in the Proceedings of the 52nd
`Hawaii International Conference on System Sciences <https://hicss.hawaii.edu/>`__
2019.

PRs welcome.

Contents
========

.. contents:: \

hardware support
================

Hardware officially claiming to support |11p| is rare, but apparently
some wireless cards that support the 5 GHz band also support
|11p|'s 5.9 GHz band [2]_.

A list of wireless cards for |11p| can be found `here <https://github.com/jfpastrana/802.11p/blob/master/Documentation/Wireless_cards.pdf>`__.

We have made good experiences with the Atheros ``AR9462`` chipset, but
– according to the link above – ``AR9280``, ``AR9382``, and probably
``AR9380-AL1A`` should work as well.

Additionally, we note the following

.. list-table::
  :header-rows: 1

  * - card
    - chip
    - TX power
    - driver
    - works
    - comment
  * - ?
    - AR9462
    - 18 dBm
    - ath9k
    - ✓
    - no official |11p| support, but operation in 5.9 GHz band works
  * - `MikroTik R11e-5HnD <https://mikrotik.com/product/R11e-5HnD>`__
    - AR9580
    - 27 dBm
    - ath9k
    - ✓
    - according to ``iw``, TX power can't be set to > 26 dBm
  * - `MikroTik R11e-5HacT <https://mikrotik.com/product/R11e-5HacT>`__
    - QCA9880
    - 28 dBm
    - ath10k
    - work (sometimes) in progress
    - see `branch ath10k
      <https://gitlab.com/hpi-potsdam/osm/g5-on-linux/11p-on-linux/-/tree/ath10k>`__
      (`compare <https://gitlab.com/hpi-potsdam/osm/g5-on-linux/11p-on-linux/-/compare/master...ath10k>`__),
      status: device accepts OCB mode but firmware crashes when switching to
      10 MHz channel width
  * - ?
    - `QCA6564AU <https://www.qualcomm.com/products/qca6564au>`__
    - ?
    - ?
    - ?
    - for "automotive connectivity", probably work in 5.9 GHz band since
  * - ?
    - `QCA6574AU <https://www.qualcomm.com/products/qca6574au>`__
    - ?
    - ?
    - ?
    - for "automotive connectivity", probably work in 5.9 GHz band since
      the chip also supports the a/g/n standard
  * - ?
    - `88w8987xa family <https://www.marvell.com/wireless/88w8987xa/>`__
    - ?
    - ?
    - ?
    - would be interesting, since official support for |11p|
  * - ?
    - `UBX-P3 <https://www.u-blox.com/en/product/ubx-p3-series>`__,
      `VERA-P1 <https://www.u-blox.com/en/product/vera-p1-series>`__
    - ?
    - ?
    - ?
    - would be interesting, since official support for |11p|
  * - ?
    - `SMBV-K54 <https://www.rohde-schwarz.com/de/produkt/smbvk54-produkt-startseite_63493-10244.html>`__
    - ?
    - ?
    - ?
    - would be interesting, since official support for |11p|

setup
=====

Most required changes for different components of the IEEE 802.11 stack
are already in Linux mainline [1]_ and the upstream versions of the
user space tools.
However, we still need do some more changes to get ready for |11p|.

The `automatic setup`_ and the `manual setup`_ are developed for and
tested on Debian Buster.
If you run another OS, you should be able to adapt the commands.
Also, feel free to prepare a pull request for your OS.

automatic setup
---------------

This is the preferred way of setting up hosts for |11p|.

How to get things going:

#. clone this repository, go there

   * ``git clone git@gitlab.com:hpi-potsdam/osm/g5-on-linux/11p-on-linux.git``
   * ``cd 11p-on-linux``

#. install Ansible from the source of your preference

   * e.g. from package repositories: ``apt/dfn/pip/… install ansible``

#. for every host you want to configure (e.g., ``myhost``)

   #. make sure you have SSH, sudo and Python working on the remote host

      ``$ ssh -t myhost sudo python -V``

      You should see the Python version information.

   #. add the host to your Ansible inventory

      ``$ echo myhost >> ansible/inventory``

#. test your Ansible setup (and ask for ``sudo`` password)

   ``$ (cd ansible && ansible all --become --ask-become-pass -m ping)``

   You should see a "pong" per host.

#. finally, configure all the hosts listed in the inventory
   file

   ``$ (cd ansible && ansible-playbook --ask-become-pass example-playbook.yml)``

After that, the wireless interfaces of your hosts are set up for
|11p| and applications can use them.

manual setup
------------

Please re-consider using the `automatic setup`_ which is maintained
better.

If something goes wrong, you can try to read the Ansible files and see
if you can get some inspiration on how things should work.

kernel
......

(The corresponding Ansible tasks can be found in
`ansible/roles/patch-driver-ath/tasks/main.yml
<ansible/roles/patch-driver-ath/tasks/main.yml>`__)

The mentioned changes in the Linux mainline kernel are implemented from
version 4.10 upwards (maybe 4.4. works as well, but that is history
anyway).

To check the kernel version available, run:

.. code-block:: shell

  apt search linux-image

Install/upgrade to the latest version:

.. code-block:: shell

  sudo apt install linux-image-{ arch }

Reboot if required.

Please keep in mind that your running kernel version (``uname -r``) and
your driver source code version need to be in line to build and work
correctly. Also, installing a matching kernel version might be easier
than adapting the patches.

patch (``ath9k``) driver
........................

(The corresponding Ansible tasks can be found in
`ansible/roles/patch-driver-ath/tasks/main.yml
<ansible/roles/patch-driver-ath/tasks/main.yml>`__)

When using a wireless card without official support for |11p| (i.e.,
support for 5.9 GHz band and OCB mode), we need to patch its drivers,
compile them as loadable kernel modules and load them.

For cards requiring the ``ath9k`` driver, it works like this:

Get a copy of the Linux source code and other necessary packages for
compilation:

.. code-block:: shell

  sudo apt install linux-headers-{ arch } linux-source build-essential \
    kernel-package libssl-dev

After installation the Linux sources can be found in
``/usr/src/linux-source-{ version }.tar.xz``.
If there is no file matching your Kernel version (check ``uname -r``),
maybe because you installed your kernel manually, get the source
directly `from the kernel repository <https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git/>`__
e.g.:

.. code-block:: shell

  wget https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git/snapshot/linux-stable-{ version }.tar.gz

In any case, change to a working directory of your choice and extract
the kernel sources:

.. code-block:: shell

  mkdir ~/my-workdir
  cd ~/my-workdir
  tar -xvjf /path/to/linux-source-{ version }.tar.bz2 -C .

Check out this repository:

.. code-block:: shell

  cd ~/my-workdir
  git clone git@gitlab.com:hpi-potsdam/osm/g5-on-linux/11p-on-linux.git

Now apply the patches to Linux provided by this repository to the
``ath9k`` driver:

.. code-block:: shell

  cd linux-source-{ version }
  patch -up0 < ~/my-workdir/11p-on-linux/patches/linux.patch

And build:

.. code-block:: shell

  # still in directory linux-source-{ version }/drivers/net/wireless/ath
  make clean all

This leaves us with several ``.ko`` files, with which we want to replace
respective modules that are currently loaded.

First, unload currently loaded ``ath`` modules.

.. code-block:: shell

  sudo rmmod -f ath9k
  sudo rmmod -f ath9k_htc
  sudo rmmod -f ath9k_common
  sudo rmmod -f ath9k_hw
  sudo rmmod -f ath

Then load what we just build:

.. code-block:: shell

  # still in linux-source-{ version }/drivers/net/wireless/ath
  sudo modprobe ath.ko
  sudo modprobe ath9k/ath9k_hw.ko
  sudo modprobe ath9k/ath9k_common.ko
  sudo modprobe ath9k/ath9k_htc.ko
  sudo modprobe ath9k/ath9k.ko

To check whether the drivers were patched and loaded correctly, check if
the output of ``iw phy`` is listing frequencies > 5825 MHz.
If so, our physical layer setup succeeded.

updating iw
...........

(The corresponding Ansible tasks can be found in
`ansible/roles/install-user-space-tools/tasks/iw.yml
<ansible/roles/install-user-space-tools/tasks/iw.yml>`__)

|11p| support is available in ``iw`` 4.0 and later [3]_ [4]_.
If your system has an older version, you need to update.
Check your version with

.. code-block:: shell

  iw --version

In case of an older version proceed as follows:
Install pkg-config and libnl development files

.. code-block:: shell

  sudo apt install pkg-config libnl-genl-3-dev

Clone the ``iw`` `official repository <http://git.kernel.org/cgit/linux/kernel/git/jberg/iw.git>`__.

.. code-block:: shell

  cd ~/my-workdir
  git clone git://git.kernel.org/pub/scm/linux/kernel/git/jberg/iw.git
  cd iw

Build and install:

.. code-block:: shell

    make
    sudo make PREFIX=/ install

Verify ``iw`` is aware of the OCB mode:

.. code-block:: shell

  iw | grep -i ocb
  dev <devname> ocb leave
  dev <devname> ocb join <freq in MHz> <5MHZ|10MHZ> [fixed-freq]

wireless-regdb, CRDA
....................

(The corresponding Ansible tasks can be found in
`ansible/roles/install-user-space-tools/tasks/wireless-regdb.yml
<ansible/roles/install-user-space-tools/tasks/wireless-regdb.yml>`__
and
`ansible/roles/install-user-space-tools/tasks/crda.yml
<ansible/roles/install-user-space-tools/tasks/crda.yml>`__)

In order to insert regulatory information about |11p|'s 5.9 GHz band,
we need to update Linux' ``wireless-regdb``.

Install the required dependencies:

.. code-block:: shell

  sudo apt install python-m2crypto

Clone, patch, build install:

.. code-block:: shell

  cd ~/my-workdir
  git clone https://git.kernel.org/pub/scm/linux/kernel/git/sforshee/wireless-regdb.git
  cd wireless-regdb
  patch -up0 < ~/my-workdir/11p-on-linux/patches/wireless-regdb.patch
  sudo make
  sudo make PREFIX=/ install

The last step is to sign that new regulatory data with the Central
Regulatory Domain Agent (CRDA). Install the required dependencies:

.. code-block:: shell

  sudo apt install python-m2crypto libgcrypt11-dev

Again, clone, patch, build, install:

.. code-block:: shell

  cd ~/my-workdir
  git clone https://git.kernel.org/pub/scm/linux/kernel/git/mcgrof/crda.git
  cd crda
  patch -up0 < ~/my-workdir/11p-on-linux/patches/crda.patch
  make REG_BIN=/lib/crda/regulatory.bin
  sudo make install PREFIX=/ REG_BIN=/lib/crda/regulatory.bin

Copy your public key (installed by wireless-regdb, see above) to CRDA
sources [3]_.

.. code-block:: shell

  cp /lib/crda/pubkeys/$USER.key.pub.pem pubkeys/

Test CRDA and the generated regulatory.bin [3]_:

.. code-block:: shell

  sudo /sbin/regdbdump /lib/crda/regulatory.bin | grep -i ocb
  …
  (5865 - 5875 @ 10), (23), NO-CCK, OCB-ONLY
  (5875 - 5885 @ 10), (33), NO-CCK, OCB-ONLY
  (5885 - 5895 @ 10), (23), NO-CCK, OCB-ONLY
  (5895 - 5905 @ 10), (33), NO-CCK, OCB-ONLY
  …

troubleshooting
...............

Is the patched driver loaded correctly?
  Check ``iw phy`` and look for the frequencies of *Band 2 > 5825 MHz*
  If not unload ``ath`` modules and reload the modules you compiled
  (see above).

Is ``iw`` up to date and knows about OCB?
  Check ``iw | grep -i ocb``
  If not, check ``iw`` version and install a more recent version
  (see above).

Is the CRDA configured correctly?
  Check ``sudo /sbin/regdbdump /lib/crda/regulatory.bin | grep -i ocb``
  If the output is empty: see above how to patch the regulatory
  database.

When ``iw`` says: "command failed: Operation not supported (-95)"
  Disconnect from any WiFi network (however your connected, possibly
  via a GUI).

  Stop any network manager to avoid further troubles, e.g.:

  .. code-block:: shell

    sudo systemctl stop NetworkManager.service

"out of range" or "device not ready"
  A network manager might be messing with the device.
  Stop any network managers, e.g.:

  .. code-block:: shell

    sudo systemctl stop NetworkManager.service

put interface in |11p| band and OCB mode
----------------------------------------

(The corresponding Ansible tasks can be found in
`ansible/roles/configure-interface-for-ocb/tasks/main.yml
<ansible/roles/configure-interface-for-ocb/tasks/main.yml>`__)

Now, with a working setup we are now able to send packets over a
5.9 GHz band of our choice (examples with interface ``wlan0`` – modify
commands to match your device name, of course).

.. code-block:: shell

  sudo ip link set wlan0 down
  sudo iw dev wlan0 set type ocb
  sudo ip link set wlan0 up
  sudo iw dev wlan0 ocb join 5880 10MHZ

Assign an IP address per device:

(The corresponding Ansible tasks can be found in
`ansible/roles/configure-interface-ip/tasks/main.yml
<ansible/roles/configure-interface-ip/tasks/main.yml>`__)

.. code-block:: shell

  ip address add 10.1.1.X/24 brd + dev wlan0

Q&A
===

Why don't you patch the ``OCB-ONLY`` flags in the regulatory database?
  tl;dr from `a discussion on the Linux kernel mailing list
  <https://lore.kernel.org/patchwork/patch/469021/#674106>`__:
  the standard requires only OCB to be used on the frequencies, but it
  is not a regulatory restriction.

  Apart from the fact, the we do research-only modifications, there is
  no need to add the additional (and somewhat wrong) regulatory
  restriction ``OCB-ONLY``.

contributors
============

In alphabetic order (last name):

* [contributors from references]
* Jossekin Beilharz
* Marcus Ding
* Lukas Pirl
* Christian Werling

.. [1] `|11p| Linux Kernel. Implementation. R. Lisový, M. Sojka, Z. Hanzálek. Czech Technical University in Prague. <https://rtime.felk.cvut.cz/publications/public/ieee80211p_linux_2014_final_report.pdf>`__

.. [2] `802.11p standard and V2X applications on commercial Wi-Fi cards. Javier Fernández Pastrana. Universidad de Valladolid. <https://uvadoc.uva.es/bitstream/10324/23064/1/TFM-G%20668.pdf>`__

.. [3] `Linux IEEE 802.11p – How to <https://ctu-iig.github.io/802.11p-linux/>`__

.. [4] https://git.kernel.org/cgit/linux/kernel/git/jberg/iw.git/commit/?id=3955e5247806b94261ed2fc6d34c54e6cdee6676

.. |11p| replace:: IEEE 802.11p
