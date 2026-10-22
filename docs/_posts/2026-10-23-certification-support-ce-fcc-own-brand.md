---
layout: post
title: "Certification Support: Get CE and FCC Made Out to Your Own Name"
date: 2026-10-23 08:00:00 +0800
categories: [guide]
tags: [CE, FCC, RED, certification, OEM, mmwave radar]
---

We supply **modules**, not finished certified consumer products. That distinction decides who holds the paperwork: **you do.** If you build a product around our radar and take it through certification, the certificate names the manufacturer who places it on the market — and that is your company, not us.

This article explains what that arrangement means in practice, what we put behind you, and what remains yours to do.

> We are an engineering component supplier. This page explains how we work; it is **not legal or regulatory advice**, and it is not a certification. Always confirm requirements with the test laboratory and, where one is required, the notified body or FCC-recognised body you engage.

## Why the certificate ends up in your name

Radio equipment rules are built around the **economic operator** who puts the product on the market, not around whoever made the chip inside it.

- **European Union.** For CE marking, the party placing equipment on the market carries responsibility for the declaration of conformity, the technical documentation, and the marking. If that is you, the CE documentation bears your name.
- **United States.** The FCC authorisation is held by a grantee — the party applying for and responsible for the equipment authorization — plus a domestic responsible party for imported devices. If your company applies, your company is the grantee of record.

Either way the result is the same and entirely normal for OEM business: **you own the certificate and can build a product line on it.** What we can offer is everything upstream of the test report.

## What we provide

- **Modules and samples** in production configuration, including the UART and GPIO variants described in the [module spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}).
- **RF and antenna test data** for the module — operating frequency band, transmit power, antenna beam figures, receiver parameters.
- **Technical file material** — datasheets, protocol documentation (including the frame formats our modules speak), interface and pinout descriptions, recommended enclosure and mounting constraints.
- **Technical support** — answers to questions that come up during testing, and clarification of how the module behaves through the test procedures.

## What you do

You select and engage your own test laboratory, run the test campaign, assemble the technical documentation, issue the declaration, apply labeling, and retain records. We do not perform or obtain certification on your behalf and we do not appear on your certificate.

Practical sequence, EU and US alike:

1. **Engage a laboratory** accredited for radio testing in your target market.
2. **Define the scope and standard(s)** with the lab — including the applicable harmonised standard(s) covering the 57–71 GHz short-range devices in the EU, and the applicable FCC Part 15 rules for unlicensed operation in the same band in the US. Your lab fixes the exact references and versions; ask them early, because standard revisions move.
3. **Provide the technical file inputs** we list above.
4. **Test** — radiated emissions, unwanted emissions in adjacent bands, band edges, and any coexistence or exposure assessment the applicable rules require for your housing and installation conditions.
5. **Assemble the documentation** — description, drawings, block and electrical diagrams, list of applied standards, test reports, calculations, for the EU case, and the DoC (Declaration of Conformity).
6. **Where required, involve a notified body** — some equipment categories and some articles of the Radio Equipment Directive require third-party assessment.
7. **Apply marking and keep records** — CE marking plus notified-body number where relevant; FCC labelling plus retention of test records.
8. **Re-assess when the design changes.** A new enclosure, antenna placement, or output power triggers another look.

Two points from our own products worth flagging to your lab in advance: the **enclosure must be non-metallic and flat over the antenna face**, and our modules specify **power-supply ripple limits**. Both affect radiated behaviour and both appear in the [presence relay article]({{ site.baseurl }}{% post_url 2026-10-21-presence-s1-relay-occupancy %}) and the spec sheet.

## Honest boundaries

- We do **not** hold CE or FCC certificates for these modules, and we do not supply one for you to re-use. A certificate or grant has to be in the name of the party taking responsibility for the equipment.
- We do **not** guarantee a pass. Test outcomes depend on your housing, antenna keep-out, power design, cabling, and installation; those are yours.
- We do **not** certify medical devices, and nothing we sell is a medical device. If your product makes a medical claim, that is a separate regulatory route with separate requirements.
- Timelines and costs belong to you and your laboratory. Get them quoted in writing before committing to a design.

## When you might not need any of this

If you are integrating a **finished unit** rather than a module, check whether that product line already carries the relevant approvals for your market, and confirm with the supplier in writing before you plan around it. Building a product around a bare module — the path we support — is what puts certification in your hands.

## Getting started

Tell us what you are building, the target market, and which module you plan to use. We will confirm what documentation and sample configuration we can supply for that variant, and flag anything in your enclosure or mounting plan that will make testing harder.

Our modules and evaluation units are on [Tindie](https://www.tindie.com/stores/fuxinsuo/); see the [spec sheet]({{ site.baseurl }}{% post_url 2026-10-21-60ghz-24ghz-radar-modules-oem-integration %}) for the three parts we build. Related reading: [Aging-in-place reference system]({{ site.baseurl }}{% post_url 2026-10-23-aging-in-place-care-system-three-radars %}) and [running your own broker]({{ site.baseurl }}{% post_url 2026-10-23-apache-artemis-mqtt-broker-radar %}).
