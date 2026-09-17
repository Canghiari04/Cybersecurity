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

When looking into the roles that human agents played in this event we have to first point out the one that made the attak possible, and that's the operator in the corporate environment that likely fell victim to a phishing (or perhaps social engineering) attack which resulted in the corporate debugging repository access being leaked (and captured by Storm). Besides, we have the engineers who were woeking in the endpoint and assummed the helper cryptographic libraries to also validate scope of the issued tokens (probably some confusion between business logic and authorization as part of the system).

As for the roles, we must first address that of the policies. Firstly, policies must explicitly classify memory crash dumps and core dumps as containing sensitive memory, inspection which Microsoft states to have enforced since then; the latter should be paired with credential scanning before any file transitions as part of ingestion policies; ideally, Just In Time + Just Enough Access should be applied to all engineers that pretend to access any relevant data, though it is true that this introduces friction; finally log retention policies should prevent cases like this one, in which definitive proof that this engineer's account was the backdoor is nowhere to be found.

Additionally, as for the training role, developers will need introduced (hands-on) to OAuth lifecycles so they can distinguish verifying signature integrity apart from checking the issuer (iss) and the audience (aud). Also, regarding safe practices around data sanitization and debugging, the operators need to recognize that race conditions often can leak secrets and personnel must treat memory snapshots as secret material rather than typical logs. Plus, anti-phishing adequation and training are a must so they can identify such attack vectors (those that have usual access to critical tools would especially benefit from such training).
