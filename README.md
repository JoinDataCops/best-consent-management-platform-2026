# Consent Management Platforms 2026: A Practical README

Comparison reference for engineering and privacy teams evaluating CMPs in 2026.

## TL;DR

| Tool | Tier | Setup | Entry price | TCF 2.2 | CMv2 default | SOC 2 |
|---|---|---|---|---|---|---|
| OneTrust | Enterprise | 2+ days | $10K/yr min | Yes | Healthy | Type II |
| Ketch | Enterprise | Half day | Free up to 5K users | Yes | Healthy | Type II |
| DataGrail | Enterprise | 2 days | Custom | Yes | Healthy | Type II |
| Securiti | Enterprise | 2 days | Custom | Yes | Healthy | Type II |
| Cookiebot | Mid-market | 30 min | Free / €7/mo | Yes | Healthy | Type II |
| Didomi | Mid-market | 4 hr | Custom | Yes | Healthy | Type II |
| CookieHub | Mid-market | 45 min | Free / €6/mo | Yes (Business) | Healthy | Yes |
| Iubenda | SMB | 35 min | $6.99/mo | Yes (Pro) | Healthy | Yes |
| Termly | SMB | 25 min | $10/mo | Partial | Healthy | Yes |
| CookieYes | SMB | 18 min | Free / $10/mo | No | Partial | No |
| DataCops | Bundled | 12 min | Free / $7.99/mo | Yes | Healthy | In progress |

## Why this README exists

The CMP market consolidated fast in late 2025 and Q1 2026. Quantcast Choice shut down. CookieFirst got acquired. Sourcepoint merged into Didomi. Cookiebot doubled prices. OneTrust enforced a $10K floor and ran layoffs. Securiti got bought by Veeam for $1.7B.

This README gives an engineer-readable shortlist, install paths, and the failure modes worth wargaming.

## Key concepts to internalize

**TCF 2.2.** IAB Europe's transparency and consent framework, version 2.2. Required for programmatic ad participation in EEA. After March 1, 2026, publishers stuck on older versions default to Limited Ads (60 to 80% CPM drops).

**Consent Mode v2 (CMv2).** Google's signal architecture for honoring user consent in tracking. Two states: basic and advanced. Misconfiguration documented to cause 90% overnight drops in measured Google Ads conversions.

**First-party CMP.** Consent state stored on your own subdomain instead of vendor's domain. Survives ITP, ad blockers, and third-party cookie deprecation. DataCops is first-party by default. Most others are third-party.

**Bot-on-consent.** Recording consent from scraper traffic. Industry default behavior is to record what comes in, regardless of whether the client is human. This is a compliance exposure if your "consented users" log includes 8% bots.

## Architecture choice

Three patterns dominate.

**Pattern A: Vendor-hosted, third-party domain (most CMPs)**

Banner script loads from `cdn.cookiebot.com` or similar. Consent stored on vendor domain. Subject to cross-site cookie restrictions and ITP.

**Pattern B: Vendor-hosted, first-party CNAME (DataCops, OneTrust enterprise tier)**

Banner script loads from a subdomain you own (`datacops.yourdomain.com`). Consent stored first-party. Survives ITP and ad blockers.

**Pattern C: Self-hosted (rare; Borlabs Cookie via WordPress plugin)**

Plugin runs on your server, no external script. Strong privacy posture. Limited to WordPress.

## Install paths (skeleton)

### DataCops

```bash
# 1. Add CNAME: datacops.yourdomain.com -> cdn.yourdomain.com
# 2. Add to <head>:
# <script src="https://datacops.yourdomain.com/dc.js"></script>
# 3. Configure consent banner in dashboard (TCF 2.2 toggle, CMv2 mapping)
# 4. Verify CMv2 signals in Google Tag Assistant
```

Time to live: 5 to 30 minutes.

### Cookiebot

```bash
# 1. Sign up at cookiebot.com
# 2. Add the script to <head>:
# <script id="Cookiebot" src="https://consent.cookiebot.com/uc.js" data-cbid="YOUR-ID"></script>
# 3. Configure scan + categorization in dashboard
# 4. Wait for first scan to complete (24h)
```

Time to live: ~30 minutes plus 24h scan.

### OneTrust

```bash
# 1. Contract + procurement (1 to 4 weeks)
# 2. Onboarding workshop with OneTrust CSM
# 3. Domain setup, banner customization
# 4. CMv2 mapping
# 5. UAT in staging
# 6. Production rollout
```

Time to live: 6 to 12 weeks per OneTrust historical data.

### Ketch

```bash
# 1. Sign up at ketch.com (free up to 5K users/mo)
# 2. Add the script
# 3. Configure rules in builder
# 4. Migration tool from OneTrust available
```

Time to live: half day.

## Observability checklist

Whatever CMP you pick, your stack should answer these in under 5 minutes:

1. What's the current banner accept-rate (last 24h)?
2. What's the CMv2 signal health on Google Ads (modeled conversion delta)?
3. What's the TCF 2.2 string validation status?
4. Are bot consents being recorded (yes/no, what % of total)?
5. Of consents collected, what's the per-purpose breakdown?

If your CMP doesn't surface 1 to 5 in real time, you're flying blind.

## Failure modes worth wargaming

- **CMv2 misconfiguration**: documented 90% overnight Google Ads conversion drops. Mitigation: monitoring + diff vs 7-day rolling baseline.
- **TCF 2.2 string corruption**: Limited Ads default kicks in. Mitigation: TCF string validation in CI.
- **Vendor outage**: third-party CMP down. Mitigation: graceful banner fallback to "essential only" default.
- **Bot consent pollution**: 8% IVT recording consent. Mitigation: pre-consent fraud filter.
- **Vendor acquisition**: pricing or roadmap changes mid-contract. Mitigation: 12-month max contract length on uncertain vendors.

## Cost model at 100K sessions/mo

- DataCops Business: $49/mo (full bundle: CMP + analytics + CAPI + fraud)
- CookieHub Business: €30/mo
- Iubenda Advanced: $27.99 per site/mo
- Termly Pro+: $15/mo (covers up to 50K banner views, multi-site needs more)
- Cookiebot Premium Medium: €30 per domain/mo
- ConsentManager: €30 to €50/mo
- Didomi: typically $1K to $2K/mo for this volume
- OneTrust: $10K/yr minimum, mid-market $40K/yr+
- Ketch Plus: $499/mo

## Compliance posture comparison

| Tool | GDPR | CCPA | TCF 2.2 | CMv2 | DSAR API | SOC 2 Type II | ISO 27001 |
|---|---|---|---|---|---|---|---|
| OneTrust | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Ketch | Yes | Yes | Yes | Yes | Pro tier | Yes | Yes |
| DataGrail | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| DataCops | Yes | Yes | Yes | Yes (in progress) | Planned | In progress | Planned |
| Cookiebot | Yes | Yes | Yes | Yes | Premium | Yes | Yes |
| Iubenda | Yes | Yes | Yes (Pro) | Yes | Ultimate | Yes | Yes |
| CookieYes | Yes | Yes | No | Partial | No | No | No |

## Decision pseudo-code

```python
def pick_cmp(profile):
 if profile == "fortune_500_enterprise":
 return "OneTrust" if budget_unlimited else "DataGrail or Ketch"
 if profile == "engineering_led_privacy_program":
 return "Privado" if codebase_scan else "Securiti"
 if profile == "european_publisher":
 return "Didomi" if post_merger_clarity else "Sourcepoint wait-and-see"
 if profile == "wordpress_one_site":
 return "Borlabs Cookie or CookieYes"
 if profile == "shopify":
 return "Enzuzo"
 if profile == "smb_solo_saas":
 return "Termly or Iubenda"
 if profile == "want_cmp_plus_tracking_plus_capi_in_one":
 return "DataCops"
 if profile == "compliance_fear_top_concern":
 return "Osano" # $500K fines guarantee
 return "evaluate based on consent rate, CMv2 health, and renewal pressure"
```

## Further reading

- Full long-form comparison with 24 CMPs scored: https://joindatacops.com/blog/best-consent-management-platform-2026
- DataCops product context: https://joindatacops.com/first-party-consent-manager-platform
- IAB TCF 2.2 spec: https://iabeurope.eu/transparency-consent-framework
- Google CMv2 docs: https://developers.google.com/tag-platform/security/concepts/consent-mode

## Contributions

If a vendor changes pricing, certifications, or default behavior, open an issue with source URL and date. CMP market in 2026 is moving fast.

License: MIT for the doc. Each linked tool retains its own license and ToS.

---

Research by [DataCops](https://www.joindatacops.com) · First-party tracking, consent infrastructure & fraud prevention.
