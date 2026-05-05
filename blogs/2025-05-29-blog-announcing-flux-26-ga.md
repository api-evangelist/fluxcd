---
title: "Blog: Announcing Flux 2.6 GA"
url: "https://fluxcd.io/blog/2025/05/flux-v2.6.0/"
date: "Thu, 29 May 2025 12:00:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
<img height="360" src="https://fluxcd.io/blog/2025/05/flux-v2.6.0/featured-image_hu0a3d34a1286ca95e0c20a1ae8ebfb057_608764_640x0_resize_box_3.png" width="640" />
<p>We are thrilled to announce the release of
<a href="https://github.com/fluxcd/flux2/releases/tag/v2.6.0" target="_blank">Flux v2.6.0</a>!
In this post, we will highlight some of the new features and improvements included in this release.</p>
<p><img alt="" src="featured-image.png" /></p>
<h2 id="highlights">Highlights</h2>
<p>Flux v2.6 marks the General Availability (GA) of the Flux Open Container Initiative (OCI) Artifacts features.
The OCI artifacts support was first introduced in 2022, and since then we&rsquo;ve been evolving Flux towards
a <strong>Gitless GitOps</strong> model. In this model, the Flux controllers are fully decoupled from Git,
relying solely on container registries as the source of truth for the desired state of Kubernetes clusters.</p>
<p>In the last couple of years, the OCI feature-set has matured, and we&rsquo;ve seen major financial institutions
and enterprises adopting Flux and OCI as their preferred way of managing production deployments.
To see it in action, you can check the reference architecture guide made by ControlPlane
on how highly regulated industries can securely implement
<a href="https://control-plane.io/posts/d2-reference-architecture-guide/" target="_blank">Gitless GitOps with Flux and OCI</a>.</p>
<p>In this release, we have also introduced several new features to the Flux controllers,
including digest pinning in image automation, object-level workload identity for container registries
and KMS services authentication, and various improvements to notifications.</p>
<p>In ecosystem news, there is a new release of
<a href="https://github.com/controlplaneio-fluxcd/flux-operator" target="_blank">Flux Operator</a>
that comes with a Model Context Protocol (MCP) implementation for allowing AI assistants to interact with Flux.
For more details on the Flux MCP Server, see the
<a href="https://fluxcd.io/blog/2025/05/ai-assisted-gitops/" target="_blank">AI-Assisted GitOps blog post</a>.</p>
<h2 id="general-availability-of-flux-oci-artifacts">General availability of Flux OCI Artifacts</h2>
<p>This release marks the General Availability (GA) of Flux
<a href="https://fluxcd.io/flux/components/source/ocirepositories/">OCIRepository</a>
API, which allows storing the desired state of Kubernetes clusters in OCI container registries.</p>
<p>The <code>OCIRepository</code> v1 API comes with new features including:</p>
<ul>
<li>Support for
<a href="#object-level-workload-identity">Object-Level Workload Identity</a>,
which allows Flux to use different cloud identities for accessing container registries on multi-tenant clusters.</li>
<li>Caching of registry credentials for cloud providers, which allows Flux to reuse the OIDC tokens
for subsequent requests to the same registry, reducing the number of authentication requests.</li>
</ul>
<p>The <code>OCIRepository</code> v1 API is backward compatible with the previous v1beta2 API, users can upgrade
by changing the <code>apiVersion</code> in the YAML files that contain <code>OCIRepository</code> definitions from
<code>source.toolkit.fluxcd.io/v1beta2</code> to <code>source.toolkit.fluxcd.io/v1</code>.</p>
<p>The Flux CLI commands for working with OCI artifacts have been promoted to stable:</p>
<ul>
<li><code>flux build artifact</code></li>
<li><code>flux push artifact</code></li>
<li><code>flux pull artifact</code></li>
<li><code>flux tag artifact</code></li>
<li><code>flux diff artifact</code></li>
<li><code>flux list artifacts</code></li>
</ul>
<p>The Flux custom media types used for OCI artifacts produced by the Flux CLI are now stable:</p>
<ul>
<li>config media type <code>application/vnd.cncf.flux.config.v1+json</code></li>
<li>content media type <code>application/vnd.cncf.flux.content.v1.tar+gzip</code></li>
</ul>
<h3 id="breaking-changes">Breaking changes</h3>
<p>Prior to <code>v2.6.0</code>, the <code>OCIRepository</code> and <code>ImageRepository</code> APIs allowed the <code>spec.provider</code> field
to be set to a value that did not necessarily match the repository URL. In these cases the controllers
would simply ignore the <code>spec.provider</code>, not configuring OIDC authentication for the repository.</p>
<p>For example, the repository <code>public.ecr.aws/aws-controllers-k8s</code> never matched Flux&rsquo;s regular expression
for the <code>aws</code> provider, but the controller would still allow the <code>spec.provider</code> to be set to <code>aws</code> in
this case and would simply ignore it. This specific configuration would work correctly because this
particular repository is public and does not require authentication.</p>
<p>Similarly, a private repository that did not match any of Flux&rsquo;s validations for the three container
registry providers (<code>aws</code>, <code>azure</code>, <code>gcp</code>) would also work with the <code>spec.provider</code> set to one of
these values, as long as it was also configured with one of the <code>spec.secretRef</code> or
<code>spec.serviceAccountName</code> fields for using image pull secrets. In these cases, the controller
would simply ignore the <code>spec.provider</code> and use the image pull secret instead.</p>
<p>Starting with <code>v2.6.0</code>, Flux is fixing this behavior. The repository URL must now match the provider
set in <code>spec.provider</code>, otherwise the controller will reject the configuration and return an error.
For automatic OIDC authentication, the <code>spec.provider</code> must be set to one of the three container
registry providers (<code>aws</code>, <code>azure</code>, <code>gcp</code>). For public repositories or authentication using image
pull secrets, the <code>spec.provider</code> must not be set, or set to <code>generic</code>. These configuration
instructions were explicit in the Flux docs since many releases, but are only now in <code>v2.6.0</code>
being strictly enforced by the controllers.</p>
<h2 id="image-automation-digest-pinning">Image Automation Digest Pinning</h2>
<p>In Flux v2.6, the image automation has been enhanced to support digest pinning
for container images. This feature allows users to configure the <code>ImagePolicy</code>
to track the latest digest of a container image, and the <code>ImageUpdateAutomation</code>
to update the manifests in the Git repository with the new digest.</p>
<p>The <code>ImagePolicy</code> can now be configured to select the latest image digest
with <code>.spec.digestReflectionPolicy</code> set to <code>Always</code>.
Once a policy is set to track the latest digest, the manifests in the Git repository
will be updated with digest references in the format <code>&lt;registry&gt;/&lt;name&gt;:&lt;tag&gt;@&lt;digest&gt;</code>.</p>
<p>A new marker has been introduced to allow setting the digest in custom resources
where <code>repository</code>, <code>tag</code> and <code>digest</code> are separate values:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>helm.toolkit.fluxcd.io/v2<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>HelmRelease<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">values</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">image</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">repository</span>:<span style="color: #bbb;"> </span>docker.io/my-org/my-app<span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># {"$imagepolicy": "flux-system:my-app:name"}</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">tag</span>:<span style="color: #bbb;"> </span>latest <span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># {"$imagepolicy": "flux-system:my-app:tag"}</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">digest</span>:<span style="color: #bbb;"> </span>sha256:ec0119...<span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># {"$imagepolicy": "flux-system:my-app:digest"}</span><span style="color: #bbb;">
</span></span></span></code></pre></div><p>For more details on how to configure image automation digest pinning,
see the following
<a href="https://fluxcd.io/flux/guides/image-update/#digest-pinning">guide</a>.</p>
<h2 id="object-level-workload-identity">Object-level Workload Identity</h2>
<p>Starting with Flux v2.6, you can configure workload identity at the object level
in the <code>Kustomization</code> API for SOPS decryption with KMS services, and in the
<code>OCIRepository</code> and <code>ImageRepository</code> APIs for accessing container registries.</p>
<p>This feature allows cluster admins to use different cloud identities on multi-tenant
clusters. Instead of relying on static Secrets that require manual rotation,
you can now assign cloud identities per tenant by leveraging Kubernetes Workload Identity.</p>
<p>To use this feature, cluster admins have to enable the feature gate
<code>ObjectLevelWorkloadIdentity</code> which is opt-in from Flux v2.6.</p>
<p>For more details on how to configure object-level workload identity for Flux,
see the following docs:</p>
<ul>
<li>
<a href="https://fluxcd.io/flux/integrations/aws/">AWS workload identity</a></li>
<li>
<a href="https://fluxcd.io/flux/integrations/azure/">Azure workload identity</a></li>
<li>
<a href="https://fluxcd.io/flux/integrations/gcp/">GCP workload identity</a></li>
</ul>
<h2 id="github-app-authentication">GitHub App Authentication</h2>
<p>In Flux v2.6, we have completed the integration of GitHub App authentication for Git repositories.
This feature was introduced in
<a href="https://fluxcd.io/blog/2025/02/flux-v2.5.0/#github-app-authentication-for-git-repositories" target="_blank">Flux v2.5</a>,
and it is now fully supported across all Flux APIs.</p>
<p>The GitHub App authentication tokens are now cached by the Flux controllers
and reused for subsequent requests for the duration of the token lifetime.</p>
<p>The notification-controller has also been updated to support GitHub App authentication
when updating
<a href="https://fluxcd.io/flux/components/notification/providers/#git-commit-status-updates">Git commit statuses</a>
and for triggering
<a href="flux/components/notification/providers/#github-dispatch">GitHub Actions workflows</a>.</p>
<h2 id="notifications-improvements">Notifications Improvements</h2>
<p>Starting with Flux v2.6, users can customize the
<a href="https://fluxcd.io/flux/components/notification/providers/#git-commit-status-updates">Git commit status</a>
identifier in the notifications sent to Git providers by using Common Expression Language (CEL) expressions.</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>notification.toolkit.fluxcd.io/v1beta3<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Provider<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>github-status<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>flux-system<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>github<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">address</span>:<span style="color: #bbb;"> </span>https://github.com/my-gh-org/my-gh-repo<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">secretRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>github-app-auth<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">commitStatusExpr</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"(event.involvedObject.kind + '/' + event.involvedObject.name + '/' + event.metadata.clusterName)"</span><span style="color: #bbb;">
</span></span></span></code></pre></div><p>Customizing the commit status ID is particularly useful when using a monorepo for a fleet of Kubernetes clusters,
as it allows you to differentiate the commit statuses for each cluster.</p>
<p>Other improvements include:</p>
<ul>
<li>The notification-controller can now use Azure Workload Identity when sending notifications to Azure Event Hub.</li>
<li>The <code>github</code> and <code>githubdispatch</code> providers now support authenticating with a GitHub App.</li>
</ul>
<h2 id="controller-improvements">Controller Improvements</h2>
<ul>
<li>The <code>GitRepository</code> v1 API now supports sparse checkout by setting a list of directories in the <code>.spec.sparseCheckout</code> field.
This allows for optimizing the amount of data fetched from the Git repository.</li>
<li>The <code>GitRepository</code> v1 API gains supports mTLS authentication for HTTPS Git repositories.</li>
<li>The <code>Kustomization</code> v1 API now supports the value <code>WaitForTermination</code> for the <code>.spec.deletionPolicy</code> field.
This instructs the controller to wait for the deletion of all resources managed by the Kustomization
before allowing the Kustomization itself to be deleted.</li>
<li>The helm-controller v1.3.0 comes with a new feature gate called <code>DisableChartDigestTracking</code>,
which allows disabling appending the digest of OCI Helm charts to the chart version.
This is useful for charts that do not follow Helm&rsquo;s recommendation of using the app version
instead of the chart version as a label in the manifests.</li>
</ul>
<h2 id="supported-versions">Supported Versions</h2>
<p>Flux v2.3 has reached end-of-life and is no longer supported.</p>
<p>Flux v2.6 supports the following Kubernetes versions:</p>
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
<td style="text-align: left;">1.31, 1.32, 1.33</td>
</tr>
<tr>
<td style="text-align: left;">OpenShift</td>
<td style="text-align: left;">4.18</td>
</tr>
</tbody>
</table>
<blockquote>
<p><strong>Enterprise support</strong> Note that the CNCF Flux project offers support only for the latest three minor versions of Kubernetes.
Backwards compatibility with older versions of Kubernetes and OpenShift is offered by vendors such as
<a href="https://control-plane.io/enterprise-for-flux-cd/" target="_blank">ControlPlane</a> that provide enterprise support for Flux.</p>
</blockquote>
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
