# A02:2021 – Cryptographic Failures    ![icon](assets/TOP_10_Icons_Final_Crypto_Failures.png){: style="height:80px;width:80px" align="right"}

## Factors

| CWEs Mapped | Max Incidence Rate | Avg Incidence Rate | Avg Weighted Exploit | Avg Weighted Impact | Max Coverage | Avg Coverage | Total Occurrences | Total CVEs |
|:-------------:|:--------------------:|:--------------------:|:--------------:|:--------------:|:----------------------:|:---------------------:|:-------------------:|:------------:|
| 29          | 46.44%             | 4.49%              |7.29                 | 6.81                |  79.33%       | 34.85%       | 233,788           | 3,075      |

## 概述

上移一个位置到#2，以前称为敏感数据暴露，这更像是一个广泛的症状而不是根本原因，重点是与密码学相关的失败（或缺乏密码学）。这往往会导致敏感数据的暴露。值得注意的CWE包括CWE-259：使用硬编码密码、CWE-327：损坏或风险的加密算法和CWE-331 熵不足。

## 描述 

首先是确定传输中和静止数据的保护需求。例如，密码、信用卡号、健康记录、个人信息和商业秘密需要额外保护，主要是如果该数据属于隐私法（例如欧盟的通用数据保护条例 (GDPR)）或法规（例如金融数据保护）例如 PCI 数据安全标准 (PCI DSS)。对于所有此类数据：

-   是否有任何数据以明文形式传输？这涉及 HTTP、SMTP 和 FTP 等协议。外部互联网流量是危险的。验证所有内部流量，例如，负载平衡器、Web 服务器或后端系统之间的流量。

-   默认情况下或在较旧的代码中是否使用任何旧的或弱的加密算法？

-   是否正在使用默认加密密钥、生成或重复使用弱加密密钥，或者是否缺少适当的密钥管理或轮换？

-   Are default crypto keys in use, weak crypto keys generated or
    re-used, or is proper key management or rotation missing?
    Are crypto keys checked into source code repositories?

-   是否未强制执行加密，例如，是否缺少任何用户代理（浏览器）安全指令或标头？

-   用户代理（例如，应用程序、邮件客户端）是否不验证收到的服务器证书是否有效？

-   Are initialization vectors ignored, reused, or not generated
    sufficiently secure for the cryptographic mode of operation?
    Is an insecure mode of operation such as ECB in use? Is encryption
    used when authenticated encryption is more appropriate?

-   Are passwords being used as cryptographic keys in absence of a
    password base key derivation function?

-   Is randomness used for cryptographic purposes that was not designed
    to meet cryptographic requirements? Even if the correct function is
    chosen, does it need to be seeded by the developer, and if not, has
    the developer over-written the strong seeding functionality built into
    it with a seed that lacks sufficient entropy/unpredictability?

-   Are deprecated hash functions such as MD5 or SHA1 in use, or are
    non-cryptographic hash functions used when cryptographic hash functions
    are needed?

-   Are deprecated cryptographic padding methods such as PKCS number 1 v1.5
    in use?

-   Are cryptographic error messages or side channel information
    exploitable, for example in the form of padding oracle attacks?

参考 ASVS 加密（V7），资料保护（V9），和SSL/TLS(V10)。

## 如何预防

至少执行以下操作，并查阅参考资料：

-   对应用程序处理、存储或传输的数据进行分类。根据隐私法、监管要求或业务需求确定哪些数据是敏感的。

-   不要不必要地存储敏感数据。尽快丢弃它或使用符合 PCI DSS 的标记化甚至截断。未保留的数据不能被窃取。

-   确保加密所有静态敏感数据。

-   确保拥有最新且强大的标准算法、协议和密钥；使用适当的密钥管理。

-   使用安全协议（例如具有完美前向保密 (PFS) 密码的 TLS、服务器的密码优先级和安全参数）加密所有传输中的数据。使用 HTTP 严格传输安全 (HSTS) 等指令强制加密。

-   对包含敏感数据的响应禁用缓存。

-   Apply required security controls as per the data classification.

-   Do not use legacy protocols such as FTP and SMTP for transporting
    sensitive data.

-   使用具有工作因子（延迟因子）的强自适应和加盐散列函数存储密码，例如 Argon2、scrypt、bcrypt 或 PBKDF2。

-   Initialization vectors must be chosen appropriate for the mode of
    operation.  For many modes, this means using a CSPRNG (cryptographically
    secure pseudo random number generator).  For modes that require a
    nonce, then the initialization vector (IV) does not need a CSPRNG.  In all cases, the IV
    should never be used twice for a fixed key.

-   Always use authenticated encryption instead of just encryption.

-   Keys should be generated cryptographically randomly and stored in
    memory as byte arrays. If a password is used, then it must be converted
    to a key via an appropriate password base key derivation function.

-   Ensure that cryptographic randomness is used where appropriate, and
    that it has not been seeded in a predictable way or with low entropy.
    Most modern APIs do not require the developer to seed the CSPRNG to
    get security.

-   Avoid deprecated cryptographic functions and padding schemes, such as
    MD5, SHA1, PKCS number 1 v1.5 .

-   独立验证配置和设置的有效性。

## 攻击场景示例

**场景 #1**: 应用程序使用自动数据库加密对数据库中的信用卡号进行加密。但是，此数据在检索时会自动解密，从而允许 SQL 注入缺陷以明文形式检索信用卡号。

**场景 #2**: A site doesn't use or enforce TLS for all pages or
supports weak encryption. An attacker monitors network traffic (e.g., at
an insecure wireless network), downgrades connections from HTTPS to
HTTP, intercepts requests, and steals the user's session cookie. The
attacker then replays this cookie and hijacks the user's (authenticated)
session, accessing or modifying the user's private data. Instead of the
above they could alter all transported data, e.g., the recipient of a
money transfer.

**场景 #3**: The password database uses unsalted or simple hashes to
store everyone's passwords. A file upload flaw allows an attacker to
retrieve the password database. All the unsalted hashes can be exposed
with a rainbow table of pre-calculated hashes. Hashes generated by
simple or fast hash functions may be cracked by GPUs, even if they were
salted.

## References

-   [OWASP Proactive Controls: Protect Data
    Everywhere](https://owasp.org/www-project-proactive-controls/v3/en/c8-protect-data-everywhere)

-   [OWASP Application Security Verification Standard (V7,
    9, 10)](https://owasp.org/www-project-application-security-verification-standard)

-   [OWASP Cheat Sheet: Transport Layer
    Protection](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)

-   [OWASP Cheat Sheet: User Privacy
    Protection](https://cheatsheetseries.owasp.org/cheatsheets/User_Privacy_Protection_Cheat_Sheet.html)

-   [OWASP Cheat Sheet: Password and Cryptographic Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

-   [OWASP Cheat Sheet:
    HSTS](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)

-   [OWASP Testing Guide: Testing for weak cryptography](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/README)


## List of Mapped CWEs

[CWE-261 Weak Encoding for Password](https://cwe.mitre.org/data/definitions/261.html)

[CWE-296 Improper Following of a Certificate's Chain of Trust](https://cwe.mitre.org/data/definitions/296.html)

[CWE-310 Cryptographic Issues](https://cwe.mitre.org/data/definitions/310.html)

[CWE-319 Cleartext Transmission of Sensitive Information](https://cwe.mitre.org/data/definitions/319.html)

[CWE-321 Use of Hard-coded Cryptographic Key](https://cwe.mitre.org/data/definitions/321.html)

[CWE-322 Key Exchange without Entity Authentication](https://cwe.mitre.org/data/definitions/322.html)

[CWE-323 Reusing a Nonce, Key Pair in Encryption](https://cwe.mitre.org/data/definitions/323.html)

[CWE-324 Use of a Key Past its Expiration Date](https://cwe.mitre.org/data/definitions/324.html)

[CWE-325 Missing Required Cryptographic Step](https://cwe.mitre.org/data/definitions/325.html)

[CWE-326 Inadequate Encryption Strength](https://cwe.mitre.org/data/definitions/326.html)

[CWE-327 Use of a Broken or Risky Cryptographic Algorithm](https://cwe.mitre.org/data/definitions/327.html)

[CWE-328 Reversible One-Way Hash](https://cwe.mitre.org/data/definitions/328.html)

[CWE-329 Not Using a Random IV with CBC Mode](https://cwe.mitre.org/data/definitions/329.html)

[CWE-330 Use of Insufficiently Random Values](https://cwe.mitre.org/data/definitions/330.html)

[CWE-331 Insufficient Entropy](https://cwe.mitre.org/data/definitions/331.html)

[CWE-335 Incorrect Usage of Seeds in Pseudo-Random Number Generator(PRNG)](https://cwe.mitre.org/data/definitions/335.html)

[CWE-336 Same Seed in Pseudo-Random Number Generator (PRNG)](https://cwe.mitre.org/data/definitions/336.html)

[CWE-337 Predictable Seed in Pseudo-Random Number Generator (PRNG)](https://cwe.mitre.org/data/definitions/337.html)

[CWE-338 Use of Cryptographically Weak Pseudo-Random Number Generator(PRNG)](https://cwe.mitre.org/data/definitions/338.html)

[CWE-340 Generation of Predictable Numbers or Identifiers](https://cwe.mitre.org/data/definitions/340.html)

[CWE-347 Improper Verification of Cryptographic Signature](https://cwe.mitre.org/data/definitions/347.html)

[CWE-523 Unprotected Transport of Credentials](https://cwe.mitre.org/data/definitions/523.html)

[CWE-720 OWASP Top Ten 2007 Category A9 - Insecure Communications](https://cwe.mitre.org/data/definitions/720.html)

[CWE-757 Selection of Less-Secure Algorithm During Negotiation('Algorithm Downgrade')](https://cwe.mitre.org/data/definitions/757.html)

[CWE-759 Use of a One-Way Hash without a Salt](https://cwe.mitre.org/data/definitions/759.html)

[CWE-760 Use of a One-Way Hash with a Predictable Salt](https://cwe.mitre.org/data/definitions/760.html)

[CWE-780 Use of RSA Algorithm without OAEP](https://cwe.mitre.org/data/definitions/780.html)

[CWE-818 Insufficient Transport Layer Protection](https://cwe.mitre.org/data/definitions/818.html)

[CWE-916 Use of Password Hash With Insufficient Computational Effort](https://cwe.mitre.org/data/definitions/916.html)
