# Groundtruth — Investment Prospectus (Sample)

## Executive Summary

Groundtruth is a property data intelligence platform that provides Australians with institutional-grade property analysis at consumer prices. The platform aggregates, normalizes, and triangulates data from 14+ public and commercial sources to deliver comprehensive, evidence-based property reports.

**Problem:** Individual property buyers spend an average of 40 hours researching a single property, navigating fragmented government portals, paying for multiple premium data services, and still missing critical risk factors that professionals catch.

**Solution:** A single $39 report that consolidates flood, fire, planning, contamination, infrastructure, and demographic intelligence into a clear, actionable document — replacing $200+ in individual data requests.

---

## Market Opportunity

### TAM/SAM/SOM

| Metric | Value | Methodology |
|--------|-------|-------------|
| TAM | $2.4B | 600K annual Australian property transactions × $4,000 average spend on due diligence |
| SAM | $180M | 600K transactions × $300 average willingness-to-pay for aggregated reports |
| SOM (Year 1) | $2.1M | 2% penetration in NSW/VIC = ~12,000 reports at $39 + subscription revenue |

### Competitive Landscape

| Competitor | Coverage | Price | Limitation |
|-----------|----------|-------|------------|
| CoreLogic | National | $100+/report | Valuation-focused, not risk intelligence |
| Domain/REA | National | Free-$50 | Listing-centric, surface-level data |
| State Gov Portals | Per-state | Free-$20/each | Fragmented, 14+ separate lookups needed |
| **Groundtruth** | **National** | **$39/report** | **First aggregator with triangulation** |

---

## Revenue Model

### Pricing Tiers

| Tier | Price | Target |
|------|-------|--------|
| Single Report | $39 | One-off buyers |
| Monthly Pro | $79/mo (5 reports) | Investors, agents |
| Agency Plan | $199/mo (20 reports) | Real estate agencies |

### Unit Economics

- **CAC (Estimated):** $18 (digital channels — SEO, SEM, social)
- **LTV (Single Report):** $39 (one-shot)
- **LTV (Monthly Pro):** $948 (12-month average retention)
- **LTV:CAC Ratio (Pro):** 52:1
- **Gross Margin:** 85% (data costs ~$6/report after aggregation)

---

## Technology Architecture

### Data Pipeline

```
Raw Sources → ETL Normalization → PostGIS Spatial DB → Triangulation Engine → Report Generator
```

- **14 integrated data sources** covering flood, fire, contamination, planning, infrastructure
- **PostGIS spatial queries** with bounding-box optimization for <200ms lookups
- **Atomic ingestion** pipeline with rollback capability
- **"Strict Extension" protocol** — new data sources cannot break existing schemas

### Stack

- **Frontend:** Next.js (App Router)
- **Backend:** Supabase (PostgreSQL + PostGIS + Auth + Storage)
- **Hosting:** Vercel (frontend), Supabase (backend)
- **CI/CD:** GitHub Actions

---

## Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Data source API changes | High | Medium | Adapter pattern, automated monitoring |
| Competitor aggregator launch | Medium | High | First-mover advantage, proprietary triangulation |
| Regulatory data restrictions | Low | High | Compliance-first design, legal review |
| Customer acquisition cost escalation | Medium | Medium | Organic SEO strategy, referral program |

---

## Funding Requirements

**Seeking:** $150K seed round

| Use of Funds | Allocation |
|-------------|-----------|
| Engineering (6 months) | $80K |
| Data licensing | $25K |
| Marketing launch | $30K |
| Legal/compliance | $10K |
| Reserve | $5K |

**Milestone triggers:** $50K at MVP, $50K at 500 paid reports, $50K at MRR $5K.

---

## Team

- **Founder:** 15 years in manufacturing technology, proven track record building B2B SaaS platforms
- **Technical Advisor:** Senior GIS engineer, 10 years in spatial data systems
- **Legal Advisor:** Property law specialist, compliance and regulatory guidance

---

*Sample business plan for R&R testing — based on Groundtruth project*
