---
title: "Blog: GitHub App bootstrap with Flux Operator"
url: "https://fluxcd.io/blog/2025/04/flux-operator-github-app-bootstrap/"
date: "Mon, 14 Apr 2025 12:00:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
<img height="360" src="https://fluxcd.io/blog/2025/04/flux-operator-github-app-bootstrap/featured-image_hu758f39f2897d9b09922a59b22a7c36a6_607567_640x0_resize_box_3.png" width="640" />
<p><img alt="" src="featured-image.png" /></p>
<p>In this blog post, we will showcase how
<a href="https://github.com/controlplaneio-fluxcd/flux-operator" target="_blank">Flux Operator</a>
can be used to bootstrap Kubernetes clusters using the GitHub App authentication method
introduced in
<a href="https://fluxcd.io/blog/2025/02/flux-v2.5.0/" target="_blank">Flux 2.5.0</a>.</p>
<p>Prior to Flux 2.5.0, the GitHub repository authentication methods were based on using a
secret that is tied to a GitHub user, be it a personal access token (PAT) or an SSH deploy key.
When the user leaves the organization, the GitHub deploy keys are revoked
resulting in Flux losing access to all repositories. To restore access, the cluster
administrators have to generate new GitHub deploy keys tied to a different user
and rotate the secret in all clusters.</p>
<p>To avoid this situation, the recommendation was for organizations to create a dedicated
GitHub user for Flux, but this is also not ideal since an extra user affects billing.
The login credentials and MFA code have to be stored in an external secret management system
like 1Password, increasing the complexity of the cluster bootstrap process.</p>
<p>Starting with Flux 2.5.0, the GitHub App authentication method allows organizations to create a GitHub App
with access to the repositories from where Flux syncs the desired state of Kubernetes clusters.
Instead of using the credentials of a GitHub user, Flux running on the clusters will use the GitHub App
private key to authenticate with the GitHub API, acquiring a short-lived access token to perform
Git operations.</p>
<h2 id="flux-operator">Flux Operator</h2>
<p>The
<a href="https://github.com/controlplaneio-fluxcd/flux-operator" target="_blank">Flux Operator</a> offers an alternative
to the Flux CLI bootstrap procedure. It removes the operational burden of managing Flux across fleets
of clusters by fully automating the installation, configuration, and upgrade of the Flux controllers
based on a declarative API called <code>FluxInstance</code>.</p>
<p>The
<a href="https://fluxcd.control-plane.io/operator/fluxinstance/" target="_blank">FluxInstance</a> custom resource defines
the desired state of the Flux components and allows the configuration of the
<a href="https://fluxcd.control-plane.io/operator/flux-sync/" target="_blank">cluster state syncing</a>
from Git repositories, OCI artifacts and S3-compatible storage.</p>
<p>When using a GitHub repository as the source of truth, the Flux instance can be configured
to use the GitHub App authentication method by referencing a Kubernetes secret that contains
the app ID, the installation ID and the private key of the GitHub App.</p>
<p>What follows is a step-by-step guide on how to install the Flux Operator and bootstrap
a cluster using the GitHub App authentication.</p>
<h3 id="bootstrap-using-flux-operator-and-helm">Bootstrap using Flux Operator and Helm</h3>
<p>First, install the Flux Operator using the Helm chart:</p>
<div class="highlight"><pre tabindex="0"><code class="language-shell"><span style="display: flex;"><span>helm install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> --namespace flux-system <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> --create-namespace
</span></span></code></pre></div><p>Next, create a GitHub App secret using the <code>flux</code> CLI (see docs on how to create a GitHub App
<a href="#github-app-docs">here</a>):</p>
<div class="highlight"><pre tabindex="0"><code class="language-shell"><span style="display: flex;"><span>flux create secret githubapp flux-system <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> --app-id<span style="color: #666;">=</span><span style="color: #40a070;">1</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> --app-installation-id<span style="color: #666;">=</span><span style="color: #40a070;">2</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> --app-private-key<span style="color: #666;">=</span>./path/to/private-key-file.pem
</span></span></code></pre></div><p>Finally, bootstrap the cluster by creating a <code>FluxInstance</code> custom resource in the <code>flux-system</code> namespace:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>fluxcd.controlplane.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>FluxInstance<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>flux<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>flux-system<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">distribution</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">version</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"2.x"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">registry</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"ghcr.io/fluxcd"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">components</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- source-controller<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- kustomize-controller<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- helm-controller<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- notification-controller<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- image-reflector-controller<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- image-automation-controller<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">cluster</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>kubernetes<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">multitenant</span>:<span style="color: #bbb;"> </span><span style="color: #007020; font-weight: bold;">false</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">networkPolicy</span>:<span style="color: #bbb;"> </span><span style="color: #007020; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">domain</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"cluster.local"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sync</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>GitRepository<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">provider</span>:<span style="color: #bbb;"> </span>github<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">url</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"https://github.com/my-org/my-fleet.git"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">ref</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"refs/heads/main"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">path</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"clusters/my-cluster"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">pullSecret</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"flux-system"</span><span style="color: #bbb;">
</span></span></span></code></pre></div><p>When the <code>FluxInstance</code> is applied on the cluster, the operator will automatically deploy the Flux controllers
and configure them to sync the cluster state from the specified repository using GitHub App authentication.
Similarly to the Flux CLI bootstrap, the operator generates a Flux <code>GitRepository</code> and <code>Kustomization</code> named
<code>flux-system</code> that points to the <code>clusters/my-cluster</code> path inside the Git repository.</p>
<p>The Flux instance can be customized in various ways including multi-tenancy lockdown,
sharding, horizontal and vertical scaling, persistent storage, and fine-tuning
the Flux controllers with Kustomize patches.
For more information on the available options, please refer
to the
<a href="https://fluxcd.control-plane.io/operator/flux-config/" target="_blank">Flux Operator documentation</a>.</p>
<h3 id="bootstrap-using-flux-operator-and-terraform">Bootstrap using Flux Operator and Terraform</h3>
<p>Alternatively, you can use Terraform or OpenTofu to install the Flux Operator and
the <code>FluxInstance</code>. A Terraform example is available in the
<a href="https://github.com/controlplaneio-fluxcd/flux-operator/blob/main/config/terraform/README.md" target="_blank">Flux Operator repository</a>.</p>
<p>The command for applying this Terraform example with a GitHub App would be the following:</p>
<div class="highlight"><pre tabindex="0"><code class="language-shell"><span style="display: flex;"><span><span style="color: #007020;">export</span> <span style="color: #bb60d5;">GITHUB_APP_PEM</span><span style="color: #666;">=</span><span style="color: #4070a0;">`</span>cat path/to/app.private-key.pem<span style="color: #4070a0;">`</span>
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span>terraform apply <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> -var <span style="color: #bb60d5;">flux_version</span><span style="color: #666;">=</span><span style="color: #4070a0;">"2.x"</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> -var <span style="color: #bb60d5;">flux_registry</span><span style="color: #666;">=</span><span style="color: #4070a0;">"ghcr.io/fluxcd"</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> -var <span style="color: #bb60d5;">github_app_id</span><span style="color: #666;">=</span><span style="color: #4070a0;">"1"</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> -var <span style="color: #bb60d5;">github_app_installation_id</span><span style="color: #666;">=</span><span style="color: #4070a0;">"2"</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> -var <span style="color: #bb60d5;">github_app_pem</span><span style="color: #666;">=</span><span style="color: #4070a0;">"</span><span style="color: #bb60d5;">$GITHUB_APP_PEM</span><span style="color: #4070a0;">"</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> -var <span style="color: #bb60d5;">git_url</span><span style="color: #666;">=</span><span style="color: #4070a0;">"https://github.com/my-org/my-fleet.git"</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> -var <span style="color: #bb60d5;">git_ref</span><span style="color: #666;">=</span><span style="color: #4070a0;">"refs/heads/main"</span> <span style="color: #4070a0; font-weight: bold;">\
</span></span></span><span style="display: flex;"><span><span style="color: #4070a0; font-weight: bold;"></span> -var <span style="color: #bb60d5;">git_path</span><span style="color: #666;">=</span><span style="color: #4070a0;">"clusters/production"</span>
</span></span></code></pre></div><h3 id="github-app-docs">GitHub App Docs</h3>
<ul>
<li>
<a href="https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app" target="_blank">Registering a GitHub App</a></li>
<li>
<a href="https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/managing-private-keys-for-github-apps" target="_blank">Managing private keys for GitHub Apps</a></li>
<li>
<a href="https://docs.github.com/en/apps/using-github-apps/installing-your-own-github-app" target="_blank">Installing your GitHub App</a></li>
</ul>
<p>After installing your GitHub App in your organization you can find the <em>installation ID</em> like this:</p>
<ol>
<li>Go to the Organization settings</li>
<li>Click on &lsquo;GitHub Apps&rsquo; under &lsquo;Third-party Access&rsquo;</li>
<li>If there are multiple GitHub apps, choose your App and click on &lsquo;Configure&rsquo;</li>
<li>Once your GitHub App is selected check the URL for obtaining &lsquo;GitHub App Installation ID&rsquo;</li>
</ol>
<p>The URL looks like this:</p>
<pre tabindex="0"><code>https://github.com/organizations/&lt;Organization-name&gt;/settings/installations/&lt;ID&gt;
</code></pre><h2 id="conclusion">Conclusion</h2>
<p>Using the GitHub App authentication method with Flux Operator offers a more secure
way of bootstrapping Flux on Kubernetes clusters, as it eliminates the need for managing
GitHub user credentials and deploy keys. This approach ensures that Flux can
continue to operate seamlessly even when users leave the organization or change their access
permissions.</p>
<p>Migrating clusters that have been bootstrapped with the Flux CLI to the Flux Operator
is a straightforward process. For more information on how to do this, please refer to the
<a href="https://fluxcd.control-plane.io/operator/flux-bootstrap-migration/" target="_blank">Flux Operator bootstrap migration guide</a>.</p>
