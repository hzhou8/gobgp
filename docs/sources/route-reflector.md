# Route Reflector

This page explains how to set up GoBGP as a route reflector.

## Prerequisites

Assumed you finished [Getting Started](https://github.com/osrg/gobgp/blob/master/docs/sources/getting-started.md).

## Configuration

Configure `RouteReflector.RouteReflectorConfig` section to enable route reflector functionality.
The configuration below configures two route reflector clients and two normal iBGP peers.

```toml
[Global]
  [Global.GlobalConfig]
    RouterId = "192.168.0.1"
    As = 65000
[Neighbors]
  [[Neighbors.NeighborList]]
    [Neighbors.NeighborList.NeighborConfig]
      NeighborAddress = "192.168.10.2"
      PeerAs = 65000
    [Neighbors.NeighborList.RouteReflector.RouteReflectorConfig]
      RouteReflectorClient = true
      RouteReflectorClusterId = "192.168.0.1"
  [[Neighbors.NeighborList]]
    [Neighbors.NeighborList.NeighborConfig]
      NeighborAddress = "192.168.10.3"
      PeerAs = 65000
    [Neighbors.NeighborList.RouteReflector.RouteReflectorConfig]
      RouteReflectorClient = true
      RouteReflectorClusterId = "192.168.0.1"
  [[Neighbors.NeighborList]]
    [Neighbors.NeighborList.NeighborConfig]
      NeighborAddress = "192.168.10.4"
      PeerAs = 65000
  [[Neighbors.NeighborList]]
    [Neighbors.NeighborList.NeighborConfig]
      NeighborAddress = "192.168.10.5"
      PeerAs = 65000
```

## Check route reflector behavior

Let's check adj-rib-out of a route reflector client.

```bash
$ gobgp neighbor 192.168.10.2 adj-out
Network              Next Hop             AS_PATH              Attrs
10.0.2.0/24          192.168.10.3                              [{Origin: i} {Med: 0} {LocalPref: 100} {Originator: 192.168.0.3} {ClusterList: [192.168.0.1]}]
10.0.3.0/24          192.168.10.4                              [{Origin: i} {Med: 0} {LocalPref: 100} {Originator: 192.168.0.4} {ClusterList: [192.168.0.1]}]
10.0.4.0/24          192.168.10.5                              [{Origin: i} {Med: 0} {LocalPref: 100} {Originator: 192.168.0.5} {ClusterList: [192.168.0.1]}]
```

You can see the routes from other iBGP peers are reflected.
Also Originator and ClusterList path attributes are added.

For the normal iBGP peer's adj-rib-out

```bash
$ gobgp neighbor 192.168.10.4 adj-out
Network              Next Hop             AS_PATH              Attrs
10.0.1.0/24          192.168.10.2                              [{Origin: i} {Med: 0} {LocalPref: 100}]
10.0.2.0/24          192.168.10.3                              [{Origin: i} {Med: 0} {LocalPref: 100}]
```

Only the routes from route reflector clients are advertised via GoBGP.
Originator and ClusterList path attributes are not added.
