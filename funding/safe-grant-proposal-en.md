# Safe Foundation Grant Proposal

**Project:** Certilia eIDAS Identity × Safe Treasury
**Repository:** https://github.com/javni-novci/svima-a-ne-samo-njima
**Contact:** [TO FILL — email]
**Date:** [TO FILL]

---

## 1. Project Description

**Certilia eIDAS Identity × Safe Treasury** is open-source infrastructure connecting the Croatian eIDAS digital identity system (Certilia Mobile.id) with Safe smart accounts on Gnosis Chain, using a ZK nullifier approach for GDPR-compliant Sybil resistance and EURe (Monerium) for EUR-denominated transactions.

### Problem

Blockchain solves transparency but suffers from Sybil attacks — one person can create unlimited wallets. Government identity systems (eIDAS) provide perfect legal identity but are closed silos. No bridge exists between these two worlds that respects GDPR.

### Solution

A ZK nullifier approach where the user proves "I am a unique citizen" without revealing WHO they are. A smart contract on Gnosis Chain verifies the ZK proof, mints a Soulbound Token (SBT), and a Safe treasury recognizes the SBT for fund governance.

### Proof of Concept

A podcast marketplace where creators receive EURe automatically via 0xSplits V2 PushSplit, and only verified citizens can participate. The platform uses Safe as its operational treasury.

### Long-term Vision

The same infrastructure for transparent public budget management and electronic voting — built entirely on the Safe ecosystem.

---

## 2. Contribution to the Safe Ecosystem

| Component | Contribution |
|-----------|-------------|
| **IIdentityVerifier Module** | A new Safe module connecting eIDAS identity with Safe governance. Extensible to all EU member states (eIDAS is pan-European). |
| **Zodiac Guard for SBT** | A Guard that checks for a valid Soulbound Token before approving Safe transactions. Only verified citizens can initiate payouts. |
| **0xSplits + Safe Integration** | Documented flow: EURe payment → 0xSplits V2 PushSplit → 90% to creator + 10% to platform Safe. A template for any marketplace. |
| **GDPR-Compliant Identity for Safe** | The first Safe module that respects GDPR via ZK nullifiers. A template for EU projects building on Safe. |
| **EIP-2771 Gasless Integration** | Documented flow for gasless SBT minting via OpenGSN on Gnosis Chain, compatible with Safe. |

### Open Source

All code under MIT license. Architecture decisions publicly documented (7 ADRs, 18 attacks in threat model, 7 deep research topics with cross-verification averaging 85/100 concordance).

---

## 3. Scope & Milestones

| Milestone | Timeline | Deliverables |
|-----------|----------|-------------|
| **M1: Certilia Integration** | Month 1-2 | OIDC flow with Certilia (WSO2 IS), CertiliaSBT contract on Gnosis testnet, IIdentityVerifier v1 (JWT verification via OpenZeppelin RSA v5.1+) |
| **M2: ZK Nullifier** | Month 3-4 | ZK circuit for JWT verification (Circom/Halo2), Groth16 verifier on Gnosis (~200-300K gas), IIdentityVerifier v2 (nullifier), DPIA document for GDPR compliance |
| **M3: Marketplace + Safe** | Month 5-6 | Frontend application, 0xSplits V2 PushSplit integration with EURe, Safe treasury with Zodiac Guard, EIP-2771 gasless SBT minting via OpenGSN |
| **M4: Launch** | Month 7-8 | User testing with Croatian Certilia users, security audit, production deployment on Gnosis mainnet |

---

## 4. Budget Breakdown

| Item | Estimate | Notes |
|------|----------|-------|
| Smart contract development | [TO FILL] | CertiliaSBT, IIdentityVerifier, ZK circuit |
| Backend development | [TO FILL] | Node.js OIDC relay, OpenGSN relayer |
| Frontend development | [TO FILL] | Web app with wallet integration |
| Security audit | $10,000-30,000 | Professional smart contract audit |
| DPIA & legal consultations | $2,000-5,000 | GDPR compliance, national DPA advisory |
| Infrastructure (12 months) | $2,000-5,000 | Server, RPC nodes, Paymaster deposit |
| **Total** | **[TO FILL]** | |

---

## 5. Team

[TO FILL — name, role, relevant experience, links]

---

## 6. Expected Impact

- **Short-term:** A functional podcast marketplace with eIDAS identity on Gnosis Chain. Proof that Safe can govern funds of verified EU citizens.
- **Mid-term:** A template that any EU member state can adapt for their own eIDAS identity + Safe treasury use case.
- **Long-term:** Infrastructure for transparent public budget management and electronic voting — all built on the Safe ecosystem.

### Success Metrics

| Metric | Target (12 months) |
|--------|-------------------|
| Verified users (CERTILIA SBT) | 100+ |
| Active creators | 10+ |
| EURe volume through 0xSplits | €1,000+ |
| Security incidents | 0 |
| GitHub stars | 50+ |
| Forks | 5+ |

---

## 7. Why This Matters for Safe

Safe is the most trusted smart account infrastructure in Web3. But today, Safe has no bridge to government-issued identity. This project creates that bridge:

1. **EU regulatory alignment:** eIDAS + GDPR + MiCA compliance makes Safe viable for regulated European use cases.
2. **New user segment:** 450M+ EU citizens with eIDAS identities — potential Safe users once the identity bridge exists.
3. **Reusable modules:** IIdentityVerifier and Zodiac Guard are generic — any EU project building on Safe can reuse them.
4. **First-mover:** No existing project connects eIDAS with Safe. This is greenfield.

---

## 8. Existing Work

The project already has extensive architectural documentation before any code has been written:

- 7 Architecture Decision Records (ADRs) with quantitative scoring matrices
- 18 identified attacks in a comprehensive threat model
- 7 deep research topics with dual-source cross-verification (Claude + Perplexity, average concordance 85/100)
- GDPR analysis incorporating EDPB Guidelines 02/2025 on blockchain
- Roadmap with phases 0/1a/1b/2/3 and clear dependencies
- Full Croatian localization (the target market)

All publicly available at: https://github.com/javni-novci/svima-a-ne-samo-njima
