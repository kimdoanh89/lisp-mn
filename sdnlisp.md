- add a fixed amount of delay to all packets going through the ens37, ens38 interface
- this adds a delay to the egress scheduler, exclusively. If it were to add the delay to both the ingress and egress schedulers, the total delay would have doubled. In general, all of these traffic control rules are applied to the egress scheduler only.
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
- Forward link:
- On the router1:
```
sudo tc qdisc add dev ens38 root netem delay 170ms
```
- On the router2:
```
sudo tc qdisc add dev ens37 root netem delay 50ms
```
- Return link, on the aircraft:
```
sudo tc qdisc add dev ens37 root netem delay 170ms
sudo tc qdisc add dev ens38 root netem delay 50ms
```
