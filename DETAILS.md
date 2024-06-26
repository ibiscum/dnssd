# Multicast DNS

- Every DNS resource record has a TTL, which is the number of seconds for which the resource record may be cached.
- mDNS allows host names of form `<single-dns-label>.local`
- The domain "local" is a link-local Multicast DNS domain.
	> This means that names within that domain are only meaningful on the link they originate. Any DNS query for a name ending with ".local" must be sent to the mDNS IPv4 link-local mutlicast adress "224.0.0.251" (or "FF02::FB" for IPv6)
	
    > […] DNS queries for names not ending "local." may be sent to the mDNS multicast address. […]
	
    > Any DNS query for a name ending with "254.169.in-addr.arpa." MUST be sent to the mDNS IPv4 link-local multicast address 224.0.0.251 or the mDNS IPv6 multicast address FF02::FB.
	
    > Likewise, any DNS query for a name within the reverse mapping domains for IPv6 link-local addresses ("8.e.f.ip6.arpa.", "9.e.f.ip6.arpa.", "a.e.f.ip6.arpa.", and "b.e.f.ip6.arpa.") MUST be sent to the mDNS IPv6 link-local multicast address FF02::FB or the mDNS IPv4 link-local multicast address 224.0.0.251.
	
    [rfc6762][mdns]
- Probing

# DNS-SD

DNS-SD specifies how services services can be described and found using standard DNS records.

- Service instance names must not contain any ASCII control characters (0x00-0x1F and 0x7F), it can contain spaces or any other Net-Unicode (what's that?). The label is limited to 63 bytes.

- If a service instance name is rejected by the DNS server, we should retry the query using the "Punnycode" algorithm.

- The characters of a service instance name, consisting of `<Instance>`, `<Service>`, and `<Domain>`, should be escaped to enuse DNS label boundaries.
	- Dots in <Instance> should be escaped, like "." becomes "\."
	- Backslashes in <Instance> should be escaped, like "\" becomes "\\"

- If more than one SRV records are returned when searching for a particular service instance, we  must interpret the priority and weight fields of the SRV record. But it's common that those fields are set to zero.

- TXT record
	- Every DNS-SD SRV record must have a TXT record, with the same name containing key-value pairs (<key>=<value>) or a single zero byte.
		- TXT record strings starting with an "=" character or having no "=" character are ignored
		- key must be at least one character, no more than 9 characters longs, printable US_ASCII values (0x20-0x7E), cases are ignored, must be unique (only use the first)
		- Examples
			- "": key is not present
			- "myKey": key present, with no value
			- "myKey=": key present, with empty value
			- "myKey=myValue": key present, with no empty value
		- value
			- must not be enclosed with quotation mark
			- is binary data (doesn't matter if US-ASCII oder UTF-8), display as hex alongside (UTF-8)
			- 
	- When using mDNS, TXT records can be up to 8900 bytes long, because the maximum packet size is 9000 bytes. DNS-SD recommends the following TXT record sizes.
		- < 200 bytes
		- < 400 bytes, to fit into a single 512-byte DNS message
		- < 1300 bytes, to fit into a single 1500-byte Eterhnet packet
		- > 1300 is not recommended
		- (Sidenote: Hardware can offer mDNS offloading, only if TXT records are not larger than 256 bytes.)
	- If there is a need to indicate the application protocol version, use the key "protovers". It's just a recommendation though from RFC6763 though.
- Service name: <1st label>.<2nd label>
	- 1st label: Name of the service starting with an underscore "_<name>"
	- 2nd label: Transport protocol, "_tcp" for TCP based transport protocols, otherwise "_udp" for all other transport protocols (which is weird)
	- Must not be empty, and shouldn't be longer than 15 characters (without the mandatory underscore)
	- One ore more letter, and digits, and no consecutive hyphens (--)
- RFC6763 Section 9 defines a meta-query for problem diagnostics and network management (not needed yet)
- RFC6763 Section 10: A mDNS client should answer mDNS queries for its PTR, SRV and TXT names ending with "local.".
- RFC6763 Section 12: Additional records can be placed in the addition section of a DNS message. It's recommended to improve network efficiency (TODO later)
