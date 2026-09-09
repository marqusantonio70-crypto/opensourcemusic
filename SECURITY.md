# Security policy

Openitify runs entirely client-side by default. If you connect a Supabase backend:

- Only ever ship the **publishable/anon** key. Never commit a service-role key.
- All privileged actions must stay behind row-level security and `SECURITY DEFINER` RPCs; the in-app role gate is cosmetic.
- Rotate the demo owner credential immediately after claiming the owner account.
- OAuth requires an http(s) origin listed in your Supabase redirect URLs.

Found a vulnerability? Open a private security advisory on this repository rather than a public issue.
