# ton-k8s

Docker images, python mini-lib, helm chart for comfortable [TON](https://ton.org) infrastructure


### Own-net locally

Run:

```
./pre-build.sh && docker-compose -f ./composes/ownnet.yaml up
```
More info about ownnet - [ownnet.md](ownnet.md)

(you need to wait some time while network is starting)

Change local validators count:

```
deploy:
    replicas: 3
```

If you want to reset your private network - remove all volumes:

```
docker-compose -f ./composes/ownnet.yaml down -v --rmi all
```

## Features

| Feature name                       | Status |
|------------------------------------|--------|
| Full node for mainnet / testnet    | ✅      |
| Lite-client for mainnet / testnet  | ✅      |
| Helm chart                         | ✅      |
| K8s secrets for keys               | ✅      |
| Custom ton network                 | ✅      |
| Compose                            | ✅      |
| RPC API / ton-http-api             | ✅      |
| Jupyter in toncenter with examples | ✅      |
| Punisher on validator nodes        | ⌛      |
| Status page                        | ⌛      |
| K8s resource limits                | ⌛      |

### Local build

It's one of the easiest way to start TON network locally:

```
./pre-build.sh
```

### Files

`pre-build.sh` - build all docker files and send them to registry

`ton-compile-source` - main Docker image, compile [TON](`https://github.com/newton-blockchain/ton/`) sources. You can
pass`is_testnet`. If `true` - compile and build `safer_overlay` branch,
because [testnet is working on it](https://t.me/testnetstatus/3).

`ton-full-node` - run mainnet / testnet full node, you need to pass `version` build argument `mainnet-v0`
or `testnet-v0`

## Helm / k8s

1. Change registry in build_for_k8s.sh
2. Run build_for_k8s.sh and push images to registry
3. Change `./chart/values.yaml` to specify your needs:
   1. Change `run` section to chose which images you want to run
   2. Change `persistent`, add your storageClassName / create persistent
   3. Change `imagePullSecrets` to your registry
   4. Notice that ports in nodes used `30001-30006` nodePorts that will open in all your nodes (to get UDP traffic to local TON stuff)
   5. Change `testnetToncenter.CONFIG` / `toncenter.CONFIG` params to your domains
   6. Change `ingress` to correct setup of domains

Then:

```bash
kubectl create namespace ton
helm upgrade --install --namespace ton ton ./chart/ --values ./chart/values.yaml 
```

### Tips and tricks

After publish UDP services to k8s you need to specify `externalIp` to bind public port.
[Read more about externalIp](https://kubernetes.io/docs/concepts/services-networking/service/#external-ips)

## ENVIRON

Feel free to change environs in compose / helm

```
config = {
    "PUBLIC_IP": os.getenv('PRIVATE_CONFIG', ip),
    "CONFIG": os.getenv('CONFIG', 'https://test.ton.org/ton-global.config.json'),
    "PRIVATE_CONFIG": os.getenv('PRIVATE_CONFIG', 'false') == 'true',
    "LITESERVER": os.getenv('LITESERVER', 'true') == 'true',  # convert to bool
    "CONSOLE_PORT": int(os.getenv("CONSOLE_PORT", 46732)),
    "PUBLIC_PORT": int(os.getenv("PUBLIC_PORT", 50001)),
    "DHT_PORT": int(os.getenv("DHT_PORT", 6302)),
    "LITESERVER_PORT": int(os.getenv("LITESERVER_PORT", 43680)),
    "NAMESPACE": os.getenv("NAMESPACE", None),
    "THREADS": int(os.getenv("CPU_COUNT", cpu_count)),
    "GENESIS": os.getenv("GENESIS", False),
    "VERBOSE": os.getenv("VERBOSE", 3)
}
```
