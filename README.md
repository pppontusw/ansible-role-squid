# Sets up a basic Squid HTTP proxy.

## What does it do?

1. Installs Squid
2. Configures local networks to accept connections from as an open proxy OR sets up a whitelist/blacklist proxy depending on what variables you define

Can also be set up to serve WPAD configuration with NGINX, just set up the variable like:

```
http_proxy_wpad: |
 function FindProxyForURL(url, host) {
 
  if (isPlainHostName(host)) 
    return "DIRECT";
  
  if (url.substring(0, 4)=="ftp:")
    return "DIRECT";
 
  if (isInNet(dnsResolve(host), "10.0.0.0", "255.0.0.0") ||
  isInNet(dnsResolve(host), "172.16.0.0",  "255.240.0.0") ||
  isInNet(dnsResolve(host), "192.168.0.0",  "255.255.0.0") ||
  isInNet(dnsResolve(host), "127.0.0.0", "255.255.255.0"))
    return "DIRECT";
  
  return "PROXY HOSTNAME:PORT";
 }
```

You can also use this together with my haproxy and keepalived role to set up a virtual IP that will serve haproxy+wpad and if the host or any service dies on the host, the vIP will be taken over by a functioning server.

## What do you have to do?

1. Check the defaults/main configuration
2. Change the variables according to how you want them, either in defaults/main or by setting vars in the play
3. Look at the example playbooks and read the rest of the documentation

## How can I use this role?

We can use this role to enforce our configurations across basic http proxies.

For example this will set up a squid proxy that accepts all requests from RFC private networks:

```
#example-open-playbook.yml
# sets up a squid proxy that accepts all requests from RFC private networks

- name: Squid (open private)
  hosts: http-proxy-open
  become: true
  vars: 
    - squid_acl_localnet:
      - 10.0.0.0/8
      - 172.16.0.0/16
      - 192.168.0.0/16
  roles:
    - http-proxy
```

``` ansible-playbook example-open-playbook.yml ```

Or we can set up a proxy that will accept requests only if the destination domain is in whitelist.j2 but deny them if the destination domain is also in blacklist.j2.

NOTE: If you use this you probably want to ensure that your squid_http_port is not available from outside as you will only limit connections based on dstdomain, anyone who has access to this can use your proxy and that is almost certainly not what you want!

```
#example-strict-playbook.yml
# sets up a strict proxy with whitelists and blacklists from the templates/default folder
- name: Squid strict
  hosts: http-proxy-strict
  become: true
  vars:
    - squid_dst_whitelist: |
     .google.com
    - squid_acl_blacklist: |
     .facebook.com
  roles:
    - http-proxy
```

``` ansible-playbook example-open-playbook.yml ```


## Compability

Tested on:

Debian 6, 7 & 8

Ubuntu 14.04

CentOS 7