# A Local Kubernetes Setup With Gitops

This repo aims at holding code and documentation on how to configure a local (Mac OS first) deployment of Kubernetes that follows Gitops principles.

## Resources

 * https://medium.com/localz-engineering/kubernetes-traefik-locally-with-a-wildcard-certificate-e15219e5255d


## Mac Host setup

### Installing DNSMasq

Installing `DNSMasq` and making it your DNS server allows to route addresse in `*.k8s.local` to `127.0.0.1`, i.e. you own computer.

```shell
brew install dnsmasq
sudo brew services start dnsmasq
echo "address=/k8s.local/127.0.0.1" >> /usr/local/etc/dnsmasq.conf
sudo brew services restart dnsmasq
mkdir -p /etc/resolver
echo > /etc/resolver/dev <<EOT
nameserver 127.0.0.1
server=8.8.8.8
EOT
networksetup -setdnsservers Wi-Fi 127.0.0.1 8.8.8.8 8.8.4.4

ping -c 1 test.k8s.local
```


### Installing mkcert And Generating A Self Signed Trusted Certificate

`mkcert` is a tool that allows to install a Certificate Authorithy (CA) in your keychain and then provision TLS certificates using that CA. That way, your browser will `trust` your own self signed certificate locally>

We will create an `infra` namespace in k8s and deploy that certificate there

```shell
brew install mkcert
mkcert --install
mkcert '*.k8s.local'
kubectl create ns infra
kubectl -n infra create secret tls traefik-tls-cert --key=_wildcard.k8s.local-key.pem --cert=_wildcard.k8s.local.pem
rm -f _wildcard.k8s.local-key.pem _wildcard.k8s.local.pem
```
