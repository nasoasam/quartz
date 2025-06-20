
https://github.com/wagoodman/dive

Cloud Shellの場合

```sh
DIVE_VERSION=$(curl -sL "https://api.github.com/repos/wagoodman/dive/releases/latest" | grep '"tag_name":' | sed -E 's/.*"v([^"]+)".*/\1/')
curl -fOL "https://github.com/wagoodman/dive/releases/download/v${DIVE_VERSION}/dive_${DIVE_VERSION}_linux_amd64.rpm"
rpm -i dive_${DIVE_VERSION}_linux_amd64.rpm
```

sudo rpm?
