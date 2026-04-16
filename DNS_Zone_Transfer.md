First of all, a DNS Zone Transfer is a mechanism for replicating domain name databases (zones) from a primary (master) DNS server to a secondary (slave) server to ensure record redundancy and consistency. 
Although designed for network administration, this protocol presents a major security risk if not properly configured, as it does not offer any authentication by default (according to the original specifications), allowing any client to retrieve the complete list of hosts in a domain.


So for this example, the domain is zonetransfer.me. Lets see if we can find some name servers :

```
┌─[eu-dedivip-1]─[10.10.15.165]─[ys4@htb-cxbzkky4ri]─[~]
└──╼ [★]$ nslookup -type=NS zonetransfer.me
Server:		1.1.1.1
Address:	1.1.1.1#53

Non-authoritative answer:
zonetransfer.me	nameserver = nsztm2.digi.ninja.
zonetransfer.me	nameserver = nsztm1.digi.ninja.
```

The two name servers are nsztm1.digi.ninja and nsztm2.digi.ninja.

With dig axfr we can get all of the DNS record types for this domain :

```
┌─[eu-dedivip-1]─[10.10.15.165]─[ys4@htb-cxbzkky4ri]─[~]
└──╼ [★]$ dig axfr @nsztm1.digi.ninja zonetransfer.me    	# or dig @nsztm2.digi.ninja -t axfr zonetransfer.me

; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> axfr @nsztm1.digi.ninja zonetransfer.me
; (1 server found)
;; global options: +cmd
zonetransfer.me.	7200	IN	SOA	nsztm1.digi.ninja. robin.digi.ninja. 2019100801 172800 900 1209600 3600
zonetransfer.me.	7200	IN	DNSKEY	256 3 7 AwEAAapoL+InQBYx2oi3dI424+dEDFgnVW0cOINfCY3jLrngZxBsEur8 ByhMOQsxoIOYu/7b3c8tj2BwlQquqxZe79QHSW78fK7D+bP/8AosnBG5 K5gJXEvphEtJ9x8/X0Y971XaW9lLmtJ6h4AXsrbgTr2g9KOiPSIbvDPM W8qLMaQkTm89hvPc+NuzrOEOPNhoXs/iPM+SQzrvTBfr6y0w2yPtYYdW I1kN76OQBxh0xjIdlyT0QKiohKq2bybPROJO7K3NlDc8oaOZoXH5/RfL DQzxzXyYSV8fLwimUeulo7YA11I/AHQ7DsUsFu2S2vxGCyR8nmx9gYbN 4sBvTF2i5eM=
zonetransfer.me.	301	IN	TXT	"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me.	7200	IN	MX	0 ASPMX.L.GOOGLE.COM.
zonetransfer.me.	7200	IN	MX	10 ALT1.ASPMX.L.GOOGLE.COM.
zonetransfer.me.	7200	IN	MX	10 ALT2.ASPMX.L.GOOGLE.COM.
zonetransfer.me.	7200	IN	MX	20 ASPMX2.GOOGLEMAIL.COM.
zonetransfer.me.	7200	IN	MX	20 ASPMX3.GOOGLEMAIL.COM.
zonetransfer.me.	7200	IN	MX	20 ASPMX4.GOOGLEMAIL.COM.
zonetransfer.me.	7200	IN	MX	20 ASPMX5.GOOGLEMAIL.COM.
zonetransfer.me.	7200	IN	A	5.196.105.14
zonetransfer.me.	7200	IN	NS	nsztm1.digi.ninja.
zonetransfer.me.	7200	IN	NS	nsztm2.digi.ninja.
zonetransfer.me.	7200	IN	CERT	PKIX 0 0 MIIDvTCCAqUCFHh5BGzOrlYrXo5h90ipm0aDUEz9MA0GCSqGSIb3DQEB CwUAMIGaMQswCQYDVQQGEwJHQjEYMBYGA1UECAwPU291dGggWW9ya3No aXJlMRIwEAYDVQQHDAlTaGVmZmllbGQxEjAQBgNVBAoMCURpZ2luaW5q YTEQMA4GA1UECwwHSGFja2luZzEYMBYGA1UEAwwPem9uZXRyYW5zZmVy Lm1lMR0wGwYJKoZIhvcNAQkBFg56dG1AZGlnaS5uaW5qYTAeFw0yNTA3 MDIxMzU1MTNaFw0yNjA3MDIxMzU1MTNaMIGaMQswCQYDVQQGEwJHQjEY MBYGA1UECAwPU291dGggWW9ya3NoaXJlMRIwEAYDVQQHDAlTaGVmZmll bGQxEjAQBgNVBAoMCURpZ2luaW5qYTEQMA4GA1UECwwHSGFja2luZzEY MBYGA1UEAwwPem9uZXRyYW5zZmVyLm1lMR0wGwYJKoZIhvcNAQkBFg56 dG1AZGlnaS5uaW5qYTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoC ggEBALzYVM9WlBqOKU1lmnKJkKdIEZOhkscHQktEJORXCismSWV3Ffbs Lw7D3sfCc0h9ecZglsYvFUmEM0I0noYtuHPAlF2+FotVuoFrYuMYrEQo Zs4kuORIEx8pwHMZQUSM6KwVVLIB/FE956GfovgxGxWs33QaTKATAVCh D9KTLf6wVh/eC+0GI6mbvGvjqZFmmV/SYmmkdqEBWB7q3+SByfVrUohC A2GO30dwk6vUBtIj+J+i4SzKzLXIvFEfbCirMPQvdflgwPbjwp+cWG7o UBvfQZfZbaTp+9+V8FoBl0f8fGj/Mae1n0rSV5hnuXot8d3PAoAWQtW3 HJUv1nEboAMCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAXop6ftpV2/r7 tkXqFCsMwub7ZBd12U14nsBon+X7K5Nr6obrVAtnWO+XwD8x2UgvYIQB uRLK9LOX6VYoiWMVrItIN8KRSsin5eJe4tzewsNGrVtkVbbKULViCeBt DgmImk8rkZeWU1uNOsq0t/wd3GUZe2CM9DpKVhPFhc9Uq3pYbAsidYlp SApuuj8ka3L+VruzJVwveyKTUkWAsN1iSv7BGgEF0039WW3IEv1ZP81c AdWFy1fx+tuteM6Iz5xkx1tp0/eLtb39cnKFQnrs8itDG2j3yBc3CClY mw4NNU2nODN4COt7uzXBez6iIFSNqQjVyFyomtPn4ae0cYRHEw==
zonetransfer.me.	300	IN	HINFO	"Casio fx-700G" "Windows XP"
_acme-challenge.zonetransfer.me. 301 IN	TXT	"6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"
_sip._tcp.zonetransfer.me. 14000 IN	SRV	0 0 5060 www.zonetransfer.me.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me. 7200	IN PTR www.zonetransfer.me.
asfdbauthdns.zonetransfer.me. 7900 IN	AFSDB	1 asfdbbox.zonetransfer.me.
asfdbbox.zonetransfer.me. 7200	IN	A	127.0.0.1
asfdbvolume.zonetransfer.me. 7800 IN	AFSDB	1 asfdbbox.zonetransfer.me.
canberra-office.zonetransfer.me. 7200 IN A	202.14.81.230
cmdexec.zonetransfer.me. 300	IN	TXT	"; ls"
contact.zonetransfer.me. 2592000 IN	TXT	"Remember to call or email Pippa on +44 123 4567890 or pippa@zonetransfer.me when making DNS changes"
dc-office.zonetransfer.me. 7200	IN	A	143.228.181.132
deadbeef.zonetransfer.me. 7201	IN	AAAA	dead:beaf::
dr.zonetransfer.me.	300	IN	LOC	53 20 56.558 N 1 38 33.526 W 0.00m 1m 10000m 10m
DZC.zonetransfer.me.	7200	IN	TXT	"AbCdEfG"
email.zonetransfer.me.	2222	IN	NAPTR	1 1 "P" "E2U+email" "" email.zonetransfer.me.zonetransfer.me.
email.zonetransfer.me.	7200	IN	A	74.125.206.26
Hello.zonetransfer.me.	7200	IN	TXT	"Hi to Josh and all his class"
home.zonetransfer.me.	7200	IN	A	127.0.0.1
Info.zonetransfer.me.	7200	IN	TXT	"ZoneTransfer.me service provided by Robin Wood - robin@digi.ninja. See http://digi.ninja/projects/zonetransferme.php for more information."
internal.zonetransfer.me. 300	IN	NS	intns1.zonetransfer.me.
internal.zonetransfer.me. 300	IN	NS	intns2.zonetransfer.me.
intns1.zonetransfer.me.	300	IN	A	81.4.108.41
intns2.zonetransfer.me.	300	IN	A	5.196.105.10
office.zonetransfer.me.	7200	IN	A	4.23.39.254
ipv6actnow.org.zonetransfer.me.	7200 IN	AAAA	2001:67c:2e8:11::c100:1332
owa.zonetransfer.me.	7200	IN	A	207.46.197.32
robinwood.zonetransfer.me. 302	IN	TXT	"Robin Wood"
rp.zonetransfer.me.	321	IN	RP	robin.zonetransfer.me. robinwood.zonetransfer.me.
sip.zonetransfer.me.	3333	IN	NAPTR	2 3 "P" "E2U+sip" "!^.*$!sip:customer-service@zonetransfer.me!" .
sqli.zonetransfer.me.	300	IN	TXT	"' or 1=1 --"
sshock.zonetransfer.me.	7200	IN	TXT	"() { :]}; echo ShellShocked"
staging.zonetransfer.me. 7200	IN	CNAME	www.sydneyoperahouse.com.
alltcpportsopen.firewall.test.zonetransfer.me. 301 IN A	127.0.0.1
testing.zonetransfer.me. 301	IN	CNAME	www.zonetransfer.me.
vpn.zonetransfer.me.	4000	IN	A	174.36.59.154
www.zonetransfer.me.	7200	IN	A	5.196.105.14
xss.zonetransfer.me.	300	IN	TXT	"'><script>alert('Boo')</script>"
zonetransfer.me.	7200	IN	SOA	nsztm1.digi.ninja. robin.digi.ninja. 2019100801 172800 900 1209600 3600
;; Query time: 16 msec
;; SERVER: 81.4.108.41#53(nsztm1.digi.ninja) (TCP)
;; WHEN: Thu Apr 16 01:50:50 CDT 2026
;; XFR size: 52 records (messages 1, bytes 3339)
```

Yeah there is a lot of informations but trust me, there is also a lot of important informations we can found in there :

The SOA record reveals the primary name server (nsztm1.digi.ninja), the administrator’s email (robin.digi.ninja → robin@digi.ninja when turning the first dot into @), the serial number (often date‑based), and key DNS timing values (refresh, retry, expiration, TTL).
- The email can be used for social‐engineering or phishing (for example, impersonating a DNS admin).
- If the serial changes regularly, it can hint at internal DNS activity or configuration updates.

The NS records list the authoritative DNS servers (nsztm1.digi.ninja and nsztm2.digi.ninja).
- It is important to test zone transfer on all available name servers, because sometimes only one is misconfigured and allows AXFR.
- Differences in the responses between servers may leak internal or staging records.

The MX records point to Google mail servers, showing that the domain uses Gmail / Google Workspace.
- This implies that incoming mail is already filtered by Google (spam/virus scanning), which affects how effective email‑based attacks (SE, phishing) might be.

The LOC record for dr.zonetransfer.me gives geographic coordinates (latitude/longitude), which can be converted to a map location.
- This exposes a physical location of a disaster‑recovery site, useful for social‑engineering or physical‑recon planning.

Several TXT records leak sensitive information:
- A phone number and email (Pippa / pippa@zonetransfer.me) tied to DNS changes help in targeted social‑engineering.
- A Google‑site‑verification token hints at Google Workspace usage.
- A TXT record used by GoDaddy for SSL‑certificate validation can reveal which certificate authorities or services the company uses.

CNAME records show that:
- testing.zonetransfer.me points to the main web host (www.zonetransfer.me), often a less‑secured test environment.
- staging.zonetransfer.me points elsewhere, indicating another staging server to investigate.

PTR and SRV records reveal further attack surface:
- A PTR record maps an IP back to a domain name, useful when mapping infrastructure from reverse DNS.
- An SRV record for _sip._tcp.zonetransfer.me exposes a SIP‑based VoIP service listening on port 5060, which can be a target for VoIP‑related attacks.

A records define the main infrastructure:
- www and the root domain point to the main web server.
- vpn and owa point to a VPN and an Outlook Web Access (OWA) server, often high‑value targets that may bypass perimeter defenses.
- Office locations (office, canberra_office, dc_office) give both network targets and geographic context, helping to time attacks (for example, after‑hours in the target region).


### how to prevent this ? ###

- Restrict transfers by IP (with ACL) : The most basic defense is to allow transfers only from trusted secondary DNS servers
- Configure TSIG keys so that only servers with the correct shared secret can perform a zone transfer. That way, even if an attacker knows your DNS IP, they cannot get the full zone without the key.
- Disable unnecessary zone transfers : if you don’t use secondary DNS servers, disable zone transfers entirely.
