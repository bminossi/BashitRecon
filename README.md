# BashitRecon

## Phoenix file

Common passive ways to extract subdomains using a domain as input entrie.

## CIDR to ip range with ipcalc
```
prips() {
read -r a b c d e f g h <<< $(
ipcalc -nb "$1" | awk '/HostM/{print $2}' | tr '\n .' ' '
);
eval "echo {$a..$e}.{$b..$f}.{$c..$g}.{$d..$h}"
}

```
## Filtering by just internal IPs

xargs -a hostslist -I@ sh -c 'ip=$(dig +short @); [ -z "${ip##*10.*}" ] && echo @'

## Filtering by up hosts

xargs -P 500 -a hostslist -I@ sh -c 'dig @ | grep -q NOERROR 1>/dev/null && echo | echo @;'

xargs -P 500 -a hostslist -I@ sh -c 'nc -w1 -z -v @ 80 2>/dev/null && echo @'

## Extract Only Http using gospider (required anew instalation)

xargs -P 500 -a hostslist -I@ sh -c 'nc -w1 -z -v @ 443 2>/dev/null && echo @' | xargs -I@ -P10 sh -c 'gospider -a -s "https://@" -d 2 | grep -Eo "(http|https)://[^/\\"]+" | anew'

## Extract only JS using gospider (required anew instalation)

xargs -P 500 -a hostslist -I@ sh -c 'nc -w1 -z -v @ 8443 2>/dev/null && echo @' | xargs -I@ -P10 sh -c 'gospider -a -s "https://@" -d 2 | grep -Eo "(http|https)://[^/\\"].*.js+" | sed "s#\] \- #\n#g" | anew'

## Extract only using openssl (required anew installation)
xargs -P100 -a hostslist -I@ sh -c 'ip=$(dig +short @);[ ! -z "$ip" ] && printf "GET / HTTP/1.1\r\nHost: $ip\r\n\r\n" | timeout 2 openssl s_client -connect $ip:443 2>/dev/null' | sed 's# \|/\|=#\n#g' | grep paypal | anew

## Geting domains using reverse DNS

### Command

xargs -P 500 -a hostslist -I@ sh -c 'dig @' 2>/dev/null | awk -F'<<>>' '{print $3}' | xargs -n1 | tee -a hosts

## Getting domains wich resolve to some IP (avoid false positives)

### Command

cat hostUnicos.txt | while read line;do xargs -P 500 -a ../../subbrute/names_small.txt -I@ sh -c "dig +noidnout +noidnin +short @.$line | grep -c '^' 1>/dev/null && echo @.$line | tee -a hostsDig";done

## Geting subdomains by ssl

### Command

xargs -P 100 -a hostUnicos -I@ sh -c 'sanssl @ 2>/dev/null | grep -v "No domains"'

Source: https://raw.githubusercontent.com/appsecco/the-art-of-subdomain-enumeration/master/san_subdomain_enum.py

## Getting subdomains by all Dns records and brute subdomain

### Command

dnsrecon -d alertmanager-1.staging.rtcdn.caffeine.tv -D /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t brt~
(Doest seems to be so useful)

### Command

altdns -t 600 -w /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -i hostUnicos.txt -o hosts -r -s results_output.txt
