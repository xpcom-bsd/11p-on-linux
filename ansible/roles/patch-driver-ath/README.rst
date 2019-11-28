When using a wireless card without official support for 802.11p
(e.g., operation in 5.9 GHz and OCB mode), we need to patch the
drivers and compile them as loadable kernel modules.

This Ansible role prepares the kernel to make it aware of IEEE 802.11p.
Specifically, the wireless device driver ``ath9k`` is patched, compiled
and loaded.

This needs to be re-run after reboots.
