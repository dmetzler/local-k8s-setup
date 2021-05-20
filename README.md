# A Local Kubernetes Setup With Gitops

This repo aims at holding code and documentation on how to configure a local (Mac OS first) deployment of Kubernetes that follows Gitops principles.

## Resources

 * https://passingcuriosity.com/2013/dnsmasq-dev-osx/
 * https://medium.com/localz-engineering/kubernetes-traefik-locally-with-a-wildcard-certificate-e15219e5255d

## Installing DNSMasq

Installing `DNSMasq` and making it your DNS server allows to route addresse in `*.k8s.local` to `127.0.0.1`, i.e. you own computer.

### Linux

Hopefully you're already using NetworkManager. In that case, you just need to get dnsmasq and setup NetworkManager to use it

* https://pkgs.org/search/?q=dnsmasq

```shell
cat <<EOF | sudo tee /etc/NetworkManager/conf.d/dns.conf
[main]
dns=dnsmasq
EOF
cat <<EOF | sudo tee /etc/NetworkManager/dnsmasq.d/local.conf
address=/.local/127.0.0.1
EOF
sudo systemctl restart NetworkManager
```

### Mac

```shell
# Install dnsmasq
brew install dnsmasq
# Copy the default configuration file.
cp $(brew list dnsmasq | grep /dnsmasq.conf.example$) /usr/local/etc/dnsmasq.conf
# Copy the daemon configuration file into place.
sudo cp $(brew list dnsmasq | grep /homebrew.mxcl.dnsmasq.plist$) /Library/LaunchDaemons/
# Start Dnsmasq automatically.
sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
echo "address=/k8s.dev/127.0.0.1" >> /usr/local/etc/dnsmasq.conf
sudo brew services restart dnsmasq
# Create a client for k8s.dev domain
mkdir -p /etc/resolver
echo > /etc/resolver/local <<EOT
nameserver 127.0.0.1
EOT

ping -c 1 test.k8s.local
```


## Installing mkcert And Generating A Self Signed Trusted Certificate

`mkcert` is a tool that allows to install a Certificate Authorithy (CA) in your keychain and then provision TLS certificates using that CA. That way, your browser will `trust` your own self signed certificate locally>

* https://github.com/FiloSottile/mkcert

We will create an `infra` namespace in k8s and deploy that certificate there

```shell
mkcert --install
mkcert '*.k8s.local'
kubectl create ns infra
kubectl -n infra create secret tls traefik-tls-cert --key=_wildcard.k8s.local-key.pem --cert=_wildcard.k8s.local.pem
rm -f _wildcard.k8s.local-key.pem _wildcard.k8s.local.pem
```

## Installing ArgoCD

We will use ArgoCD to install everything from now on.

```shell
# Create ArgoCD namespace
kubectl create namespace argocd
# Deploy latest stable install
kubectl apply -n argocd -k infra-cd/argocd
```

## Deploy with Gitops

Let's deploy now our first ArgoCD application
```shell
kubectl apply -n argocd -f https://raw.githubusercontent.com/pgmillon/local-k8s-setup/main/infra-app.yaml
```

## Expose ArgoCD thru Traefik

```shell
kubectl apply -f infra-cd/argocd/argocd-ingress-route.yaml
```





