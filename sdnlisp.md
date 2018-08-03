add a fixed amount of delay to all packets going through the ens37, ens38 interface
```
sudo tc qdisc add dev ens37 root netem delay 340ms
sudo tc qdisc add dev ens38 root netem delay 40ms
```
delete the netem, routing setting
```
sudo tc qdisc del dev ens37 root netem
sudo tc qdisc del dev ens38 root netem
```