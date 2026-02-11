## Passive (=OSINT)

- [[OSINT]]

---
---
## Active

- DNS Enumeration: [[53 - DNS|53 - DNS]]

- PortScan：[[Port Scan & Vuln Scan]]

- SMB Enumeration: [[139,445 -NetBIOS, SMB]]

- SMTP Enumeration: [[25,465,587 - SMTP]]

- SNMP Enumeration: [[161,162 - SNMP]]

- Exiftool: [[Module 12：Client-side Attacks#Exiftool]]

###### 🤖LLM

- サブドメイン列挙のwordlist作成プロンプト
```txt
Using public data from [domain]'s website and any information that can be inferred about its organizational structure, products, or services, generate a comprehensive list of potential subdomain names.
	•	Incorporate common patterns used for subdomains, such as:
	•	Infrastructure-related terms (e.g., "api", "dev", "test", "staging").
	•	Service-specific terms (e.g., "mail", "auth", "cdn", "status").
	•	Departmental or functional terms (e.g., "hr", "sales", "support").
	•	Regional or country-specific terms (e.g., "us", "eu", "asia").
	•	Factor in industry norms and frequently used terms relevant to [domain]'s sector.

Finally, compile the generated terms into a structured wordlist of 1000  words, optimized for subdomain brute-forcing against [domain]

Ensure the output is in a clean, lowercase format with no duplicates, no bulletpoints and ready to be copied and pasted.
Make sure the list contains 1000 unique entries.
```


