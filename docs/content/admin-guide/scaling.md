---
title: Scaling
sidebar_position: 9
---

# Scaling

RDFArchitect is designed for small teams (dozens of users) rather than public internet scale. The limiting factor is almost always the triple store:

- One Fuseki instance on modest hardware (4 cores, 8 GB RAM, SSD) comfortably serves a team of 20–30 concurrent modellers on a full set of CGMES + ENTSO-E profiles.
- The backend is stateless and can be scaled horizontally behind a load balancer, with session affinity, if needed. The frontend is static and trivially scalable.
- The triple store itself is not horizontally scalable in the open-source Fuseki. For very large or very busy deployments, a commercial triple store that supports clustering can be dropped in — RDFArchitect only requires SPARQL 1.1 + GSP.
