# Questions

### Links:
-> [MS Mitigates](https://www.microsoft.com/en-us/msrc/blog/2023/07/microsoft-mitigates-china-based-threat-actor-storm-0558-targeting-of-customer-email)
-> [MS On The Issues](https://blogs.microsoft.com/on-the-issues/2023/07/11/mitigation-china-based-threat-actor/)
-> [Results of Major Investigations](https://www.microsoft.com/en-us/msrc/blog/2023/09/results-of-major-technical-investigations-for-storm-0558-key-acquisition)

## How did they separate access & infrastructure according to data relevance & impact?

To accomplish this, Microsoft made a distinction between two environments. On one hand, they isolated the very high impact (and data relevance) production environment, closing off collaboration tools such as email, web browsers and conference tools from external network traffic; they also took security measures like providing dedicated accounts for admin work, Just In Time + Just Enough Access models and, importantly, the policy to never allow secret keys (or whatever cryptographic material) to leave the premise.

On the other hand, the corporate environment would be provisioned with the standard collaboration tools to ensure a natural workflow; the key to securing this scene is to apply the zero trust model, meaning the workstations must be assummed as compromised because of their vulnerable nature. It was withing this high surface of impact zone that a debugging environment sat in and captured the private signing key from a crash dump due to a race condition.

Having addressed access distinctions, the separation of infrastructure is straightforward, but with a crucial flaw. Signing keys architecturally intended to separate two concerns: the consumer (MSA keys) and the enterprise (Azure AD keys). The assumption that consumer-signed tokens wouldn't be accepted in enterprise mailboxes was breached by the fact that, due to cryptographic helper libraries verified the digital signature of the private keys against public keys without validating the scope of the key, fundamentally breaking the contract.

## How do roles and personnel fit into this, and which role could policies and training play?

...
