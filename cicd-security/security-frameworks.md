1. What is OWASP TOP 10?
   - OWASP stands for Open Worldwide Application Security Project.
   - The OWASP Top 10 is a list of the most critical/common web application security risks.
   - The current OWASP Top 10
```yaml
| #  | Risk                                     | Simple meaning                                                 |
| -- | ---------------------------------------- | -------------------------------------------------------------- |
| 1  | Broken Access Control                    | User can access something they shouldn't                       |
| 2  | Cryptographic Failures                   | Sensitive data isn't properly protected                        |
| 3  | Injection                                | Attacker sends malicious input that gets executed              |
| 4  | Insecure Design                          | Security wasn't considered during system design                |
| 5  | Security Misconfiguration                | Incorrect/insecure configuration                               |
| 6  | Vulnerable and Outdated Components       | Using vulnerable libraries/images                              |
| 7  | Identification & Authentication Failures | Weak authentication/session management                         |
| 8  | Software & Data Integrity Failures       | Untrusted code/dependencies or compromised updates             |
| 9  | Security Logging & Monitoring Failures   | Attacks happen but aren't detected                             |
| 10 | SSRF                                     | Server is tricked into making requests to unintended locations |

```
<br><br>

2. What is NIST?
   NIST (National Institute of Standards and Technology) is a U.S. organization that provides cybersecurity standards, guidelines, and frameworks to help organizations identify, protect, detect, respond to, and recover from security risks.

<br><br>


3. What is CIS(Center for Internet Security (CIS)) benchmark?
 In AWS: AWS Security Hub
   For example, admins can follow the step-by-step CIS AWS Foundations Benchmark guidelines to help them set up a strong password policy for AWS Identity and Access Management (IAM). Password policy enforcement, multi-factor authentication (MFA) usage, disabling root, ensuring access keys are rotated every 90 days, and other tactics are distinct, but related, identity guidelines to improve the security of an AWS account.

In Azure: Defender for cloud, Azure policy
<br><br>

4. What is Zero Trust Architecture?
   - While traditional security models assume everything in an organization’s network is trustworthy, Zero Trust security architecture authenticates every user and device before they can access resources—whether they’re located within or outside the corporate network.
   - Zero Trust architecture (ZTA) is a security framework that authenticates every access request and proactively anticipates cyberattacks.
   - Businesses adopt this framework to ensure only authorized users and devices can enter their networks, access business resources, and view sensitive data.
   - It operates using end-to-end encryption, robust access control mechanisms, AI, and network monitoring capabilities.
   - ZTA enables businesses to support remote work, minimize risk, ease regulatory compliance, save time, and strengthen security postures.
   - Zero Trust solutions include multifactor authentication (MFA) and identity and access management systems.
   - Zero Trust is built on the principle of “never trust, always verify”, meaning no user, device, or system is inherently trusted—trust must be established and continuously validated for every access request.
   - Verify explicitly
Zero Trust handles every attempt to access business resources as if the request originated from an open network. Rather than verifying credentials once at the point of entry, ZTA regularly and comprehensively evaluates data points—such as the user’s identity, location, and device—in real time to identify red flags and help ensure only authorized users and devices can access your network.

   - Use least privileged access
ZTA provides each user with only the minimum level of access needed to perform their tasks. Limiting access rights in this way helps your business minimize the damage that a compromised account can cause.

   - Assume breach
Zero Trust operates under the premise that breaches are inevitable. Instead of solely focusing on preventing them, this approach also proactively anticipates cyberattacks by assuming users, devices, and systems across your business are already compromised.




