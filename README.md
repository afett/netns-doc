## How to setup a network namespace with iproute2

#### Create a veth pair

Create a veth pair with the device names host0 and guest0.
```
   ip link add name host0 type veth peer name guest0
```

#### Configure the the host interface

Set the host device up.
```
   ip link set host0 up
```

Add an address to the host device.
```
   ip address add 192.168.42.1/24 dev host0
```

Add a device route to the guest address via the host interface.
```
   ip route add 192.168.42.2/32 dev host0
```

Add additional routes to get the traffic from the guest to other
host interfaces.

#### Configure the guest interface

Create a network namespace named nsguest0.
```
   ip netns add nsguest0
```

Move the guest veth to the namespace nsguest0.
```
   ip link set guest0 netns nsguest0
```

From this point on the interface guest0 is no longer visible in the global network namespace.
Use `ip netns exec nsguest0 ip ....` to configure it. You can use `ip netns exec nsguest0 bash`
to get a bash running within this network namespace.

Optionally rename the guest interface (it may be the same name used in the global namespace).
```
   ip netns exec nsguest0 ip link set dev guest0 name foo0
```

Optionally change the mac address of the guest interface.
```
   ip netns exec nsguest0 ip link set dev foo0 address 52:54:00:0d:ae:35
```

Set the guest interface up.
```
   ip netns exec nsguest0 ip link set foo0 up
```

Add an ip address to the guest interface.
```
   ip netns exec nsguest0 ip address add 192.168.42.2/24 dev foo0
```

Add a default route to the guest interface.
```
   ip netns exec nsguest0 ip route add default via 192.168.42.1
```

Ping the gateway from the guest namespace.
```
   ip netns exec nsguest0 ping 192.168.42.1
```

You should see the traffic by running `tcpdump -eni host0` in the global network namespace.

#### Deconfigure the guest

Make sure there is no process (ie. ping) using this namespace anymore or you will get an
`device or resource busy` error.

Delete the guest namespace.
```
   ip netns delete nsguest0
```

Delete the veth pair.
```
   ip link delete host0
```

#### Misc

List all network namespaces.
```
   ip netns show
```
