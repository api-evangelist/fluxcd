---
title: "Blog: May 2022 Security Announcement"
url: "https://fluxcd.io/blog/2022/05/may-2022-security-announcement/"
date: "Tue, 10 May 2022 08:30:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
tl;dr The Flux Team has found three security vulnerabilities in Flux, and we strongly advise you to upgrade your clusters as soon as you can. CVE Advisory Severity Affected versions CVE-2022-24817 Improper kubeconfig validation allows arbitrary code execution Critical < 0.29.0 >= v0.1.0 CVE-2022-24877 Improper path handling in Kustomization files allows path traversal Critical < v0.29.0 CVE-2022-24878 Improper path handling in Kustomization files allows for denial of service High < v0.29.0 >= v0.19.0 Breaking changes to be aware of in the upgrade process: 0.29 , 0.28 , 0.27 , 0.26 , 0.24 -…
