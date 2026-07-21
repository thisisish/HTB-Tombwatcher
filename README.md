# TombWatcher — HackTheBox Writeup

**Difficulty:** Medium

**OS:** Windows (Active Directory)

**Category:** Active Directory / ADCS

---

## Summary

TombWatcher is an Active Directory box chaining together classic ACL abuse
primitives with a lesser-known technique: reanimating a **deleted AD
object** to recover leftover privileges, then abusing an ADCS certificate
template via **ESC3 (Enrollment Agent)** to obtain a certificate for the
domain Administrator and gain full domain compromise.

The interesting part isn't any single technique — it's the chain: a
sequence of ACL rights (`GenericAll`, `ForceChangePassword`, `WriteOwner`)
handed off between low-privileged accounts, ending in the discovery that a
long-deleted service account (`cert_admin`) still held write access to a
certificate template, and could be restored from AD's Deleted Objects
container to reclaim that access.

---

## Attack path

​```mermaid
flowchart TD
    A["Henry\nInitial access"] -->|WriteSPN / Kerberoast| B["Alfred"]
    B -->|AddSelf| C["Infrastructure\ngroup"]
    C -->|ReadGMSAPassword| D["Ansible_dev5\ngMSA"]
    D -->|ForceChangePassword| E["Sam"]
    E -->|GenericAll| F["John"]
    F -->|WriteOwner + DACL write| G["cert_admin\nreanimated deleted object"]
    G -->|Enrollment Agent cert\nESC3| H["Administrator\nvia certipy on-behalf-of"]
    H -->|PKINIT + NT hash| I["DC01\nAdministrator shell"]
​```