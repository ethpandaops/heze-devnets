**Title:** Outbound dials use QUIC only when a peer advertises it, with no TCP fallback, and failures are logged only at TRACE

### Description

Since the experimental QUIC transport landed (#10805) and QUIC was enabled by default, `MultiaddrUtil.fromDiscoveryPeer` builds a QUIC multiaddr whenever the discovered peer's ENR has a `quic` field and the local node has QUIC enabled. Teku dials only that address. If the QUIC handshake times out, the attempt counts as failed and the peer is retried over QUIC again. It never tries the peer's advertised TCP address.

When a peer advertises a QUIC port that isn't actually reachable (for example, the client listens on UDP 9001 in a container but the port isn't published, or a firewall drops it), Teku can't make **any** outbound connection to that peer, even though TCP works. On a network where every peer is like that, Teku ends up with zero outbound peers and depends entirely on inbound connections.

The failures are also hard to see: the per-attempt error is logged only at TRACE (`ConnectionManager.java:239`), so at INFO or DEBUG the node just reports "No connected peers" with no reason.

### Observed on

- Teku `v26.9.1+51-g58e88c7319` (`ethpandaops/teku:focil-devnet-0`), QUIC enabled by default
- focil-devnet-0: Teku and Lodestar nodes advertise `quic=9001` in their ENR, but their containers didn't publish UDP 9001
- Every Teku node on the devnet showed **0 outbound / 14 inbound** peers. All of its peers were Lodestar nodes dialling in over TCP.
- Reproduced locally in Kurtosis against the same devnet with the same result: 0 peers with QUIC enabled, 10 peers with `--Xp2p-quic-enabled=false`.

### Evidence

Metrics after about 2 minutes:

```
network_peer_connection_attempt_count_total{status="attempted"} 720.0
network_peer_connection_attempt_count_total{status="failed"} 720.0
network_peer_connection_attempt_count_total{status="successful"} 0.0
```

DEBUG only shows the dial, which always targets the QUIC address even though the peer also advertises TCP:

```
DEBUG - Connecting to 16Uiu2HAm3oZVi4... (/ip4/209.38.96.64/udp/9001/quic-v1/p2p/16Uiu2HAm3oZVi4...)
```

The actual failure, visible only with `tech.pegasys.teku.networking.p2p.connection` at TRACE:

```
TRACE - Failed to connect to peer: 16Uiu2HAmA72iJ1RAZtm3YJsiHHyVgLo224vytyQcFGypDzuU8TMm
java.util.concurrent.CompletionException: io.libp2p.etc.types.NonCompleteException: java.util.concurrent.CompletionException: io.netty.channel.ConnectTimeoutException: connection timed out: QuicConnectionAddress{EPHEMERAL} -> QuicConne...
    at io.netty.handler.codec.quic.QuicChannelBootstrap.lambda$connect$0(QuicChannelBootstrap.java:269)
```

The same peers accept TCP from Lodestar, and from Teku once QUIC is disabled, so the network path itself works. The path MTU was also fine: 1440-byte packets with Don't-Fragment set got through.

### Steps to reproduce

1. Run Teku with QUIC enabled (the default) on a network where peers advertise `quic` in their ENR but their QUIC port is unreachable while TCP is reachable.
2. Watch `beacon_peer_count`: no outbound peers, and `network_peer_connection_attempt_count_total{status="failed"}` keeps climbing.
3. Restart with `--Xp2p-quic-enabled=false`: outbound peers connect over TCP right away.

### Expected behaviour

- If a QUIC dial fails or times out, fall back to the peer's advertised TCP address (or dial both and keep whichever connects first), rather than retrying QUIC only.
- Log connection failures at DEBUG with the cause, so "no peers" problems can be diagnosed without TRACE.

### Relevant code

- `networking/p2p/src/main/java/tech/pegasys/teku/networking/p2p/libp2p/MultiaddrUtil.java:34`: picks QUIC exclusively when `localNodeQuicEnabled` and the peer has a QUIC address
- `networking/p2p/src/main/java/tech/pegasys/teku/networking/p2p/connection/ConnectionManager.java:239`: failure logged at TRACE

### Context

We fixed the infrastructure side by publishing the QUIC ports (ethpandaops/ansible-collection-general#591). But any network with an advertised but unreachable QUIC port, which is common behind NAT, Docker or restrictive firewalls, will strand Teku in the same way.
