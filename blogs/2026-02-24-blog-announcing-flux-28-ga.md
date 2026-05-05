---
title: "Blog: Announcing Flux 2.8 GA"
url: "https://fluxcd.io/blog/2026/02/flux-v2.8.0/"
date: "Tue, 24 Feb 2026 11:00:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
<img height="360" src="https://fluxcd.io/blog/2026/02/flux-v2.8.0/featured-image_hu2678bc1273f30d966760c2cfdf3890dd_250090_640x0_resize_box_3.png" width="640" />
<p>We are thrilled to announce the release of
<a href="https://github.com/fluxcd/flux2/releases/tag/v2.8.0" target="_blank">Flux v2.8.0</a>!
In this post, we highlight some of the new features and improvements included in this release.</p>
<p><img alt="" src="featured-image.png" /></p>
<h2 id="highlights">Highlights</h2>
<p>Flux v2.8 comes with Helm v4 support, bringing server-side apply and enhanced health checking to Helm releases.
Big thanks to the Helm maintainers for their work on Helm v4 and for collaborating with us to ensure a smooth integration with Flux!</p>
<p>In this release, we have also introduced several new features to the Flux controllers:</p>
<ul>
<li>Reduced the mean time to recovery of Flux-managed applications</li>
<li>Readiness evaluation of Helm-managed objects with CEL expressions</li>
<li>ArtifactGenerator support for extracting and modifying Helm charts</li>
<li>Support for commenting on Pull Requests directly from Flux notifications</li>
<li>Custom SSA apply stages for ordering resource application in kustomize-controller</li>
<li>Automatic GitHub App installation ID lookup from the repository owner</li>
<li>Support for Cosign v3 for verifying OCI artifacts and container images</li>
</ul>
<p>In ecosystem news, there is a new release of
<a href="https://github.com/controlplaneio-fluxcd/flux-operator" target="_blank">Flux Operator</a>
that comes with a dedicated
<a href="https://fluxoperator.dev/web-ui/" target="_blank">Flux Web UI</a> and new providers for preview environments.</p>
<h2 id="helm-v4-support">Helm v4 Support</h2>
<p>Flux now ships with Helm v4. This brings two major improvements to how Flux manages Helm releases:
server-side apply and kstatus-based health checking.</p>
<p>With server-side apply, the API server takes ownership of merging fields, rather than the client.
This means fewer conflicts when multiple controllers or tools manage overlapping resources, and
more accurate drift detection out of the box.</p>
<p>Health checking now defaults to
<a href="https://github.com/fluxcd/cli-utils/tree/master/pkg/kstatus" target="_blank">kstatus</a>,
the same library used by the kustomize-controller. Instead of relying on Helm&rsquo;s legacy readiness
logic, Flux can now understand the actual rollout status of Deployments, StatefulSets, Jobs, and
other resources — including custom resources that follow standard status conventions. For teams
that rely on custom readiness logic, Flux now supports
<a href="https://fluxcd.io/flux/cheatsheets/cel-healthchecks/" target="_blank">CEL-based health check expressions</a> on
HelmReleases, giving you the same flexibility already available in the Kustomization API.</p>
<p>Both server-side apply and kstatus health checking are the new defaults. Because Helm persists
the apply method in its release storage, existing HelmReleases will continue to use client-side
apply until explicitly opted in. Health checking, on the other hand, will switch to kstatus for
all HelmReleases. For teams that prefer Helm v3&rsquo;s behavior across the board, the <code>UseHelm3Defaults</code>
feature gate restores the previous defaults.</p>
<p>Finally, HelmReleases now track an inventory of managed resources in <code>.status.inventory</code>,
giving you full visibility into what Flux has deployed — useful for debugging, auditing, and
building tooling on top of Flux.</p>
<h2 id="faster-recovery-from-failed-deployments">Faster recovery from failed deployments</h2>
<p>A common pain point with GitOps is the wait time after pushing a fix for a broken deployment. When a
release fails health checks, Flux would previously wait for the full timeout before acting — even if
a new revision was already available. This delay directly impacts the mean time to recovery (MTTR).</p>
<p>In Flux 2.7, the kustomize-controller introduced the <code>CancelHealthCheckOnNewRevision</code> feature gate,
allowing ongoing health checks to be canceled when a new source revision is detected. With Flux 2.8,
this capability has been extended to helm-controller and expanded to react to more kinds of changes:</p>
<ul>
<li>Changes in the resource spec (e.g. path, patches, images, values)</li>
<li>Changes in referenced ConfigMaps and Secrets (var substitutions, SOPS decryption keys, Kubeconfig)</li>
<li>Reconciliations triggered manually with <code>flux reconcile</code> or via notification-controller receivers</li>
</ul>
<p>In all these cases, Flux cancels the ongoing health checks and immediately starts reconciling the
new state. Instead of waiting several minutes for a failing release to time out, the fix is picked
up as soon as it lands.</p>
<p>For observability, a new <code>HealthCheckCanceled</code> reason is added to the <code>Ready</code> condition when this
happens.</p>
<p>This feature gate is opt-in for now, and we plan to enable it by default once the implementation is
stable across both controllers.</p>
<div class="alert alert-info">
<h4 class="alert-heading">Note</h4>
When enabling <code>CancelHealthCheckOnNewRevision</code> for helm-controller, enabling
<code>DefaultToRetryOnFailure</code> together is recommended. HelmReleases are more prone to get
stuck after the cancellation when using the default retry configuration (no retries).
</div>
<h2 id="ecosystem-news">Ecosystem News</h2>
<h3 id="flux-operator-web-ui">Flux Operator Web UI</h3>
<p>At KubeCon Atlanta 2025, the Flux maintainers from ControlPlane
gave a sneak peek of the new Flux Web UI, which is now available
in the latest release of Flux Operator.</p>
<p>The Flux Web UI provides a modern and user-friendly interface for managing and
monitoring your Flux-managed clusters. It offers a comprehensive view of your GitOps resources,
including:</p>
<ul>
<li>Cluster dashboard with sync statistics and overall system health</li>
<li>Deep-dive views for ResourceSets, HelmReleases and Kustomizations</li>
<li>Workload monitoring from deployment rollouts to pod statuses</li>
<li>Powerful search and filtering</li>
<li>Favorites for quick access to critical resources</li>
<li>SSO support via OIDC &amp; Kubernetes RBAC for multi-tenant clusters</li>
<li>GitOps Graphs for visualizing the delivery pipeline</li>
<li>GitOps Actions guarded by RBAC for manual interventions and incident response</li>
</ul>






<div class="gallery-wrapper" id="gallery-4857f48b2006deaad0ac3a72e5512a43-1-wrapper">
<div class="justified-gallery" id="gallery-4857f48b2006deaad0ac3a72e5512a43-1">
<div>
<a class="galleryImg" href="https://fluxcd.io/blog/2026/02/flux-v2.8.0/images/flux-status-home-light.png">
<img class="lazy" height="877" src="data:image/jpeg;base64,/9j/2wCEAAoHBwgHBgoICAgLCgoLDhgQDg0NDh0VFhEYIx8lJCIfIiEmKzcvJik0KSEiMEExNDk7Pj4&#43;JS5ESUM8SDc9PjsBCgsLDg0OHBAQHDsoIig7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7O//AABEIAB8AIAMBIgACEQEDEQH/xAGiAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgsQAAIBAwMCBAMFBQQEAAABfQECAwAEEQUSITFBBhNRYQcicRQygZGhCCNCscEVUtHwJDNicoIJChYXGBkaJSYnKCkqNDU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6g4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2drh4uPk5ebn6Onq8fLz9PX29/j5&#43;gEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoLEQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4&#43;Tl5ufo6ery8/T19vf4&#43;fr/2gAMAwEAAhEDEQA/APW&#43;kn7pVLjJPqKkRpjneCPpSAB5cA9OWGzr&#43;NSqrKevHoBigVhuW9W/KmHzQ6hVGzuTwR&#43;GKkAkDElsjsPSkKyb87/l/u4FAWIWQNPjLZ91OPzqQRMOjgfTP&#43;NGFDl97fL1GTj8qcZVAByOenFMYgSRTkOv45P9akydvOM47Uc&#43;gprOAQpwCaQH/9k=" width="900" />
</a>
</div>
<div>
<a class="galleryImg" href="https://fluxcd.io/blog/2026/02/flux-v2.8.0/images/flux-status-helmrelease-light.png">
<img class="lazy" height="900" src="data:image/jpeg;base64,/9j/2wCEAAoHBwgHBgoICAgLCgoLDhgQDg0NDh0VFhEYIx8lJCIfIiEmKzcvJik0KSEiMEExNDk7Pj4&#43;JS5ESUM8SDc9PjsBCgsLDg0OHBAQHDsoIig7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7O//AABEIACAAHgMBIgACEQEDEQH/xAGiAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgsQAAIBAwMCBAMFBQQEAAABfQECAwAEEQUSITFBBhNRYQcicRQygZGhCCNCscEVUtHwJDNicoIJChYXGBkaJSYnKCkqNDU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6g4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2drh4uPk5ebn6Onq8fLz9PX29/j5&#43;gEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoLEQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4&#43;Tl5ufo6ery8/T19vf4&#43;fr/2gAMAwEAAhEDEQA/APWgQkoKhR13MBzU25v7x/KmhwkhyW5PrnFTAY75&#43;tBNiMkkYJJ/CmAy7zwMdj3/AJVIrTFuYlUf73/1qcc45AH0NA7EWCJc7W5PXJx&#43;VTgcc4P4VFyCcD8&#43;f607f6pn8v8AGmMI3D54xinMOKbv/wBg/p/jRnI&#43;7igD/9k=" width="848" />
</a>
</div>
<div>
<a class="galleryImg" href="https://fluxcd.io/blog/2026/02/flux-v2.8.0/images/flux-status-graph-light.png">
<img class="lazy" height="900" src="data:image/jpeg;base64,/9j/2wCEAAoHBwgHBgoICAgLCgoLDhgQDg0NDh0VFhEYIx8lJCIfIiEmKzcvJik0KSEiMEExNDk7Pj4&#43;JS5ESUM8SDc9PjsBCgsLDg0OHBAQHDsoIig7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7O//AABEIACAAGwMBIgACEQEDEQH/xAGiAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgsQAAIBAwMCBAMFBQQEAAABfQECAwAEEQUSITFBBhNRYQcicRQygZGhCCNCscEVUtHwJDNicoIJChYXGBkaJSYnKCkqNDU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6g4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2drh4uPk5ebn6Onq8fLz9PX29/j5&#43;gEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoLEQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4&#43;Tl5ufo6ery8/T19vf4&#43;fr/2gAMAwEAAhEDEQA/APWlWUTExiPH8RGN1Tfvff8ASnCIBtwZue2eKTfhymzOO&#43;RzVXFYawlxwMntnFC&#43;aFAbAPtUw6dKY3WlcAaVYyqt1bpTfK3SlyFI&#43;nP50jp5jowYjZ&#43;tSrwKBjI5lccdqc33qasaou0dPwpT1oA//9k=" width="772" />
</a>
</div>
<div>
<a class="galleryImg" href="https://fluxcd.io/blog/2026/02/flux-v2.8.0/images/flux-status-favorites-light.png">
<img class="lazy" height="664" src="data:image/jpeg;base64,/9j/2wCEAAoHBwgHBgoICAgLCgoLDhgQDg0NDh0VFhEYIx8lJCIfIiEmKzcvJik0KSEiMEExNDk7Pj4&#43;JS5ESUM8SDc9PjsBCgsLDg0OHBAQHDsoIig7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7O//AABEIABgAIAMBIgACEQEDEQH/xAGiAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgsQAAIBAwMCBAMFBQQEAAABfQECAwAEEQUSITFBBhNRYQcicRQygZGhCCNCscEVUtHwJDNicoIJChYXGBkaJSYnKCkqNDU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6g4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2drh4uPk5ebn6Onq8fLz9PX29/j5&#43;gEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoLEQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4&#43;Tl5ufo6ery8/T19vf4&#43;fr/2gAMAwEAAhEDEQA/APV/tEUUr7lcMCeUhY/rtp/9oQ/9Nv8Avw/&#43;FKmDKQGLH&#43;7uOBRMVWQAsVGOgJ5pak&#43;8MbU4FOD5/wCFvIf/AGWnwXUdxnyg/wAuM74mT8sipJs&#43;WP4OeoNOUfKTtI&#43;pP9aa8yuhEhbzMEbVx1BNLIW8wFBkdz6U7vRQISX7nyZY&#43;jZpUJ2/Ox3egJxRQOooGj//2Q==" width="900" />
</a>
</div>
</div>
</div>

<p>Get started by installing the latest version of Flux Operator and following the
<a href="https://fluxoperator.dev/web-ui/" target="_blank">Flux Web UI documentation</a>.</p>
<h3 id="preview-environments">Preview environments</h3>
<p>Flux Operator&rsquo;s
<a href="https://fluxoperator.dev/docs/crd/resourceset/" target="_blank">ResourceSet API</a> makes it easy to
deploy ephemeral preview environments from GitHub Pull Requests and GitLab Merge Requests. With
Flux 2.8, closing the feedback loop on these environments is now much simpler thanks to new
notification-controller provider types: <code>githubpullrequestcomment</code>, <code>gitlabmergerequestcomment</code>
and <code>giteapullrequestcomment</code>.</p>
<p>Previously, posting deployment status on a Pull Request required setting up a <code>githubdispatch</code>
provider and a GitHub Actions workflow to parse the event payload and post a comment. With the new
providers, notification-controller posts and updates comments directly on the PR or MR page — no
CI workflow needed. Comments are automatically deduplicated, so the PR stays clean with a single
status comment that gets updated on each deployment.</p>
<p>In addition, commit status reporting now works with any Flux API — not just Kustomizations and
GitRepositories. This means HelmReleases deployed for preview environments can now report their
status as commit checks on the PR, giving developers immediate visibility into whether their
changes deployed successfully.</p>
<p>To annotate your Flux resources for these providers, use the standard event metadata keys:</p>
<ul>
<li><code>event.toolkit.fluxcd.io/change_request</code> — identifies the PR/MR number for comment providers</li>
<li><code>event.toolkit.fluxcd.io/commit</code> — identifies the commit SHA for commit status providers</li>
</ul>
<p>For complete setup guides, see the Flux Operator documentation:</p>
<ul>
<li>
<a href="https://fluxoperator.dev/docs/resourcesets/github-pull-requests/" target="_blank">Ephemeral Environments for GitHub Pull Requests</a></li>
<li>
<a href="https://fluxoperator.dev/docs/resourcesets/gitlab-merge-requests/" target="_blank">Ephemeral Environments for GitLab Merge Requests</a></li>
</ul>
<h2 id="supported-versions">Supported Versions</h2>
<p>Flux v2.5 has reached end-of-life and is no longer supported.</p>
<p>Flux v2.8 supports the following Kubernetes versions:</p>
<table>
<thead>
<tr>
<th style="text-align: left;">Distribution</th>
<th style="text-align: left;">Versions</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: left;">Kubernetes</td>
<td style="text-align: left;">1.33, 1.34, 1.35</td>
</tr>
<tr>
<td style="text-align: left;">OpenShift</td>
<td style="text-align: left;">4.20</td>
</tr>
</tbody>
</table>
<blockquote>
<p><strong>Enterprise support</strong> Note that the CNCF Flux project offers support only for the latest three minor versions of Kubernetes.
Backwards compatibility with older versions of Kubernetes and OpenShift is offered by vendors such as
<a href="https://control-plane.io/enterprise-for-flux-cd/" target="_blank">ControlPlane</a> that provide enterprise support for Flux.</p>
</blockquote>
<h2 id="upgrade-procedure">Upgrade Procedure</h2>
<p>Note that in Flux v2.8, the following APIs have reached end-of-life and have been removed from the CRDs:</p>
<ul>
<li><code>source.toolkit.fluxcd.io/v1beta2</code></li>
<li><code>kustomize.toolkit.fluxcd.io/v1beta2</code></li>
<li><code>helm.toolkit.fluxcd.io/v2beta2</code></li>
</ul>
<p>Before upgrading to Flux v2.8, make sure to migrate all your resources to the stable APIs
using the
<a href="https://fluxcd.io/flux/cmd/flux_migrate/">flux migrate</a> command.</p>
<div class="alert alert-info">
<h4 class="alert-heading">Upgrade Procedure for Flux v2.8</h4>
We have published a dedicated step-by-step upgrade guide, please follow the instructions from
<a href="https://github.com/fluxcd/flux2/discussions/5572" target="_blank">Upgrade Procedure for Flux v2.7+</a>.
</div>
<h2 id="over-and-out">Over and out</h2>
<p>If you have any questions or simply just like what you read and want to get involved,
here are a few good ways to reach us:</p>
<ul>
<li>Join our
<a href="https://fluxcd.io/community/#meetings" target="_blank">upcoming dev meetings</a>.</li>
<li>Talk to us in the #flux channel on
<a href="https://slack.cncf.io/" target="_blank">CNCF Slack</a>.</li>
<li>Join the
<a href="https://github.com/fluxcd/flux2/discussions" target="_blank">planning discussions</a>.</li>
<li>Follow
<a href="https://twitter.com/fluxcd" target="_blank">Flux on Twitter</a>, or join the
<a href="https://www.linkedin.com/groups/8985374/" target="_blank">Flux LinkedIn group</a>.</li>
</ul>
