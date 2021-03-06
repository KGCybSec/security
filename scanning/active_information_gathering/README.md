# Active Information Gathering

Once the passive phase is over it is time to move to the active phase. In this phase we start interacting with the target.

## Netdiscover

This tool is used to scan a network for live machines.

```text
netdiscover -r 192.168.1.1/24
```

## Nikto

Nikto is a good tool to scan webservers. It is very intrusive.

```text
nikto -host 192.168.1.101
# Using a proxy
nitko -host http://192.168.124.130 -useproxy 192.168.124.130:3128
```

## References

[https://blog.bugcrowd.com/discovering-subdomains](https://blog.bugcrowd.com/discovering-subdomains)

[https://high54security.blogspot.cl/2016/01/recon-ng-and-power-to-crawl-trough.html](https://high54security.blogspot.cl/2016/01/recon-ng-and-power-to-crawl-trough.html)

