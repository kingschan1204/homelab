操作系统比较：`alpine-virt 3.21.2`,`roky linux 9.4`,`debian 12`,`ubuntu 22`
```
root@pve:~# pvesh get /cluster/resources --type vm
┌──────────┬──────┬─────────────┬─────────┬───────┬────────┬────────────┬───────────┬─────────┬───────┬──────┬────────┬───────────┬──────────┬────────────┬─────────────┬────────────┬───────────┬───
│ id       │ type │ cgroup-mode │ content │   cpu │   disk │   diskread │ diskwrite │ hastate │ level │ lock │ maxcpu │   maxdisk │   maxmem │        mem │ name        │      netin │    netout │ no
╞══════════╪══════╪═════════════╪═════════╪═══════╪════════╪════════════╪═══════════╪═════════╪═══════╪══════╪════════╪═══════════╪══════════╪════════════╪═════════════╪════════════╪═══════════╪═══
│ qemu/100 │ qemu │             │         │ 0.64% │ 0.00 B │  50.33 MiB │    0.00 B │         │       │      │      4 │ 10.00 GiB │ 2.00 GiB │ 121.95 MiB │ alpine-virt │    60.00 B │    0.00 B │ pv
├──────────┼──────┼─────────────┼─────────┼───────┼────────┼────────────┼───────────┼─────────┼───────┼──────┼────────┼───────────┼──────────┼────────────┼─────────────┼────────────┼───────────┼───
│ qemu/101 │ qemu │             │         │ 0.12% │ 0.00 B │ 237.44 MiB │ 11.95 MiB │         │       │      │      4 │ 32.00 GiB │ 2.00 GiB │ 349.04 MiB │ rokylinux   │ 760.95 KiB │ 36.62 KiB │ pv
├──────────┼──────┼─────────────┼─────────┼───────┼────────┼────────────┼───────────┼─────────┼───────┼──────┼────────┼───────────┼──────────┼────────────┼─────────────┼────────────┼───────────┼───
│ qemu/103 │ qemu │             │         │ 0.07% │ 0.00 B │ 125.37 MiB │  4.68 MiB │         │       │      │      4 │  2.00 GiB │ 2.00 GiB │ 210.29 MiB │ debian      │ 103.96 KiB │  8.29 KiB │ pv
├──────────┼──────┼─────────────┼─────────┼───────┼────────┼────────────┼───────────┼─────────┼───────┼──────┼────────┼───────────┼──────────┼────────────┼─────────────┼────────────┼───────────┼───
│ qemu/104 │ qemu │             │         │ 0.07% │ 0.00 B │ 229.45 MiB │ 44.00 MiB │         │       │      │      4 │  2.20 GiB │ 2.00 GiB │ 418.72 MiB │ ubuntu      │    60.00 B │    0.00 B │ pv
└──────────┴──────┴─────────────┴─────────┴───────┴────────┴────────────┴───────────┴─────────┴───────┴──────┴────────┴───────────┴──────────┴────────────┴─────────────┴────────────┴───────────┴───
root@pve:~# 

```
资源占用比较（整理一下）
```
┬─────────────┬────────────┬───────┬────────┬──────────┬────────────┬───────────┬───────────┬────────────┬───────────┬
│ name        │        mem │   cpu │ maxcpu │   maxmem │   diskread │ diskwrite │   maxdisk │      netin │    netout │
╪═════════════╪════════════╪═══════╪════════╪══════════╪════════════╪═══════════╪═══════════╪════════════╪═══════════╪
│ alpine-virt │ 121.95 MiB │ 0.64% │      4 │ 2.00 GiB │  50.33 MiB │    0.00 B │ 10.00 GiB │    60.00 B │    0.00 B │
┼─────────────┼────────────┼───────┼────────┼──────────┼────────────┼───────────┼───────────┼────────────┼───────────┼
│ rokylinux   │ 349.04 MiB │ 0.12% │      4 │ 2.00 GiB │ 237.44 MiB │ 11.95 MiB │ 32.00 GiB │ 760.95 KiB │ 36.62 KiB │
┼─────────────┼────────────┼───────┼────────┼──────────┼────────────┼───────────┼───────────┼────────────┼───────────┼
│ debian      │ 210.29 MiB │ 0.07% │      4 │ 2.00 GiB │ 125.37 MiB │  4.68 MiB │  2.00 GiB │ 103.96 KiB │  8.29 KiB │
┼─────────────┼────────────┼───────┼────────┼──────────┼────────────┼───────────┼───────────┼────────────┼───────────┼
│ ubuntu      │ 418.72 MiB │ 0.07% │      4 │ 2.00 GiB │ 229.45 MiB │ 44.00 MiB │  2.20 GiB │    60.00 B │    0.00 B │
┴─────────────┴────────────┴───────┴────────┴──────────┴────────────┴───────────┴───────────┴────────────┴───────────┴
```
