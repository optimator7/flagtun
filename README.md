# flagtun

A fast ICMP tunnel for Linux — carries your traffic inside ping packets between
two servers.

## Install

Run as root on each server:

```sh
bash <(wget -qO- https://github.com/optimator7/flagtun/releases/latest/download/flagtun.sh)
```

## Config file reference

Each tunnel is a file at `/etc/flagtun/<name>.toml`. The menu writes it for you;
you can edit it and run `systemctl restart flagtun@<name>` to apply. Every value:

```toml
tunnel_name = "kharej-1"          # label shown in logs (default: the file name)
mode        = "server"            # "server" (iran) or "client" (kharej)   [required]
remote_ip   = "203.0.113.10"      # the peer's public IP                    [required]
local_ip    = "198.51.100.20"     # local IP to send from (default: auto-detected)
local_tun   = "155.155.1.2/24"    # this side's tunnel IP                   [required]
peer_tun    = "155.155.1.1/24"    # the peer's tunnel IP (enables the health check)
tun_name    = "tun1"              # name of the TUN device (default: flagtun0)
mtu         = 1320                # inner packet size, 576–9000 (default: 1320)
tun_qdisc   = "fq"                # TUN queue discipline: fq, fq_codel, cake, pfifo_fast, pfifo, sfq, noqueue

transport      = "icmp"           # transport type — only "icmp" (ICMPv4) is supported
icmp_id        = 2001             # id on the ICMP packets, 1–65535 (MUST match on both ends; unique per tunnel on a host)
icmp_send_type = 0                # ICMP type sent: 8 = echo request, 0 = echo reply (client 8 / server 0)
icmp_recv_type = 8                # ICMP type accepted from the peer (client 0 / server 8)

health_port = 19001               # TCP port used to check the tunnel is alive, 1–65535
log_level   = "info"              # how chatty: debug | info | warn | error
stats_interval_secs = 30          # how often to log a traffic stats line, 0–3600 seconds

# Resources. workers defaults to all CPU cores; the rest are auto-sized.
# Leave these unless you specifically need to override.
workers               = 4         # parallel send workers (default: all CPU cores)
gomaxprocs            = 0         # max CPU cores this process may use (0 = all cores)
sock_buf_bytes        = 0         # socket send/recv buffer in bytes (0 = 128 MB)
tun_write_queue_depth = 0         # TUN write queue in packets (0 = auto, otherwise ≥ 512)
tun_write_workers     = 0         # number of TUN write workers (0 = auto)
tx_queue_len          = 0         # TUN device tx queue length (0 = 10000)

# Tuning — these are what the profile sets. Override only when you picked "custom".
busy_poll_us    = 0               # busy-poll the receiver for lower latency, 0–10000 µs (0 = off; >0 burns a CPU core)
dscp            = 0               # DSCP / QoS marking on outgoing packets, 0–63 (e.g. 46 = EF)
so_priority     = 6               # socket priority, 0–7 (higher = preferred by the qdisc)
recv_batch_size = 256             # packets read from the socket per cycle, 1–4096 (higher = more throughput, more latency)
```
