---
title: "Blog: Introducing Flux Schema and the Ecosystem Catalog"
url: "https://fluxcd.io/blog/2026/07/flux-schema-validation/"
date: "2026-07-13"
feed_url: "https://fluxcd.io/blog/index.xml"
---
In this blog post, we introduce Flux Schema , a new Flux CLI plugin for validating Kubernetes manifests against JSON Schema and CEL rules using the same evaluation semantics as the Kubernetes API server. Alongside the plugin, we’re announcing the Ecosystem Schema Catalog , a hosted catalog of JSON Schemas and LLM-optimized indexes covering the Kuberentes ecosystem of controllers, served to CI pipelines over CDN and to AI agents over MCP. Why Another Validation Tool The GitOps workflow has a well-known blind spot: a manifest with a typo, a wrong type, or a violated CEL rule sails through git pu
