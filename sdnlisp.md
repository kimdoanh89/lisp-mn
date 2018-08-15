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
To verify the command set the rule run tc -s
```

```
## Testbed
For BGAN, AeroMACS delays, respectively
Forward link, on the router:
```
sudo tc qdisc add dev ens39 root netem delay 170ms
sudo tc qdisc add dev ens40 root netem delay 25ms
```
