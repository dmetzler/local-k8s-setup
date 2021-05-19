# A Local Kubernetes Setup With Gitops

This repo aims at holding code and documentation on how to configure a local (Mac OS first) deployment of Kubernetes that follows Gitops principles.

## Resources

 * https://passingcuriosity.com/2013/dnsmasq-dev-osx/
 * https://medium.com/localz-engineering/kubernetes-traefik-locally-with-a-wildcard-certificate-e15219e5255d


## Mac Host setup

### Installing DNSMasq

Installing `DNSMasq` and making it your DNS server allows to route addresse in `*.k8s.dev` to `127.0.0.1`, i.e. you own computer.

```shell
# Install dnsmasq
brew install dnsmasq
# Copy the default configuration file.
cp $(brew list dnsmasq | grep /dnsmasq.conf.example$) /usr/local/etc/dnsmasq.conf
# Copy the daemon configuration file into place.
sudo cp $(brew list dnsmasq | grep /homebrew.mxcl.dnsmasq.plist$) /Library/LaunchDaemons/
# Start Dnsmasq automatically.
sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
echo "address=/dev/127.0.0.1" >> /usr/local/etc/dnsmasq.conf
sudo brew services restart dnsmasq
# Create a client for .dev domain
mkdir -p /etc/resolver
echo > /etc/resolver/dev <<EOT
nameserver 127.0.0.1
EOT

ping -c 1 test.k8s.dev
```


### Installing mkcert And Generating A Self Signed Trusted Certificate

`mkcert` is a tool that allows to install a Certificate Authorithy (CA) in your keychain and then provision TLS certificates using that CA. That way, your browser will `trust` your own self signed certificate locally>

We will create an `infra` namespace in k8s and deploy that certificate there

```shell
brew install mkcert
mkcert --install
mkcert '*.k8s.dev'
kubectl create ns infra
kubectl -n infra create secret tls traefik-tls-cert --key=_wildcard.k8s.dev-key.pem --cert=_wildcard.k8s.dev.pem
rm -f _wildcard.k8s.dev-key.pem _wildcard.k8s.dev.pem
```

### Installing ArgoCD

We will use ArgoCD to install everything from now on.

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Deploy with Gitops

Let's deploy now our first ArogCD application
```shell
kubectl apply -n argocd -f https://raw.githubusercontent.com/dmetzler/loca-k8s-setup/infra-app.yaml
```
