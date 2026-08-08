---
title: "Blog: Selective drift correction with ignore rules"
url: "https://fluxcd.io/blog/2026/08/ignore-rules-drift-detection/"
date: "2026-08-03"
feed_url: "https://fluxcd.io/blog/index.xml"
---
We are excited to introduce drift ignore rules for Flux Kustomizations, a long-requested capability that lets you tell Flux to leave specific fields alone during drift detection and correction, while continuing to reconcile everything else. One of the core promises of GitOps with Flux is continuous reconciliation: the kustomize-controller runs a periodic server-side apply dry-run to detect drift between the desired state in Git and the live state in the cluster, and corrects any divergence. This is exactly what you want for most fields; it guarantees that the cluster always matches the source 
