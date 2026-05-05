---
title: "Blog: Bootstrapping Flux with Terraform, the right way"
url: "https://fluxcd.io/blog/2026/04/terraform-flux-operator-bootstrap/"
date: "Tue, 28 Apr 2026 09:00:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
<img height="360" src="https://fluxcd.io/blog/2026/04/terraform-flux-operator-bootstrap/featured-image_hud70d597f2de2ef143d3d0013909f7723_555346_640x0_resize_box_3.png" width="640" />
<p><img alt="" src="featured-image.png" /></p>
<p>This post introduces a new
<a href="https://github.com/controlplaneio-fluxcd/terraform-kubernetes-flux-operator-bootstrap" target="_blank">Terraform module</a>
(fully compatible with OpenTofu) that bootstraps
<a href="https://fluxoperator.dev" target="_blank">Flux Operator</a>
into a Kubernetes cluster and then steps aside, letting Flux do what Flux does best.</p>
<p>Here are some of the problems it sets out to fix.</p>
<h2 id="ownership-handoff">Ownership handoff</h2>
<p>Terraform is the natural place to install Flux right after a cluster comes up, since
credentials are in scope and providers are wired. But once Flux is online, every
object Terraform applied is now also an object Flux wants to reconcile. The traditional
workarounds (the
<a href="https://registry.terraform.io/providers/fluxcd/flux/latest" target="_blank"><code>fluxcd/flux</code></a>
provider, or chained <code>helm_release</code> resources) keep Terraform on the hook for
steady-state reconciliation forever.</p>
<p>This module takes a different approach. Terraform owns only the bootstrap mechanism: a
namespace, temporary RBAC, and a Kubernetes Job that applies Flux Operator
and the
<a href="https://fluxoperator.dev/docs/crd/fluxinstance/" target="_blank">FluxInstance</a>.
The module implements a <strong>create-if-missing</strong> strategy. Flux adopts the resources and
Terraform stops touching it. When inputs are unchanged, <code>terraform plan</code>
shows zero diff.</p>
<h2 id="using-one-gitops-repository">Using one GitOps repository</h2>
<p>The Terraform root module and the Flux manifests live side-by-side in the same repository,
so the bootstrap inputs and the steady-state desired state are versioned together:</p>
<div class="highlight"><pre tabindex="0"><code class="language-text"><span style="display: flex;"><span>repo/
</span></span><span style="display: flex;"><span>├── terraform/ # Terraform root module
</span></span><span style="display: flex;"><span>│ ├── main.tf
</span></span><span style="display: flex;"><span>│ ├── providers.tf
</span></span><span style="display: flex;"><span>│ └── variables.tf
</span></span><span style="display: flex;"><span>└── clusters/
</span></span><span style="display: flex;"><span> └── staging/ # reconciled by Flux via FluxInstance.spec.sync.path
</span></span><span style="display: flex;"><span> └── flux-system/
</span></span><span style="display: flex;"><span> ├── flux-instance.yaml # applied by the bootstrap Job
</span></span><span style="display: flex;"><span> ├── flux-operator-values.yaml # shared between Terraform and the Flux-managed HelmRelease
</span></span><span style="display: flex;"><span> ├── flux-operator.yaml # ResourceSet wrapping the Flux Operator HelmRelease
</span></span><span style="display: flex;"><span> ├── runtime-info.yaml # Git-managed fields of flux-runtime-info (optional)
</span></span><span style="display: flex;"><span> └── kustomization.yaml # configMapGenerator for flux-operator-values
</span></span></code></pre></div><p>The Terraform module loads the same <code>flux-instance.yaml</code> that Flux will reconcile after
bootstrap and provisions the Git pull secret it needs to keep syncing the repository:</p>
<div class="highlight"><pre tabindex="0"><code class="language-hcl"><span style="display: flex;"><span><span style="color: #007020; font-weight: bold;">module</span> <span style="color: #4070a0;">"flux_operator_bootstrap"</span> {
</span></span><span style="display: flex;"><span> source <span style="color: #666;">=</span> <span style="color: #4070a0;">"controlplaneio-fluxcd/flux-operator-bootstrap/kubernetes"</span>
</span></span><span style="display: flex;"><span> revision <span style="color: #666;">=</span> <span style="color: #40a070;">1</span>
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span> gitops_resources <span style="color: #666;">=</span> {
</span></span><span style="display: flex;"><span> instance_yaml <span style="color: #666;">=</span> <span style="color: #007020; font-weight: bold;">file</span>(<span style="color: #4070a0;">"${path.root}/../clusters/${var.cluster_name}/flux-system/flux-instance.yaml"</span>)
</span></span><span style="display: flex;"><span> }
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span> managed_resources <span style="color: #666;">=</span> {
</span></span><span style="display: flex;"><span> secrets_yaml <span style="color: #666;">=</span> <span>&lt;&lt;-</span><span style="color: #007020; font-weight: bold;">YAML</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">apiVersion</span><span>:</span> <span style="color: #007020; font-weight: bold;">v1</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">kind</span><span>:</span> <span style="color: #007020; font-weight: bold;">Secret</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">metadata</span><span>:</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">name</span><span>:</span> <span style="color: #007020; font-weight: bold;">flux</span><span>-</span><span style="color: #007020; font-weight: bold;">system</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">type</span><span>:</span> <span style="color: #007020; font-weight: bold;">Opaque</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">stringData</span><span>:</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">username</span><span>:</span> <span style="color: #007020; font-weight: bold;">git</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">password</span><span>:</span> <span>'</span><span style="color: #70a0d0;">${</span><span>var</span>.<span>git_token</span><span style="color: #70a0d0;">}</span><span>'</span>
</span></span><span style="display: flex;"><span> <span style="color: #007020; font-weight: bold;">YAML</span>
</span></span><span style="display: flex;"><span> }
</span></span><span style="display: flex;"><span>}
</span></span></code></pre></div><p>No secret material ever lands in the Terraform state file. The
module marks <code>managed_resources</code> as <code>sensitive</code> and only persists a SHA-256 hash to
detect changes, while still reconciling drift on every run with server-side apply - the
same model as kustomize-controller. Pull values from Vault, AWS Secrets Manager, or any
other store via <code>data</code> sources and compose them into <code>secrets_yaml</code>; the rendered YAML
never appears in state.</p>
<h2 id="no-two-phase-apply">No two-phase apply</h2>
<p>The module does not require cluster connectivity at plan time.
Because the configuration is static, it can live in the same Terraform root
module that creates the cluster.
Since the plan doesn&rsquo;t need runtime information, the operator bootstrap can directly <code>depends_on</code> the cluster module instance:</p>
<div class="highlight"><pre tabindex="0"><code class="language-hcl"><span style="display: flex;"><span>module "cluster" { source <span style="color: #666;">=</span> <span style="color: #4070a0;">"..."</span> }
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span><span style="color: #007020; font-weight: bold;">provider</span> <span style="color: #4070a0;">"helm"</span> {
</span></span><span style="display: flex;"><span> kubernetes <span style="color: #666;">=</span> {
</span></span><span style="display: flex;"><span> host <span style="color: #666;">=</span> <span style="color: #007020; font-weight: bold;">module</span>.<span style="color: #007020; font-weight: bold;">cluster</span>.<span style="color: #007020; font-weight: bold;">endpoint</span>
</span></span><span style="display: flex;"><span> cluster_ca_certificate <span style="color: #666;">=</span> <span style="color: #007020; font-weight: bold;">base64decode</span>(<span style="color: #007020; font-weight: bold;">module</span>.<span style="color: #007020; font-weight: bold;">cluster</span>.<span style="color: #007020; font-weight: bold;">ca_certificate</span>)
</span></span><span style="display: flex;"><span> token <span style="color: #666;">=</span> <span style="color: #007020; font-weight: bold;">module</span>.<span style="color: #007020; font-weight: bold;">cluster</span>.<span style="color: #007020; font-weight: bold;">token</span>
</span></span><span style="display: flex;"><span> }
</span></span><span style="display: flex;"><span>}
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span><span style="color: #007020; font-weight: bold;">module</span> <span style="color: #4070a0;">"flux_operator_bootstrap"</span> {
</span></span><span style="display: flex;"><span> depends_on <span style="color: #666;">=</span> [<span style="color: #007020; font-weight: bold;">module</span>.<span style="color: #007020; font-weight: bold;">cluster</span>]
</span></span><span style="display: flex;"><span> source <span style="color: #666;">=</span> <span style="color: #4070a0;">"controlplaneio-fluxcd/flux-operator-bootstrap/kubernetes"</span>
</span></span><span style="display: flex;"><span> revision <span style="color: #666;">=</span> <span style="color: #40a070;">1</span><span style="color: #60a0b0; font-style: italic;">
</span></span></span><span style="display: flex;"><span><span style="color: #60a0b0; font-style: italic;"> # ...
</span></span></span><span style="display: flex;"><span><span style="color: #60a0b0; font-style: italic;"></span>}
</span></span></code></pre></div><h2 id="fluxs-own-dependencies-cni-and-storage">Flux&rsquo;s own dependencies: CNI and Storage</h2>
<p>Some components have to exist before Flux can run (a self-managed CNI like Cilium is
a good example). Without a CNI, pods lack network access, and this includes the Flux
controllers themselves. The new Terraform module accepts an ordered list of prerequisite
Helm charts and manifests, which are applied in sequence by the bootstrap Job before
Flux Operator. For the CNI scenario, we let the Job run with <code>host_network: true</code>,
since pod networking is unavailable until after the CNI comes up:</p>
<div class="highlight"><pre tabindex="0"><code class="language-hcl"><span style="display: flex;"><span>job <span style="color: #666;">=</span> {
</span></span><span style="display: flex;"><span> host_network <span style="color: #666;">=</span> <span style="color: #902000;">true</span>
</span></span><span style="display: flex;"><span>}
</span></span><span style="display: flex;"><span>
</span></span><span style="display: flex;"><span>gitops_resources <span style="color: #666;">=</span> {
</span></span><span style="display: flex;"><span> instance_yaml <span style="color: #666;">=</span> <span style="color: #007020; font-weight: bold;">file</span>(<span style="color: #4070a0;">"${path.root}/../clusters/${var.cluster_name}/flux-system/flux-instance.yaml"</span>)
</span></span><span style="display: flex;"><span> prerequisites <span style="color: #666;">=</span> {
</span></span><span style="display: flex;"><span> charts <span style="color: #666;">=</span> [
</span></span><span style="display: flex;"><span> { name <span style="color: #666;">=</span> "cilium", repository <span style="color: #666;">=</span> "quay.io/cilium/charts/cilium", namespace <span style="color: #666;">=</span> <span style="color: #4070a0;">"kube-system"</span> },
</span></span><span style="display: flex;"><span> ]
</span></span><span style="display: flex;"><span> }
</span></span><span style="display: flex;"><span>}
</span></span></code></pre></div><p>This extends to any component your Flux install depends on.
The same mechanism can handle CSI drivers that the Flux controllers may need to mount before
they can start. This lays the groundwork for an upcoming SPIFFE/SPIRE integration that
we&rsquo;ll have more to share about in the next few releases.
Any of these components then become adopted by Flux for steady-state reconciliation,
following the same handoff described above that&rsquo;s used for the Flux Operator HelmRelease
and FluxInstance.</p>
<p>This module bootstraps Flux Operator without fighting Flux
for resource ownership. It keeps secrets out of the state file, runs in the same root module
as the cluster itself, and bootstraps platform prerequisites like CNI and CSI that Flux itself
depends on before handing management of those add-ons back to Flux.</p>
<h2 id="migrating">Migrating</h2>
<ul>
<li>From the
<a href="https://registry.terraform.io/providers/fluxcd/flux/latest" target="_blank"><code>fluxcd/flux</code></a> provider -
<a href="https://github.com/controlplaneio-fluxcd/terraform-kubernetes-flux-operator-bootstrap/blob/main/docs/migration-from-flux-provider.md" target="_blank">migration guide</a></li>
<li>From the previous flux-operator Terraform example -
<a href="https://github.com/controlplaneio-fluxcd/terraform-kubernetes-flux-operator-bootstrap/blob/main/docs/migration-from-previous-approach.md" target="_blank">migration guide</a></li>
<li>Minimal example -
<a href="https://github.com/controlplaneio-fluxcd/flux-operator/tree/main/config/terraform" target="_blank">flux-operator/config/terraform</a></li>
<li>Full reference setup -
<a href="https://github.com/controlplaneio-fluxcd/d2-fleet/tree/main/terraform" target="_blank">d2-fleet</a></li>
</ul>
