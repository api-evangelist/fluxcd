---
title: "Blog: Flux puts the Git into GitOps"
url: "https://fluxcd.io/blog/2022/03/flux-puts-the-git-into-gitops/"
date: "Fri, 25 Mar 2022 09:30:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
Ever since the rewrite of Flux as a set of focused controllers, it has become clearer what each of its functions and capabilities are. The aptly named controllers carry in their name what they are responsible for and which data or tooling they interact with, so that is, e.g. source , kustomize , image-automation , notification , helm , etc. If you wanted to string a proof-of-concept for a GitOps tool together, a naïve solution could be to just shell out to various tools like curl , git , kubectl and helm .
