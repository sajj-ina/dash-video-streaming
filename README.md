# DASH Video Streaming Project

## Description
DASH video streaming implementation using two Kali Linux VMs,
ffmpeg, Apache2, iperf3 and Linux Traffic Control (tc).

## Videos
- Bunny - Big Buck Bunny HD
- Steel - Steel HD

## Server Setup
```bash
sudo apt install ffmpeg apache2 iperf3 -y
sudo systemctl start apache2
```

## Transcode Videos
```bash
# 1.5 Mbps
ffmpeg -i input.mp4 -c:v libx264 -b:v 1500k -vf scale=1280:720 output_1500k.mp4

# 2.0 Mbps
ffmpeg -i input.mp4 -c:v libx264 -b:v 2000k -vf scale=1280:720 output_2000k.mp4

# 4.0 Mbps
ffmpeg -i input.mp4 -c:v libx264 -b:v 4000k -vf scale=1920:1080 output_4000k.mp4
```

## Create DASH Manifest
```bash
ffmpeg -i v1.mp4 -i v2.mp4 -i v3.mp4 \
  -map 0:v -map 1:v -map 2:v \
  -c:v copy -use_timeline 1 -use_template 1 \
  -seg_duration 4 -f dash manifest.mpd
```

## Traffic Control Scenarios

### TBF - Token Bucket Filter
```bash
sudo tc qdisc add dev eth0 root tbf rate 2.5mbit burst 20k latency 50ms
```

### HTB - Hierarchical Token Bucket
```bash
sudo tc qdisc add dev eth0 root handle 1: htb default 10
sudo tc class add dev eth0 parent 1: classid 1:1 htb rate 5mbit burst 20k
sudo tc class add dev eth0 parent 1:1 classid 1:10 htb rate 2.5mbit ceil 5mbit burst 20k
```

### Traffic Policing - Client Ingress
```bash
sudo tc qdisc add dev eth0 ingress
sudo tc filter add dev eth0 parent ffff: protocol ip u32 \
  match ip src 0.0.0.0/0 police rate 3.5mbit burst 100k drop flowid :1
```

## Reset TC Rules
```bash
sudo tc qdisc del dev eth0 root 2>/dev/null
sudo tc qdisc del dev eth0 ingress 2>/dev/null
```

## Access URLs
- Bunny: http://SERVER_IP/Bunny/manifest.mpd
- Steel: http://SERVER_IP/Steel/manifest.mpd
- Player: http://SERVER_IP/player.html

## MOS Results
| Scenario | Throughput | DASH Bitrate | Buffering | MOS |
|---|---|---|---|---|
| Baseline | 5.03 Mbps | 4.0 Mbps | No | 5 |
| TBF 2.5Mbps | 2.5 Mbps | 1.5 Mbps | Brief | 3 |
| HTB 2.5-5Mbps | 5.03 Mbps | 2.0 Mbps | No | 4 |
| Policing 3.5Mbps | 3.5 Mbps | 1.5 Mbps | Yes | 2-3 |

## Author
Your Name - Network Technologies 2026
