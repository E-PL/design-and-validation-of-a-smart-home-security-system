# 3. Tradeoff Analysis & Technology Decisions

## Contents

**Main Deliverable:**
- [Decision-Matrices.xlsx](Decision-Matrices.xlsx) - 7 decision matrices with weighted scoring

**Supporting File:**
- [PHASE3_SUMMARY.md](PHASE3_SUMMARY.md) - Detailed analysis and rationale

## Summary

7 comprehensive decision matrices (weighted 0-1 scoring):

1. **Motion Sensors** → Active IR Selected (0.40 score, 99.5% reliability, 200ms response)
2. **Door/Window Sensors** → Magnetic Contact Selected (0.49 score, 99.8% reliability)
3. **Water Leak Sensors** → Resistive Cord Selected (most cost-effective)
4. **Cameras** → Multi-Tier Strategy (4K PoE primary, doorbell secondary, WiFi/4G tertiary)
5. **Controllers** → WiFi + Z-Wave Backup (primary + fallback for reliability)
6. **Server/Hub Architecture** → Local Hub + Optional Cloud (hybrid approach)
7. **Connectivity** → Triple Redundancy (WiFi primary, LoRaWAN secondary, Cellular tertiary)

**Key Outcomes:**
- All decisions are vendor-agnostic and objective
- Multiple weighted criteria (not just cost)
- Total hardware cost: $4,335 (one-time)
- Operating cost: $120-760/year (highly optional, cloud-dependent)
