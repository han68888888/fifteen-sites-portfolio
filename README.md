# Fifteen Sites — Portfolio Landing Page

A static landing page indexing fifteen standalone websites, each built with its own
design language, palette, typography pairing and bespoke scroll interaction.

## Deploy to Netlify

This repo is a **static site with no build step**.

1. Netlify → *Add new site* → *Import an existing project* → pick this GitHub repo.
2. Leave the build command **empty**.
3. Set publish directory to **`.`** (repo root).
4. Deploy.

`netlify.toml` already encodes those settings, so the defaults should be picked up
automatically.

## The fifteen sites

| # | Project | Category | Live URL |
|---|---------|----------|----------|
| 01 | Ashbourne University | Education | https://ashbourne-university.higgsfield.app |
| 02 | Kamer Studio | Architecture | https://kamer-studio.higgsfield.app |
| 03 | Pelle Skin Clinic | Dermatology | https://pelle-clinic.higgsfield.app |
| 04 | Strand Institute | Hair clinic | https://strand-institute.higgsfield.app |
| 05 | Tand Dental | Dental | https://tand-dental.higgsfield.app |
| 06 | Forma Atelier | Plastic surgery | https://forma-atelier.higgsfield.app |
| 07 | Halide Studio | Photography | https://halide-studio.higgsfield.app |
| 08 | Foundry Athletic Club | Premium gym | https://foundry-club.higgsfield.app |
| 09 | Soleia | Salon & spa | https://soleia-spa.higgsfield.app |
| 10 | Iris Vale | Personal brand | https://iris-vale.higgsfield.app |
| 11 | Rep House | Strength gym | https://rep-house.higgsfield.app |
| 12 | Maren | Yoga | https://maren-yoga.higgsfield.app |
| 13 | The Neighborhood Pack | Pet services | https://neighborhood-pack.higgsfield.app |
| 14 | Meridian | Model agency | https://meridian-models.higgsfield.app |
| 15 | Altus | Real estate | https://altus-estates.higgsfield.app |

## Note on the site source code

The fifteen sites themselves are **not** in this repo. They are React 19 + TanStack Start
apps that were built in, and are hosted by, the Higgsfield platform — each one lives in its
own platform-managed git remote and is served as a Cloudflare Worker.

Two consequences:

- **Source is not included here.** Platform repo access was revoked, so the application
  source could not be exported into this repository.
- **They would not deploy to Netlify unmodified.** They are server-rendered on Cloudflare
  Workers; moving them to Netlify would require swapping the deployment adapter and
  re-testing each build. The landing page in this repo is static and deploys cleanly.

## Known issues in the deployed sites

See [`ISSUES.md`](./ISSUES.md) for the full audit, the confirmed root causes of the
scroll/stutter/blank-image problems, and the exact patch to apply once source access is
restored.
