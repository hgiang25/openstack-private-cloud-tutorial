# Security Policy

## Supported Versions

This repository is maintained as a personal infrastructure portfolio project.

The latest version on the `main` branch is considered the supported version.

| Version         | Supported |
| --------------- | --------- |
| Latest (main)   | ✅        |
| Older revisions | ❌        |

---

# Reporting a Security Issue

If you discover a security issue related to the documentation, deployment process, or published configuration examples, please report it responsibly.

You may open a GitHub Issue or contact the repository owner directly.

Please include:

- Description of the issue
- Steps to reproduce
- Expected behavior
- Actual behavior
- Possible impact
- Suggested mitigation (if available)

---

# Sensitive Information

This repository intentionally excludes any confidential infrastructure information.

The following items should never be committed:

- Passwords
- Private SSH keys
- API tokens
- TLS private keys
- Cloud credentials
- VMware licenses
- Internal IP addresses from production environments
- Customer information

Example:

❌ admin_password = MyPassword123

✅ admin_password = <REDACTED>

---

# Configuration Files

All configuration files included in this repository are intended for educational and demonstration purposes.

Before using them in production environments, ensure that you:

- Review authentication settings
- Replace default credentials
- Enable TLS encryption
- Apply least privilege principles
- Validate network segmentation

---

# Responsible Disclosure

If a security issue is identified, please avoid publishing exploit details publicly until the issue has been reviewed.

Responsible disclosure helps maintain the integrity of educational resources while minimizing unnecessary risk.

---

# Best Security Practices

When deploying OpenStack in production, consider implementing:

- TLS for all API endpoints
- Role-Based Access Control (RBAC)
- Multi-Factor Authentication (MFA)
- Secret management (e.g., HashiCorp Vault)
- Database encryption
- Network segmentation
- Security Groups
- Firewall hardening
- Regular security updates
- Centralized logging and auditing

---

# Disclaimer

This repository documents a laboratory deployment intended for learning and demonstration purposes.

It should not be considered a production-ready reference without additional hardening, validation, and security review.
