# DevOps Task #13

## P8-L05 — Inspecting HTTPS, TLS, and Certificates

**Student:** Hamad Tarawa  
**Stage:** Part 8 — Network Security  
**Estimated time:** 25–35 minutes  
**Level:** Beginner  
**Focus:** DevOps Engineering  
**Environment:** WSL / Local Computer

---

## 1. Connection to the Previous Lesson

In **P8-L04 — Certificates and Trust**, you learned that a client checks:

```text
Correct domain?
      |
      v
Valid dates?
      |
      v
Trusted certificate chain?
      |
      v
Server proves control of matching private key?
      |
      v
Accept or reject
```

You also created a local self-signed certificate and inspected its metadata.

This lesson moves from a local certificate to a real public HTTPS server.

You will use:

- `curl` to inspect HTTPS behavior.
- `openssl s_client` to inspect the TLS connection and certificate chain.
- `openssl x509` to extract readable certificate information.

---

## 2. Lesson Goal

By the end of this lesson, you should be able to:

- Confirm that an HTTPS server responds.
- Observe the negotiated TLS version.
- Identify the certificate subject.
- Identify the certificate issuer.
- Identify the certificate validity dates.
- Identify the certificate domain names.
- Inspect the certificate chain at a high level.
- Recognize a successful certificate verification result.
- Save useful, non-sensitive terminal evidence for the report.

---

## 3. Why This Matters for a DevOps Engineer

When HTTPS fails, a DevOps engineer must determine where the problem exists.

Possible causes include:

```text
DNS points to the wrong server
Port 443 is blocked
The TLS service is not listening
The certificate expired
The certificate is for the wrong domain
An intermediate certificate is missing
The client does not trust the issuer
The reverse proxy has incorrect certificate paths
```

Different tools answer different questions.

| Tool | Main question |
|---|---|
| `curl -I` | Does the HTTPS endpoint return an HTTP response? |
| `curl -v` | What happened during connection, TLS, and HTTP exchange? |
| `openssl s_client` | What TLS and certificate information did the server present? |
| `openssl x509` | What readable metadata exists inside one certificate? |

This is an evidence-first troubleshooting approach.

---

## 4. Simple Explanation

### Inspection has layers

An HTTPS connection contains several layers:

```text
DNS
  |
  v
TCP connection to port 443
  |
  v
TLS handshake
  |
  v
Certificate verification
  |
  v
HTTP request and response
```

A successful HTTP response normally means the earlier required layers also
succeeded for that client.

However, inspecting each layer separately helps explain a failure.

### What are we trying to prove?

For `https://example.com`, the practical questions are:

1. Can the client reach port `443`?
2. Can the client negotiate TLS?
3. Which TLS version was selected?
4. Which certificate did the server present?
5. Does the certificate cover `example.com`?
6. Who issued it?
7. Is it within its validity period?
8. Can the client verify the chain?
9. Does the server return an HTTP response?

---

## 5. Important Terms

| Term | Simple meaning |
|---|---|
| HTTPS endpoint | Server address accepting HTTP through TLS |
| Negotiated version | TLS version selected by client and server |
| Subject | Identity described by the certificate |
| Issuer | CA that signed the certificate |
| Validity period | Time between `notBefore` and `notAfter` |
| SAN | Domain names covered by the certificate |
| Certificate chain | Server certificate connected through CAs to a trusted root |
| Leaf certificate | Server certificate at the beginning of the chain |
| SNI | Domain name sent during TLS setup so the server selects the correct certificate |
| Verification | Client checking certificate identity and trust |
| PEM | Common text format used to represent certificates |
| Exit result | Whether a command succeeded or failed |

---

## 6. Simple Real-World Example

Suppose users report:

> `https://student.example.com` does not open.

A DevOps engineer can inspect the problem in order:

```text
1. Resolve DNS
        |
        v
2. Connect to port 443
        |
        v
3. Inspect TLS negotiation
        |
        v
4. Inspect certificate and domain
        |
        v
5. Inspect HTTP response
```

Possible evidence:

```text
DNS works
Port 443 connects
TLS negotiates
Certificate covers the domain
Certificate chain verifies
HTTP returns 502
```

That result suggests TLS is working, but the reverse proxy may be unable to
reach the backend.

Another result:

```text
Port 443 connects
Certificate is expired
Certificate verification fails
```

That points to certificate renewal or deployment rather than Spring Boot
business logic.

---

## 7. Command 1 — Inspect HTTPS Headers

Run:

```bash
curl -I https://example.com
```

### Purpose

Send an HTTPS request and display response headers.

### Important option

```text
-I
```

requests headers rather than the normal response body.

### What to identify

Look for:

- HTTP version
- Status code
- `content-type`
- Redirect information if present

Possible output:

```text
HTTP/2 200
content-type: text/html
```

### What this proves

For this `curl` client:

- DNS resolution succeeded.
- A network connection was established.
- TLS negotiation succeeded.
- Certificate verification was accepted.
- An HTTP response was returned.

It does not reveal every internal component behind the public endpoint.

---

## 8. Command 2 — Inspect Verbose HTTPS Details

Run:

```bash
curl -v https://example.com
```

### Purpose

Display detailed connection, TLS, request, and response information.

### How to read the symbols

Verbose output commonly uses:

```text
*  Connection or TLS information
>  HTTP data sent by curl
<  HTTP data received from the server
```

### What to identify

Look for:

- Resolved IP address
- Connection to port `443`
- TLS version
- Certificate subject
- Certificate issuer
- Validity dates
- Certificate verification result
- HTTP request
- HTTP status

The exact wording depends on:

- `curl` version
- TLS library
- Operating system
- Server configuration

### Security warning

Do not use verbose output containing real:

- Authorization headers
- Session cookies
- API keys
- Passwords
- Private URLs with sensitive query data

Verbose commands can reveal request information in screenshots.

---

## 9. Command 3 — Connect with OpenSSL

Run:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  </dev/null
```

### Purpose

Create a TLS client connection and display detailed TLS and certificate
information.

### Important options

```text
-connect example.com:443
```

Connects to `example.com` on port `443`.

```text
-servername example.com
```

Sends the domain using Server Name Indication (`SNI`).

```text
</dev/null
```

Closes input so `openssl s_client` does not remain waiting for you to type
additional data.

### Why SNI matters

One server IP address may host several HTTPS domains.

```text
One IP address
    |
    +-- example.com
    +-- api.example.com
    +-- shop.example.com
```

SNI tells the server which domain the client wants.

Without the correct `-servername`, the server may present a default certificate
for another domain.

---

## 10. Reading `openssl s_client` Output

The output can be long. Focus on a few important areas.

### Connected

A connection message means the TCP/TLS endpoint was reached.

It does not by itself prove that certificate verification succeeded.

### Certificate chain

You may see a section similar to:

```text
Certificate chain
 0 s:CN = example.com
   i:CN = Example Intermediate CA
 1 s:CN = Example Intermediate CA
   i:CN = Example Root CA
```

Simple interpretation:

```text
0 -> Server or leaf certificate
1 -> Intermediate certificate
```

The real issuer names and chain length may differ.

### Server certificate

The output includes the server certificate in a text format.

You do not need to read the encoded certificate manually.

### TLS version

Look for output identifying the negotiated protocol, such as:

```text
Protocol  : TLSv1.3
```

or similar wording.

The selected version depends on both client and server capabilities.

### Cipher

You may see a cipher name.

The cipher describes cryptographic methods selected for the connection.

You do not need to memorize cipher names in this beginner lesson.

### Verification result

Look near the end for:

```text
Verify return code: 0 (ok)
```

This indicates that OpenSSL successfully verified the chain using its
available trust configuration for that connection.

A non-zero code indicates a verification problem that requires
investigation.

---

## 11. Command 4 — Show the Certificate Chain

Run:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -showcerts \
  </dev/null
```

### Purpose

Display the certificates sent by the server.

The output may contain several blocks:

```text
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

The server normally sends:

- Its leaf certificate.
- Required intermediate certificates.

The trusted root is commonly already stored by the client and may not be sent
by the server.

### Important safety note

Public certificates are not private keys.

The command displays certificates presented publicly during TLS.

It should never display the server's private key because the private key must
remain on the server.

---

## 12. Command 5 — Extract Readable Certificate Metadata

Run:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  </dev/null 2>/dev/null \
  | openssl x509 \
      -noout \
      -subject \
      -issuer \
      -dates \
      -ext subjectAltName
```

### Purpose

Take the leaf certificate received from the server and display only important
metadata.

### Pipeline explanation

The first command:

```bash
openssl s_client ...
```

connects to the server and outputs certificate information.

The pipe:

```text
|
```

sends that output to the next command.

The second command:

```bash
openssl x509 ...
```

reads the first certificate and prints selected fields.

### Selected fields

| Option | Displays |
|---|---|
| `-subject` | Certificate subject |
| `-issuer` | Certificate issuer |
| `-dates` | Start and expiration dates |
| `-ext subjectAltName` | Covered domain names |
| `-noout` | Hides the encoded certificate body |

### Why errors are hidden

```text
2>/dev/null
```

hides diagnostic messages from the first command so the certificate can be
passed cleanly to `openssl x509`.

For troubleshooting, run the full `s_client` command without hiding errors.

---

## 13. Command 6 — Brief TLS Summary

If your OpenSSL version supports it, run:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -brief \
  </dev/null
```

### Purpose

Display a shorter TLS connection summary.

It may show:

- Negotiated TLS version
- Selected cipher
- Peer certificate subject
- Verification status

If `-brief` is not supported in your environment, use the normal `s_client`
command.

---

## 14. How to Interpret Validity Dates

The certificate may display:

```text
notBefore=...
notAfter=...
```

### `notBefore`

The certificate should not be accepted before this time.

### `notAfter`

The certificate expires after this time.

The certificate is within its validity period when:

```text
notBefore <= current time <= notAfter
```

Certificate times are commonly displayed in GMT or UTC.

### DevOps responsibility

A DevOps engineer should:

- Monitor the expiration date.
- Renew before expiration.
- Verify that renewal installed the new certificate.
- Verify that the service reloaded successfully.
- Test from an external client.

---

## 15. How to Interpret Subject and SAN

The subject may show a common name:

```text
subject=CN = example.com
```

The Subject Alternative Name extension may show:

```text
DNS:example.com
DNS:www.example.com
```

For domain validation, modern clients use the SAN entries.

The requested domain should appear in the certificate's permitted names.

Example:

```text
Requested: example.com
SAN:       example.com, www.example.com
Result:    Name covered
```

Do not assume that a visually similar name is correct.

```text
example.com
example.net
```

are different domains.

---

## 16. Safe Practical Lab

### Lab goal

Inspect a real public HTTPS connection and record:

- HTTP status
- TLS version
- Certificate subject
- Certificate issuer
- Validity dates
- SAN domain names
- Verification result

### Safety

This lab:

- Uses a public demonstration domain.
- Sends no credentials.
- Changes no local configuration.
- Changes no remote configuration.
- Displays public certificate information only.

---

### Step 1 — Confirm the tools

Run:

```bash
curl --version
openssl version
```

Record the installed tool versions if useful.

---

### Step 2 — Confirm HTTPS response

Run:

```bash
curl -I https://example.com
```

Record the status code.

---

### Step 3 — Inspect verbose HTTPS details

Run:

```bash
curl -v https://example.com
```

Identify:

```text
Connected port:
TLS version:
Certificate verification:
HTTP status:
```

Do not worry if the exact wording differs from the lesson example.

---

### Step 4 — Inspect the OpenSSL connection

Run:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  </dev/null
```

Find:

- Certificate-chain section
- TLS version or protocol
- Cipher
- Verification return code

---

### Step 5 — Extract certificate metadata

Run:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  </dev/null 2>/dev/null \
  | openssl x509 \
      -noout \
      -subject \
      -issuer \
      -dates \
      -ext subjectAltName
```

Record:

```text
Subject:
Issuer:
Not Before:
Not After:
SAN:
```

---

### Step 6 — Complete the inspection worksheet

```text
Target domain: example.com
HTTPS port: 443
HTTP status: ______________________________
Negotiated TLS version: ___________________
Certificate subject: ______________________
Certificate issuer: _______________________
Valid from: _______________________________
Valid until: ______________________________
Domain appears in SAN: Yes / No
Verify return code: _______________________
```

---

### Step 7 — Write the result

Model answer:

> The client connected to `example.com` on port 443 and negotiated a TLS
> connection. The server presented a certificate covering the requested
> domain. The certificate showed an issuer and validity period, and OpenSSL
> reported the chain-verification result. After TLS succeeded, the server
> returned an HTTP response.

Replace the general statements with your actual results.

---

## 17. Expected Result

Your result should provide evidence similar to:

```text
HTTPS endpoint responded.
TLS version was negotiated.
Certificate subject was displayed.
Certificate issuer was displayed.
Validity dates were displayed.
SAN contained the target domain.
Certificate verification succeeded.
```

The actual:

- IP address
- Issuer
- Certificate dates
- Chain length
- TLS version
- Cipher
- HTTP version
- Headers

may change over time.

Do not copy the example values into your report. Use the values displayed by
your own commands.

---

## 18. Evidence to Save

### Screenshot 1 — HTTPS and TLS

Save output from:

```bash
curl -I https://example.com
```

and selected non-sensitive output from:

```bash
curl -v https://example.com
```

Suggested filename:

```text
01-https-tls-inspection.png
```

### Screenshot 2 — Certificate metadata

Save output from:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  </dev/null 2>/dev/null \
  | openssl x509 \
      -noout \
      -subject \
      -issuer \
      -dates \
      -ext subjectAltName
```

Suggested filename:

```text
02-certificate-metadata.png
```

### Screenshot 3 — Verification result

Save the part of normal `openssl s_client` output showing:

```text
Verify return code: 0 (ok)
```

Suggested filename:

```text
03-certificate-verification.png
```

### Suggested evidence explanation

> I used `curl` to confirm that the HTTPS endpoint returned an HTTP response
> through a TLS connection. I then used `openssl s_client` and `openssl x509`
> to inspect the negotiated TLS version, certificate subject, issuer,
> validity dates, Subject Alternative Names, certificate chain, and
> verification result. The target domain was covered by the certificate, and
> the certificate was within its displayed validity period.

Use the final sentence only if your actual output supports it.

---

## 19. Common Mistakes

### Mistake 1 — Omitting `-servername`

Without SNI, a multi-domain server may return a default certificate for the
wrong hostname.

Use:

```bash
-servername example.com
```

---

### Mistake 2 — Treating `CONNECTED` as successful verification

A network connection can succeed while certificate validation fails.

Check the verification result and certificate fields.

---

### Mistake 3 — Checking only the Common Name

Modern domain validation uses the Subject Alternative Name extension.

Inspect the SAN.

---

### Mistake 4 — Confusing issuer with subject

The subject identifies the certificate's covered identity.

The issuer identifies who signed it.

---

### Mistake 5 — Copying old certificate dates into the report

Certificates are renewed.

Always record the values from your current command output.

---

### Mistake 6 — Thinking the server sends its private key

The server sends public certificates.

Its private key must remain secret on the server.

---

### Mistake 7 — Hiding errors during troubleshooting

The short pipeline hides `s_client` diagnostic output.

When diagnosing a failure, run the full command without:

```text
2>/dev/null
```

---

### Mistake 8 — Including sensitive request headers

Do not save verbose evidence containing real tokens, cookies, or passwords.

---

## 20. Task Report Notes

You may adapt the following text for your final assignment:

> HTTPS and TLS can be inspected using command-line tools. The `curl -I`
> command confirms that an HTTPS endpoint returns an HTTP response, while
> `curl -v` displays connection, TLS, certificate, request, and response
> details.

> The `openssl s_client` command connects directly to a TLS endpoint and
> displays the negotiated protocol, cipher, presented certificate chain, and
> verification result. The `-servername` option sends the intended domain
> using SNI so that a multi-domain server can present the correct certificate.

> Certificate metadata includes the subject, issuer, validity dates, and
> Subject Alternative Names. The requested domain should be covered by a SAN,
> the current time should fall within the validity period, and the certificate
> chain should verify through a trusted root.

### Practical demonstration note

> I inspected `https://example.com` using `curl` and OpenSSL. I confirmed the
> HTTPS response, observed the negotiated TLS connection, and extracted the
> certificate subject, issuer, dates, and SAN. I also inspected the OpenSSL
> verification result. The recorded screenshots contain only public
> certificate information and no credentials.

---

## 21. Five Review Questions

1. What is the difference between `curl -I` and `curl -v`?
2. Why should `openssl s_client` include the `-servername` option?
3. Which certificate field lists the domain names covered by the certificate?
4. Does a successful TCP connection prove that the certificate is trusted?
5. Which values should a DevOps engineer record when investigating certificate
   expiration?

---

## 22. Lesson Summary

HTTPS inspection follows the connection layers:

```text
DNS
  |
  v
TCP port 443
  |
  v
TLS negotiation
  |
  v
Certificate and chain verification
  |
  v
HTTP response
```

Use:

```bash
curl -I https://example.com
```

to inspect the HTTPS response.

Use:

```bash
curl -v https://example.com
```

to inspect detailed connection and TLS information.

Use:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  </dev/null
```

to inspect TLS, the chain, and verification.

Use:

```bash
openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

to display readable certificate metadata.

---

## 23. Progress

### Stage 1 — Part 7: Linux Security

- [x] P7-L01 through P7-L10 completed

### Stage 2 — Part 8: Network Security

- [x] P8-L01 — The Path of a Web Request
- [x] P8-L02 — HTTP and HTTPS
- [x] P8-L03 — TLS Explained Simply
- [x] P8-L04 — Certificates and Trust
- [x] P8-L05 — Inspecting HTTPS, TLS, and Certificates
- [ ] P8-L06 — DNS Security
- [ ] P8-L07 — Firewalls and Reverse Proxy Security
- [ ] P8-L08 — Common Network Attacks at a High Level
- [ ] P8-L09 — Part 8 Network Security Report

---

## Exact Next Lesson

**P8-L06 — DNS Security**

Do not continue until the student writes:

```text
next
```
