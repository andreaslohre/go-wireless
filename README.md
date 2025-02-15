# go-wireless

![Go](https://github.com/plantiga/go-wireless/workflows/Go/badge.svg) [![Go Report Card](https://goreportcard.com/badge/github.com/plantiga/go-wireless)](https://goreportcard.com/report/github.com/plantiga/go-wireless) [![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/github.com/plantiga/go-wireless)

A way to interact with the Wireless interfaces on a Linux machine using WPA Supplicant.

## Requirements

Requires a running wpa_supplicant with control interface at `/var/run/wpa_supplicant` (which is usually 
a symlink to `/run/wpa_supplicant`).  This requires the config file to contain the line:

```
ctrl_interface=DIR=/run/wpa_supplicant GROUP=wheel
```

Or for the `wpa_supplicant` instance to be running with the `-O /run/wpa_supplicant` argument.

You will probably also need to be running as root unless you are in the specified group (`wheel` in the above example).

# Usage

Get a list of wifi cards attached:

```golang
ifaces := wireless.Interfaces()
```

From there you can use the client:

```golang
wc, err := wireless.NewClient("wlan0")
defer wc.Close()
```

Get a list of APs that are in range:

```golang
aps, err := wc.Scan()
fmt.Println(aps, err)
ap, ok := wireless.APs(aps).FindBySSID("CIA Predator Drone 237A")
```

Get a list of known networks (**note:** the password cannot be retrieved so are not populated):

```golang
nets, err := wc.Networks()
fmt.Println(nets, err)
```

Connect to networks:

```golang
net := NewNetwork("FBI Surveillance Van #4", "secretpass")
net, err := wc.Connect(net)
```

Disable networks:

```golang
nets, err:= wc.Networks()
net, err := net, ok := wireless.Networks(nets).FindBySSID("FBI Surveillance Van #4")
net.Disable(true)
net, err := wc.UpdateNetwork(net)
```

Subscribe to events:

```golang
sub := wc.Subscribe(wireless.EventConnected, wireless.EventAuthReject, wireless.EventDisconnected)

ev := <-sub.Next()
switch ev.Name {
	case wireless.EventConnected:
		fmt.Println(ev.Arguments)
	case wireless.EventAuthReject:
		fmt.Println(ev.Arguments)
	case wireless.EventDisconnected:
		fmt.Println(ev.Arguments)
}
```

Check the status of the connection:

```golang
st, err := wc.Status()
fmt.Printf("%+v\n", st)
```
