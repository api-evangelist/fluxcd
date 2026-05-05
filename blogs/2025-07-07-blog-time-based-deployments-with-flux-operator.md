---
title: "Blog: Time-based deployments with Flux Operator"
url: "https://fluxcd.io/blog/2025/07/time-based-deployments/"
date: "Mon, 07 Jul 2025 12:00:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
<img height="360" src="https://fluxcd.io/blog/2025/07/time-based-deployments/featured-image_hu4d2f40587f850d3f31bb1c5a3038f0ad_154744_640x0_resize_box_3.png" width="640" />
<p>We are thrilled to announce time-based deployments, a feature long-awaited by Flux users, in
<a href="https://github.com/controlplaneio-fluxcd/flux-operator/releases/tag/v0.23.0" target="_blank">Flux Operator v0.23.0</a>!</p>
<p><img alt="" src="featured-image.png" /></p>
<p>Organizations using Flux for GitOps deployments frequently require sophisticated control over when
changes are applied to production systems, particularly in regulated industries or critical business
environments. Key requirements include adhering to Change Advisory Board (CAB) approval windows,
enforcing &ldquo;No Deploy Fridays&rdquo; policies, and restricting deployments during peak business hours to
ensure service stability.</p>
<p>Maintenance windows become critical when managing helm upgrades, where teams need to skip reconciliation
unless the current time falls within a specified interval. In regulated environments like medical device
companies, automated deployments must be controlled to prevent unexpected disruptions during critical
operational periods. Large telecommunications providers and ISVs managing multiple client clusters need
gating mechanisms to control application rollouts, allowing tenants to consume platform updates when ready.</p>
<p>In this post, we show how to use time-based deployment with Flux Operator
<a href="https://fluxcd.control-plane.io/operator/resourcesets/introduction/" target="_blank"><code>ResourceSets</code></a>.</p>
<h2 id="how-it-works">How it works</h2>
<p>The Flux Operator ResourceSet API allows defining bundles of Flux objects by
templating a set of resources with inputs provided by the ResourceSetInputProvider API.</p>
<p>The ResourceSetInputProvider API allows pulling inputs from external sources, such as
GitHub pull requests, branches and tags. For example, on the reconciliation of a
ResourceSetInputProvider of type <code>GitHubTag</code>, the operator will list the tags of
a GitHub repository, filter them according to a semver range, and export a set of
inputs for each matching tag in the ResourceSetInputProvider <code>.status.exportedInputs</code>
field. For example:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">status</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">exportedInputs</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">id</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"48955639"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">tag</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"6.0.4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sha</span>:<span style="color: #bbb;"> </span>11cf36d83818e64aaa60d523ab6438258ebb6009<span style="color: #bbb;">
</span></span></span></code></pre></div><p>Starting with Flux Operator v0.23.0, the ResourceSetInputProvider API now has the field
<a href="https://fluxcd.control-plane.io/operator/resourcesetinputprovider/#schedule" target="_blank"><code>.spec.schedule</code></a>,
which allows defining a cron-based schedule for the reconciliation of the ResourceSetInputProvider.
For example:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">schedule</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># Every day-of-week from Monday through Thursday</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># between 10:00 to 16:00</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">cron</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"0 10 * * 1-4"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">timeZone</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"Europe/London"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">window</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"6h"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># Every Friday from 10:00 to 13:00</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">cron</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"0 10 * * 5"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">timeZone</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"Europe/London"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">window</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"3h"</span><span style="color: #bbb;">
</span></span></span></code></pre></div><p>With this configuration, reconciliations of the ResourceSetInputProvider object
would only be allowed to run within the specified time windows. When the window
is active, the reconciliation happens normally, according to the interval defined
in the <code>fluxcd.controlplane.io/reconcileEvery</code> annotation.</p>
<h2 id="a-complete-example">A complete example</h2>
<ul>
<li><strong>Define a ResourceSetInputProvider</strong>: This provider will scan a Git branch or tag
for changes and export the commit SHA as an input.</li>
<li><strong>Configure schedule</strong>: The provider will have a reconciliation schedule
that defines when it should check for changes in the Git repository.</li>
<li><strong>Define a ResourceSet</strong>: The ResourceSet will use the inputs from the provider
to create a <code>GitRepository</code> and <code>Kustomization</code> that deploys the application
at the specified commit SHA.</li>
</ul>
<h3 id="resourcesetinputprovider-definition">ResourceSetInputProvider Definition</h3>
<p>Assuming the Kubernetes deployment manifests for an application are stored in a Git repository,
you can define a input provider that scans a branch for changes
and exports the commit SHA:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>fluxcd.controlplane.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ResourceSetInputProvider<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>my-app-main<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">labels</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">app.kubernetes.io/name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">annotations</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">fluxcd.controlplane.io/reconcileEvery</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"10m"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">fluxcd.controlplane.io/reconcileTimeout</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"1m"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">schedule</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">cron</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"0 8 * * 1-5"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">timeZone</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"Europe/London"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">window</span>:<span style="color: #bbb;"> </span>8h<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>GitHubBranch<span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># or GitLabBranch</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">url</span>:<span style="color: #bbb;"> </span>https://github.com/my-org/my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">secretRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>gh-app-auth<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">filter</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">includeBranch</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"^main$"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">defaultValues</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">env</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"production"</span><span style="color: #bbb;">
</span></span></span></code></pre></div><p>For when Git tags are used to version the application, you can define an input provider
that scans the Git tags and exports the latest tag according to a semantic versioning:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>fluxcd.controlplane.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ResourceSetInputProvider<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>my-app-release<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">labels</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">app.kubernetes.io/name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">annotations</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">fluxcd.controlplane.io/reconcileEvery</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"10m"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">fluxcd.controlplane.io/reconcileTimeout</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"1m"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">schedule</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">cron</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"0 8 * * 1-5"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">timeZone</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"Europe/London"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">window</span>:<span style="color: #bbb;"> </span>8h<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>GitHubTag<span style="color: #bbb;"> </span><span style="color: #60a0b0; font-style: italic;"># or GitLabTag</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">url</span>:<span style="color: #bbb;"> </span>https://github.com/my-org/my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">secretRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>gh-auth<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">filter</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">semver</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"&gt;=1.0.0"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">limit</span>:<span style="color: #bbb;"> </span><span style="color: #40a070;">1</span><span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="resourceset-definition">ResourceSet Definition</h3>
<p>The exported inputs can then be used in a <code>ResourceSet</code> to deploy the application
using the commit SHA from the input provider:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>fluxcd.controlplane.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ResourceSet<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">inputsFrom</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ResourceSetInputProvider<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">selector</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">matchLabels</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">app.kubernetes.io/name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">resources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>source.toolkit.fluxcd.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>GitRepository<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>&lt;&lt; inputs.provider.namespace &gt;&gt;<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">interval</span>:<span style="color: #bbb;"> </span>12h<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">url</span>:<span style="color: #bbb;"> </span>https://github.com/my-org/my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">ref</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">commit</span>:<span style="color: #bbb;"> </span>&lt;&lt; inputs.sha &gt;&gt;<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">secretRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>gh-auth<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sparseCheckout</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- deploy<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>kustomize.toolkit.fluxcd.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Kustomization<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>&lt;&lt; inputs.provider.namespace &gt;&gt;<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">interval</span>:<span style="color: #bbb;"> </span>30m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">retryInterval</span>:<span style="color: #bbb;"> </span>5m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">prune</span>:<span style="color: #bbb;"> </span><span style="color: #007020; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">wait</span>:<span style="color: #bbb;"> </span><span style="color: #007020; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">timeout</span>:<span style="color: #bbb;"> </span>5m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sourceRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>GitRepository<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>my-app<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">path</span>:<span style="color: #bbb;"> </span>deploy/&lt;&lt; inputs.env &gt;&gt;<span style="color: #bbb;">
</span></span></span></code></pre></div><p>When the <code>ResourceSetInputProvider</code> runs according to its schedule, if it finds a new commit,
the <code>ResourceSet</code> will be automatically updated with the new commit SHA which will trigger
an application deployment for the new version.</p>
<h2 id="further-reading">Further reading</h2>
<ul>
<li>
<a href="https://fluxcd.control-plane.io/operator/resourcesets/time-based-delivery/" target="_blank">Complete Guide</a></li>
<li>
<a href="https://fluxcd.control-plane.io/operator/resourcesets/introduction/" target="_blank">ResourceSets Introduction</a></li>
<li>
<a href="https://fluxcd.control-plane.io/operator/resourceset/" target="_blank">ResourceSets Documentation</a></li>
<li>
<a href="https://fluxcd.control-plane.io/operator/resourcesetinputprovider/#schedule" target="_blank">Schedule Documentation</a></li>
<li>
<a href="https://fluxcd.control-plane.io/operator/resourcesetinputprovider/#schedule-status" target="_blank">Schedule Status Documentation</a></li>
</ul>
