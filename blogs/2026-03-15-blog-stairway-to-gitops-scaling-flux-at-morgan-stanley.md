---
title: "Blog: Stairway to GitOps: Scaling Flux at Morgan Stanley"
url: "https://fluxcd.io/blog/2026/03/stairway-to-gitops-morgan-stanley/"
date: "Sun, 15 Mar 2026 12:00:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
<img height="336" src="https://fluxcd.io/blog/2026/03/stairway-to-gitops-morgan-stanley/featured-image_hu58eaf01012de3c16f8b6669226f97dbb_231142_640x0_resize_box_3.png" width="640" />
<p>One of the things we love most about this community is hearing how you take Flux and run with it - truly solving problems for teams at scale. At our inaugural FluxCon NA, <strong>Tiffany Wang</strong> and <strong>Simon Bourassa</strong> from <strong>Morgan Stanley</strong> gave us a glimpse of their Flux environment.</p>
<p>Their talk, <strong>&ldquo;Stairway to GitOps,&rdquo;</strong> walked us through a five-year journey from push-based pipelines to a self-service GitOps platform managing over 500 clusters. Hearing the core principles of Flux - <strong>Lean, Performant, Extensible, and Secure</strong> - validated by end-users at this scale matters a lot to us as maintainers. We think their lessons are worth sharing with all of you.</p>
<p><img alt="Flux maintainers together at FluxCon NA 2025" src="https://fluxcd.io/img/fluxcon-na-25/maintainers-5.png" />
<em>Matheus Pimenta cracking a joke with the Flux team together at FluxCon NA 2025 - (Moments with all of these people in-person are rare!)</em></p>
<h2 id="the-early-days-pushing-limits">The Early Days: Pushing Limits</h2>
<p>Like many teams, Morgan Stanley started with traditional push-based CI/CD pipelines. App teams used tools like Helm to push manifests directly to clusters. While functional for initial deployments, challenges emerged as they scaled. Familiar pain points crept in:</p>
<ul>
<li><strong>Configuration Drift:</strong> Without an agent continuously reconciling state, clusters drifted from the source of truth in Git. Manual changes and failed deployments left systems in an unknown state.</li>
<li><strong>Fragile Recovery:</strong> Cluster rebuilds required heavy coordination. The platform team could restore infrastructure, but application teams had to manually redeploy their workloads. (Not a great place to be at 2 AM in another team&rsquo;s timezone)</li>
</ul>
<p>At &ldquo;Step 0&rdquo; of their Stairway to GitOps, they realized they needed to decouple delivery from the pipeline and embrace continuous reconciliation.</p>
<h2 id="step-1-security-and-self-service">Step 1: Security and Self-Service</h2>
<p>In a highly regulated financial environment, security isn&rsquo;t optional. The team chose Flux to fit their strict multi-tenancy model.</p>
<p>Morgan Stanley leveraged <strong>Flux&rsquo;s service account impersonation</strong> and native Kubernetes RBAC to enforce least-privilege access. Controllers reconciling manifests for one team had zero visibility into another team&rsquo;s resources. Granular, secure multi-tenancy is a first priority part of Flux&rsquo;s design, so this is the golden path, but implementing it always involves deciding what teams get what permissions, and they put in that work.</p>
<p>To streamline adoption, they built a <strong>self-service onboarding platform</strong>. Instead of requiring developers to manage low-level Kubernetes details, they created tooling that:</p>
<ol>
<li>Automated entitlement checks and change control processes.</li>
<li>Registered services in their CMDB.</li>
<li>&ldquo;Primed&rdquo; the target namespace with the necessary Flux <code>GitRepository</code> and <code>Kustomization</code> resources.</li>
<li>Scaffolded a ready-to-use application repository.</li>
</ol>
<p>This approach demonstrates Flux&rsquo;s extensibility. Flux can serve as the glue between systems. Developers interact with their normal tooling, while company specific systems like CMDB&rsquo;s (which likely predate Kubernetes adoption at all) integrate smoothly into the GitOps flow.</p>
<h2 id="step-2-operating-at-scale">Step 2: Operating at Scale</h2>
<p>As adoption grew, so did the deployment footprint. Tiffany shared some numbers from their environment:</p>
<blockquote>
<p><em>&ldquo;And now we have over 500 clusters, over 2,000 nodes, over 100,000 containers, and tens of thousands of Flux resources.&rdquo;</em> (13:34)</p>
</blockquote>
<p>Operating at this magnitude brings challenges around performance. The team shared how they tuned Flux to handle this load without overwhelming the Kubernetes control plane.</p>
<h3 id="tuning-for-performance">Tuning for Performance</h3>
<p>With tens of thousands of resources reconciling, the team started some performance tuning. Their focus areas:</p>
<ul>
<li><strong>Reconciliation Intervals:</strong> They increased their platform defaults, tuning intervals to balance responsiveness with load.</li>
<li><strong>Controller Concurrency:</strong> By adjusting the <code>--concurrent</code> flags on Flux controllers, they increased how many reconciliations could happen in parallel.</li>
<li><strong>Resource Management:</strong> They monitored and adjusted resource limits for Flux components to ensure reliability under sustained load.</li>
</ul>
<p>We put a lot of thought into making these knobs available. Flux should run well on a Raspberry Pi and on a fleet of 500 clusters alike. The platform team taking ownership of Flux&rsquo;s runtime in this manner shows operational excellence.</p>
<h3 id="moving-from-git-to-s3">Moving from Git to S3</h3>
<p>The team also moved from a self-hosted Git provider to <strong>S3 buckets</strong> as the source of truth for their clusters. Driven by high availability and compliance requirements, they built a mechanism to push artifacts from CI to S3. Because Flux&rsquo;s <code>Source Controller</code> supports various sources - Git, Helm repositories, OCI Repositories, and S3-compatible buckets - this transition was possible. The <strong>GitOps Toolkit</strong> architecture makes this kind of swap straightforward. You change the source layer but keep the delivery pipeline.</p>
<h2 id="step-3-observability-and-feedback-loops">Step 3: Observability and Feedback Loops</h2>
<p>Managing 500 clusters requires effective observability. The team built a centralized Grafana dashboard providing a unified view of their fleet.</p>
<p>They extended the open-source Flux dashboards with custom metrics from <code>kube-state-metrics</code>, tailored to their developers&rsquo; needs. At a glance, they could see which reconciliations were failing and why.</p>
<p>They also closed the developer experience loop by integrating Flux&rsquo;s <strong>Notification Controller</strong> - sending success and failure notifications directly to the pipelines and tools developers were already using.</p>
<h2 id="looking-ahead">Looking Ahead</h2>
<p>The team also shared what&rsquo;s next on their roadmap:</p>
<ul>
<li><strong>Flux Sharding:</strong> Exploring sharding Flux controllers to distribute load across multiple instances within a cluster.</li>
<li><strong>OCI Artifacts:</strong> Considering OCI artifacts as the primary source of truth, aligning with the &ldquo;Git-less GitOps&rdquo; model for improved performance and security.</li>
<li><strong>Progressive Delivery:</strong> Planning to adopt <strong>Flagger</strong> for canary and blue-green deployments, helping de-risk releases.</li>
</ul>
<p>It&rsquo;s cool to see a team that&rsquo;s been running Flux for five years still finding new ways to push it further. This is a sophisticated environment, and these improvements could win some performance and improve their developer experience further.</p>
<h2 id="watch-the-full-talk">Watch the Full Talk</h2>
<p>For the full story, including the architectural decisions and lessons learned, watch the recording:</p>
<div class="responsive-video">

</div>
<p>Thank you to Tiffany, Simon, and the team at Morgan Stanley for sharing their journey so openly. Stories like theirs remind us why we build Flux - what we build for the Raspberry Pi&rsquo;s in our closets at home is the same software that is so widely deployed all around us at enterprise scale. We can&rsquo;t help but wonder what wild stories we&rsquo;ll hear from you all next week at FluxCon and KubeCon!</p>
<h2 id="join-us-at-fluxcon-europe-2026">Join Us at FluxCon Europe 2026</h2>
<p>Inspired by Morgan Stanley&rsquo;s infra? Come connect with the community and learn from teams running Flux in production. <strong>
<a href="https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/co-located-events/fluxcon/" target="_blank">FluxCon Europe</a></strong> is happening on <strong>March 23, 2026</strong> at <strong>RAI Amsterdam</strong>, co-located with KubeCon. Speakers from KLM, NatWest Group, Orange, and more will be sharing their Flux stories.</p>
<p>We&rsquo;d love to see you there &ndash; come say hi! 🙂
We&rsquo;ll also be in the Project Pavilion all week. Catch up with us at
<a href="https://fluxcd.io/kubecon" target="_blank">fluxcd.io/kubecon</a> 👋</p>
