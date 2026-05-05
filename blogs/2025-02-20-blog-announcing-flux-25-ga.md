---
title: "Blog: Announcing Flux 2.5 GA"
url: "https://fluxcd.io/blog/2025/02/flux-v2.5.0/"
date: "Thu, 20 Feb 2025 12:00:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
<img height="360" src="https://fluxcd.io/blog/2025/02/flux-v2.5.0/featured-image_hu01632177776d3af78deffdce84473b92_598056_640x0_resize_box_3.png" width="640" />
<p>We are thrilled to announce the release of
<a href="https://github.com/fluxcd/flux2/releases/tag/v2.5.0" target="_blank">Flux v2.5.0</a>!
In this post, we will highlight some of the new features and improvements included in this release.</p>
<p><img alt="" src="featured-image.png" /></p>
<h2 id="highlights">Highlights</h2>
<p>Flux v2.5 marks a significant milestone in the project&rsquo;s evolution, we have integrated Common Expression Language (CEL)
with the Flux controllers to enable long-awaited features such as custom health checks and webhook receiver filters.
Moreover, we have added support for GitHub App authentication, custom event metadata for notifications and Flux CLI helpers
for troubleshooting Flux resources.</p>
<p>In ecosystem news, the
<a href="https://github.com/controlplaneio-fluxcd/flux-operator" target="_blank">Flux Operator</a> v0.14 release brings
one of the most requested features: deploy app code and/or config changes made in a GitHub Pull Request
or GitLab Merge Request to an ephemeral environment for testing and validation.</p>
<p>The Flux Operator has the ability to create, update and delete application instances on-demand based
on the
<a href="https://fluxcd.control-plane.io/operator/resourcesets/introduction/" target="_blank">ResourceSet</a>
definitions and Pull/Merge Requests state.</p>
<p>For more details on how to use the ephemeral environments feature, see the following guides:</p>
<ul>
<li>
<a href="https://fluxcd.control-plane.io/operator/resourcesets/github-pull-requests/" target="_blank">Ephemeral Environments for GitHub Pull Requests</a></li>
<li>
<a href="https://fluxcd.control-plane.io/operator/resourcesets/gitlab-merge-requests/" target="_blank">Ephemeral Environments for GitLab Merge Requests</a></li>
</ul>
<h3 id="health-checks-for-custom-resources">Health Checks for Custom Resources</h3>
<p>In this release, we have extended the Flux
<a href="https://fluxcd.io/flux/components/kustomize/kustomizations/">Kustomization</a> API
with support for defining custom health checks using Common Expression Language (CEL).
The health checks are used to verify the readiness of the resources managed by Flux and are a key feature
for ensuring that the desired state of the cluster is achieved.</p>
<p>While Flux performs a series of built-in health checks for Kubernetes core resources, the new feature
allows users to teach Flux how to check the health of Kubernetes custom resources.
This is particularly useful for custom resources that do not subscribe to the Kubernetes API conventions
or for resources that require additional logic to determine if they reached the desired state.</p>
<p>A common use case for custom health checks is to verify the status of <code>Cluster</code> objects reconciled by
the
<a href="https://cluster-api.sigs.k8s.io/" target="_blank">Cluster API</a> controllers. When Flux is used to manage a fleet
of Kubernetes clusters, the health checks can be used to ensure that the clusters are ready before
deploying cluster addons and applications.</p>
<p>Example of a Kustomization with a custom health check for Cluster API:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>kustomize.toolkit.fluxcd.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Kustomization<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>prod-clusters<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>infra<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">interval</span>:<span style="color: #bbb;"> </span>30m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">retryInterval</span>:<span style="color: #bbb;"> </span>5m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">prune</span>:<span style="color: #bbb;"> </span><span style="color: #007020; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sourceRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>GitRepository<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>fleet<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">path</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"./production"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">timeout</span>:<span style="color: #bbb;"> </span>15m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">wait</span>:<span style="color: #bbb;"> </span><span style="color: #007020; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">healthCheckExprs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>cluster.x-k8s.io/v1beta1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Cluster<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">failed</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"status.conditions.filter(e, e.type == 'Ready').all(e, e.status == 'False')"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">current</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"status.conditions.filter(e, e.type == 'Ready').all(e, e.status == 'True')"</span><span style="color: #bbb;">
</span></span></span></code></pre></div><p>The above example configures Flux to wait for all the <code>Cluster</code> objects to reach the Ready state
before proceeding with the reconciliation of other Kustomizations that have a
<a href="https://fluxcd.io/flux/components/kustomize/kustomizations/#dependencies">dependsOn</a> relationship
defined for the <code>prod-clusters</code>.</p>
<p>We have published a
<a href="https://fluxcd.io/flux/cheatsheets/cel-healthchecks/">health check library</a> that contains CEL
expressions for popular custom resources. The library is community-maintained, and we encourage
users to contribute new health checks.</p>
<p>Other kustomize-controller improvements include:</p>
<ul>
<li>Fine-grained control of garbage collection with
<a href="https://fluxcd.io/flux/components/kustomize/kustomizations/#deletion-policy" target="_blank">.spec.deletionPolicy</a>.</li>
<li>SOPS support for decryption of Kubernetes secrets generated by Kustomize components.</li>
</ul>
<h3 id="github-app-authentication-for-git-repositories">GitHub App Authentication for Git Repositories</h3>
<p>Starting with Flux v2.5, you can configure source-controller and image-automation-controller
to authenticate against GitHub repositories using a GitHub App installation.</p>
<p>Instead of relying on personal access tokens or SSH keys that require manual rotation,
you can now configure Flux to authenticate against GitHub repositories using an identity
that is not tied to a specific user account.</p>
<p>We have added a new command to the Flux CLI that can be used to create the Kubernetes Secret
required for the GitHub App authentication.</p>
<div class="highlight"><pre tabindex="0"><code class="language-shell"><span style="display: flex;"><span>flux create secret githubapp github-auth <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> --app-id<span style="color: #666;">=</span><span style="color: #40a070;">1</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> --app-installation-id<span style="color: #666;">=</span><span style="color: #40a070;">2</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> --app-private-key<span style="color: #666;">=</span>~/private-key.pem
</span></span></code></pre></div><p>The Kubernetes Secret generated by the above command can be referenced in a <code>GitRepository</code>
and <code>ImageUpdateAutomation</code> with <code>.spec.secretRef.name</code>.</p>
<p>For more details on how to configure the GitHub App authentication, see the
<a href="https://fluxcd.io/flux/components/source/gitrepositories/#github" target="_blank">GitRepository API documentation</a>.</p>
<h3 id="custom-event-metadata-for-notifications">Custom event metadata for notifications</h3>
<p>Starting with Flux v2.5, users can enrich the metadata of the events sent by the notification-controller
by adding annotations on the Flux <code>Kustomization</code> and <code>HelmRelease</code> resources.
The metadata is included in the notifications sent to the configured providers, such as Slack, Microsoft Teams, etc.,
and can be used to provide additional context about a particular application or environment.</p>
<p>One highly requested feature was the ability to include the image tag in the notifications send when
Flux image automation updates the container image tag in HelmRelease values.</p>
<p>Example of a HelmRelease with custom event metadata containing the image tag:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>helm.toolkit.fluxcd.io/v2<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>HelmRelease<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">annotations</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">event.toolkit.fluxcd.io/image</span>:<span style="color: #bbb;"> </span>docker.io/org/my-app:1.0.0<span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># {"$imagepolicy": "apps:my-app"}</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">chart</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">chart</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sourceRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>HelmRepository<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>podinfo<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">values</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">image</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">tag</span>:<span style="color: #bbb;"> </span><span style="color: #40a070;">1.0.0</span><span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># {"$imagepolicy": "apps:my-app:tag"}</span><span style="color: #bbb;">
</span></span></span></code></pre></div><p>When the image automation updates the <code>my-app</code> HelmRelease with a new image tag e.g. <code>1.0.1</code>,
the notification sent after the Helm release upgrade will include <code>image: docker.io/org/my-app:1.0.1</code>
in message body.</p>
<p>For more details on how to configure custom event metadata, see the
<a href="https://fluxcd.io/flux/components/notification/alerts/#event-metadata-from-object-annotations" target="_blank">Alert API documentation</a>.</p>
<p>Other notifications improvements include:</p>
<ul>
<li>The notification-controller is now capable of updating
<a href="https://fluxcd.io/flux/cheatsheets/oci-artifacts/#git-commit-status-updates" target="_blank">Git commit statuses</a>
from events about Kustomizations that consume OCIRepositories.</li>
<li>The
<a href="https://fluxcd.io/flux/components/notification/receivers/#filtering-reconciled-objects-with-cel" target="_blank">Receiver API</a>
now supports filtering the declared resources that match a given Common Expression Language (CEL) expression.</li>
</ul>
<h3 id="cli-improvements">CLI Improvements</h3>
<p>To help users troubleshoot Flux, we&rsquo;ve added a new <code>flux debug</code> command the following subcommands:</p>
<ul>
<li><code>flux debug kustomization --show-vars</code> used to inspect the final variables values by merging the Flux <code>Kustomization</code>
inline vars with the vars coming from Kubernetes ConfigMaps/Secrets.</li>
<li><code>flux debug helmrelease --show-values</code> used to inspect the final Helm values by merging the <code>HelmRelease</code>
inline values with the values coming from Kubernetes ConfigMaps/Secrets.</li>
</ul>
<p>Note that these commands will print sensitive information if Kubernetes Secrets are referenced in
the Flux <code>Kustomization</code> or <code>HelmRelease</code> resources.</p>
<p>Other CLI improvements include:</p>
<ul>
<li>A new command was added, <code>flux create secret githubapp</code> that can be used to generate a Kubernetes Secret
for GitHub App authentication.</li>
<li>The <code>flux create source git</code> command now supports the <code>--provider=github</code> flag to configure GitHub App authentication
for Git repositories.</li>
</ul>
<h2 id="supported-versions">Supported Versions</h2>
<p>Flux v2.2 has reached end-of-life and is no longer supported.</p>
<p>Flux v2.5 supports the following Kubernetes versions:</p>
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
<td style="text-align: left;">1.30, 1.31, 1.32</td>
</tr>
<tr>
<td style="text-align: left;">OpenShift</td>
<td style="text-align: left;">4.17</td>
</tr>
</tbody>
</table>
<div class="alert alert-info">
<h4 class="alert-heading">Enterprise support</h4>
<p>Note that the CNCF Flux project offers support only for the latest
three minor versions of Kubernetes.</p>
<p>Backwards compatibility with older versions of Kubernetes and OpenShift is offered by vendors
such as
<a href="https://control-plane.io/enterprise-for-flux-cd/" target="_blank">ControlPlane</a> that provide
enterprise support for Flux.</p>
</div>
<h2 id="over-and-out">Over and out</h2>
<p>If you have any questions, or simply just like what you read and want to get involved,
here are a few good ways to reach us:</p>
<ul>
<li>Join our
<a href="https://fluxcd.io/community/#meetings" target="_blank">upcoming dev meetings</a>.</li>
<li>Join the
<a href="https://lists.cncf.io/g/cncf-flux-dev" target="_blank">Flux mailing list</a> and let us know what you need help with.</li>
<li>Talk to us in the #flux channel on
<a href="https://slack.cncf.io/" target="_blank">CNCF Slack</a>.</li>
<li>Join the
<a href="https://github.com/fluxcd/flux2/discussions" target="_blank">planning discussions</a>.</li>
<li>Follow
<a href="https://twitter.com/fluxcd" target="_blank">Flux on Twitter</a>, or join the
<a href="https://www.linkedin.com/groups/8985374/" target="_blank">Flux LinkedIn group</a>.</li>
</ul>
