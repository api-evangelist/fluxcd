---
title: "Blog: Announcing Flux 2.7 GA"
url: "https://fluxcd.io/blog/2025/09/flux-v2.7.0/"
date: "Tue, 30 Sep 2025 06:00:00 +0000"
author: ""
feed_url: "https://fluxcd.io/blog/index.xml"
---
<img height="360" src="https://fluxcd.io/blog/2025/09/flux-v2.7.0/featured-image_hu0a3d34a1286ca95e0c20a1ae8ebfb057_608626_640x0_resize_box_3.png" width="640" />
<p>We are thrilled to announce the release of
<a href="https://github.com/fluxcd/flux2/releases/tag/v2.7.0" target="_blank">Flux v2.7.0</a>!
In this post, we highlight some of the new features and improvements included in this release.</p>
<p><img alt="" src="featured-image.png" /></p>
<h2 id="highlights">Highlights</h2>
<p>Flux v2.7 marks the General Availability (GA) of the image update automation features
and comes with new APIs <code>ExternalArtifact</code> and <code>ArtifactGenerator</code>
for advanced source
<a href="#artifact-generators">composition and decomposition</a> patterns.</p>
<p>In this release, we have also introduced several new features to the Flux controllers,
including watching for changes in ConfigMaps and Secrets references,
extended readiness evaluation of dependencies with CEL expressions,
and support for OpenTelemetry tracing for Flux <code>Kustomization</code> and <code>HelmRelease</code> reconciliation.</p>
<p>In ecosystem news, there is a new release of
<a href="https://github.com/controlplaneio-fluxcd/flux-operator" target="_blank">Flux Operator</a>
that comes with
<a href="https://fluxcd.control-plane.io/operator/resourcesets/image-automation/" target="_blank">in-cluster image update automation</a>
features, that can be used for GitLess GitOps workflows.</p>
<h2 id="general-availability-of-image-update-automation">General availability of Image Update Automation</h2>
<p>This release marks the General Availability (GA) of Flux
<a href="https://fluxcd.io/flux/components/image/">Image Automation</a>
APIs and controllers. The image-reflector-controller and image-automation-controller work together to
update Kubernetes manifests in Git repositories when new container images are available in
container registries.</p>
<p>The following APIs have been promoted to stable v1:</p>
<ul>
<li>
<a href="https://fluxcd.io/flux/components/image/imagerepositories/">ImageRepository</a></li>
<li>
<a href="https://fluxcd.io/flux/components/image/imagepolicies/">ImagePolicy</a></li>
<li>
<a href="https://fluxcd.io/flux/components/image/imageupdateautomations/">ImageUpdateAutomation</a></li>
</ul>
<p>The <code>ImagePolicy</code> API now supports the <code>.spec.suspend</code> field to pause and resume the policy evaluation.</p>
<p>The <code>ImageUpdateAutomation</code> API gains support for Git sparse checkout. To enable this optimization,
the image-automation-controller can be configured with the <code>--feature-gates=GitSparseCheckout=true</code> flag.</p>
<p>In addition, the image-automation-controller can now be configured to use Kubernetes Workload Identity
for authenticating with AzureDevOps repositories.</p>
<p><strong>Breaking changes</strong>:</p>
<ul>
<li>The image-reflector-controller <code>autologin</code> flags which were deprecated since 2023 are now removed.
Users should set <code>ImageRepository.spec.provider</code> to the appropriate cloud provider for their container registry.</li>
<li>The <code>ImageUpdateAutomation</code> commit template fields <code>.Updated</code> and <code>.Changed.ImageResult</code> which were deprecated since 2024 are now removed.
Users should migrate to:
<ul>
<li><code>.Changed.FileChanges</code> for detailed change tracking</li>
<li><code>.Changed.Objects</code> for object-level changes</li>
<li><code>.Changed.Changes</code> for a flat list of changes</li>
</ul>
</li>
</ul>
<h2 id="watching-for-changes-in-configmaps-and-secrets">Watching for changes in ConfigMaps and Secrets</h2>
<p>Starting with Flux v2.7, the <code>kustomize-controller</code>, <code>helm-controller</code> and <code>notification-controller</code>
gain support for reacting to changes in ConfigMaps and Secrets references.</p>
<p>The following references are now watched for changes:</p>
<ul>
<li><code>Kustomization.spec.postBuild.substituteFrom</code></li>
<li><code>Kustomization.spec.decryption.secretRef</code></li>
<li><code>Kustomization.spec.kubeConfig.secretRef</code></li>
<li><code>Kustomization.spec.kubeConfig.configMapRef</code></li>
<li><code>HelmRelease.spec.valuesFrom</code></li>
<li><code>HelmRelease.spec.kubeConfig.secretRef</code></li>
<li><code>HelmRelease.spec.kubeConfig.configMapRef</code></li>
<li><code>Receiver.spec.secretRef</code></li>
</ul>
<p>When a referenced ConfigMap or Secret changes, the controller will immediately trigger a reconciliation
if the referenced object is labelled with <code>reconcile.fluxcd.io/watch: Enabled</code>.</p>
<p>To enable the watching of all referenced objects without the need to label them,
the controllers can be configured with the <code>--watch-configs-label-selector=owner!=helm</code> flag.</p>
<h2 id="workload-identity-authentication-for-remote-clusters">Workload Identity Authentication for Remote Clusters</h2>
<p>Starting with Flux v2.7, you can configure workload identity at the object level
in the <code>Kustomization</code> and <code>HelmRelease</code> resources to authenticate with cloud providers
when running Flux in the hub-and-spoke model.</p>
<p>This feature allows cluster admins to use cloud identities on the hub cluster to
configure Flux authentication to spoke clusters, without the need to create and manage
static <code>kubeconfig</code> Secrets.</p>
<p>For more details on how to configure secret-less authentication to remote clusters,
please refer to the following guides:</p>
<ul>
<li>
<a href="https://fluxcd.io/flux/components/kustomize/kustomizations/#secret-less-authentication">Kustomization - remote cluster apply</a></li>
<li>
<a href="https://fluxcd.io/flux/components/helm/helmreleases/#secret-less-authentication">HelmRelease - remote cluster apply</a></li>
</ul>
<h2 id="object-level-workload-identity">Object-level Workload Identity</h2>
<p>In Flux v2.7, we have completed the integration of Kubernetes Workload Identity
at the object level for all Flux APIs that support authentication with cloud providers.</p>
<p>This includes the following resources:</p>
<ul>
<li><code>Bucket.spec.serviceAccountName</code> for authenticating with AWS S3, Azure Blob Storage and Google Cloud Storage.</li>
<li><code>GitRepository.spec.serviceAccountName</code> for authenticating with Azure DevOps.</li>
<li><code>OCIRepository.spec.serviceAccountName</code> for authenticating with AWS ECR, Azure Container Registry and Google Artifact Registry.</li>
<li><code>ImageRepository.spec.serviceAccountName</code> for authenticating with AWS ECR, Azure Container Registry and Google Artifact Registry.</li>
<li><code>Kustomization.spec.decryption.serviceAccountName</code> for authenticating with AWS KMS, Azure Key Vault and Google KMS.</li>
<li><code>Kustomization.spec.kubeConfig.configMapRef.name</code> for authenticating with remote clusters on AWS EKS, Azure AKS and Google GKE.</li>
<li><code>HelmRelease.spec.kubeConfig.configMapRef.name</code> for authenticating with remote clusters on AWS EKS, Azure AKS and Google GKE.</li>
<li><code>Provider.spec.serviceAccountName</code> for authenticating with Azure DevOps, Azure Event Hub and Google Pub/Sub.</li>
</ul>
<p>For more details on how to configure object-level workload identity for Flux, see the following docs:</p>
<ul>
<li>
<a href="https://fluxcd.io/flux/integrations/aws/">AWS workload identity</a></li>
<li>
<a href="https://fluxcd.io/flux/integrations/azure/">Azure workload identity</a></li>
<li>
<a href="https://fluxcd.io/flux/integrations/gcp/">GCP workload identity</a></li>
</ul>
<h2 id="opentelemetry-tracing">OpenTelemetry Tracing</h2>
<p>Starting with Flux v2.7, users can enable OpenTelemetry tracing for Flux reconciliations
by configuring a Provider of type <code>otel</code>:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>notification.toolkit.fluxcd.io/v1beta3<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Provider<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>jaeger<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>flux-system<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">type</span>:<span style="color: #bbb;"> </span>otel<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">address</span>:<span style="color: #bbb;"> </span>http://jaeger-collector.jaeger:4318/v1/traces<span style="color: #bbb;">
</span></span></span></code></pre></div><p>The notification-controller converts Flux events into OTEL spans with proper trace relationships
based on the Flux object hierarchy. Source objects (<code>GitRepository</code>, <code>HelmChart</code>, <code>OCIRepository</code>, <code>Bucket</code>)
create root spans, while <code>Kustomization</code> and <code>HelmRelease</code> objects create child spans within the same trace.
Each span includes event metadata as attributes and uses the alert name and namespace as the service identifier.</p>






<div class="gallery-wrapper" id="gallery-1fba03f4fc4c04a0a3c39bc0fd775312-0-wrapper">
<div class="justified-gallery" id="gallery-1fba03f4fc4c04a0a3c39bc0fd775312-0">
<div>
<a class="galleryImg" href="https://fluxcd.io/blog/2025/09/flux-v2.7.0/images/flux-helm-otel-trace.png">
<img class="lazy" height="458" src="data:image/jpeg;base64,/9j/2wCEAAoHBwgHBgoICAgLCgoLDhgQDg0NDh0VFhEYIx8lJCIfIiEmKzcvJik0KSEiMEExNDk7Pj4&#43;JS5ESUM8SDc9PjsBCgsLDg0OHBAQHDsoIig7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7O//AABEIABAAIAMBIgACEQEDEQH/xAGiAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgsQAAIBAwMCBAMFBQQEAAABfQECAwAEEQUSITFBBhNRYQcicRQygZGhCCNCscEVUtHwJDNicoIJChYXGBkaJSYnKCkqNDU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6g4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2drh4uPk5ebn6Onq8fLz9PX29/j5&#43;gEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoLEQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4&#43;Tl5ufo6ery8/T19vf4&#43;fr/2gAMAwEAAhEDEQA/APXREB7/APARSfMrYDYH&#43;7Thn3qJ2Il61CepTJ8E/wAf6UhRj/H&#43;lRhyf4qa7sD944qyT//Z" width="900" />
</a>
</div>
<div>
<a class="galleryImg" href="https://fluxcd.io/blog/2025/09/flux-v2.7.0/images/flux-git-otel-trace.png">
<img class="lazy" height="398" src="data:image/jpeg;base64,/9j/2wCEAAoHBwgHBgoICAgLCgoLDhgQDg0NDh0VFhEYIx8lJCIfIiEmKzcvJik0KSEiMEExNDk7Pj4&#43;JS5ESUM8SDc9PjsBCgsLDg0OHBAQHDsoIig7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7Ozs7O//AABEIAA4AIAMBIgACEQEDEQH/xAGiAAABBQEBAQEBAQAAAAAAAAAAAQIDBAUGBwgJCgsQAAIBAwMCBAMFBQQEAAABfQECAwAEEQUSITFBBhNRYQcicRQygZGhCCNCscEVUtHwJDNicoIJChYXGBkaJSYnKCkqNDU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6g4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2drh4uPk5ebn6Onq8fLz9PX29/j5&#43;gEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoLEQACAQIEBAMEBwUEBAABAncAAQIDEQQFITEGEkFRB2FxEyIygQgUQpGhscEJIzNS8BVictEKFiQ04SXxFxgZGiYnKCkqNTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqCg4SFhoeIiYqSk5SVlpeYmZqio6Slpqeoqaqys7S1tre4ubrCw8TFxsfIycrS09TV1tfY2dri4&#43;Tl5ufo6ery8/T19vf4&#43;fr/2gAMAwEAAhEDEQA/APUwAOvPvT0Rm6Sso9BmmD3o3SBvlOB/n2q2c8Hd6lj7O3/PdqBAwzmUn6io1eUj73b1/wDrU1nm67zj6/8A1qk107H/2Q==" width="900" />
</a>
</div>
</div>
</div>

<p>For more details on how to configure OpenTelemetry tracing for Flux, please refer to the
<a href="https://fluxcd.io/flux/components/notification/providers/#otel">notification-controller documentation</a>.</p>
<h2 id="controller-improvements">Controller Improvements</h2>
<ul>
<li>The <code>GitRepository</code> API gains support for mTLS in GitHub App authentication.</li>
<li>The <code>Kustomization</code> API now supports
<a href="https://fluxcd.io/flux/components/kustomize/kustomizations/#dependency-ready-expression">CEL expressions</a> for extended readiness evaluation of dependencies.</li>
<li>The <code>Kustomization</code> API gains a new field <code>.spec.ignoreMissingComponents</code> for ignoring missing Kustomize components in the source.</li>
<li>The kustomize-controller now supports global SOPS decryption for Age keys, allowing centralized management of decryption keys.</li>
<li>The kustomize-controller can be configured to cancel ongoing health checks when a new source revision is detected with the <code>--feature-gates=CancelHealthCheckOnNewRevision=true</code> flag.</li>
<li>The <code>HelmRelease</code> API now supports
<a href="https://fluxcd.io/flux/components/helm/helmreleases/#dependency-ready-expression">CEL expressions</a> for extended readiness evaluation of dependencies.</li>
<li>The <code>HelmRelease</code> API gains a new strategy called <code>RetryOnFailure</code> for better handling of release failures.</li>
<li>The <code>Provider</code> API now supports setting proxy via <code>spec.proxySecretRef</code> and mTLS via <code>spec.certSecretRef</code>.</li>
<li>The <code>Provider</code> API has been extended with support for Zulip and OpenTelemetry tracing.</li>
</ul>
<h2 id="cli-improvements">CLI Improvements</h2>
<ul>
<li>The <code>flux bootstrap</code> and <code>flux install</code> commands now support the <code>--components-extra=source-watcher</code> flag to enable the new source-watcher component.</li>
<li>A new <code>flux migrate</code> command has been added to migrate Flux resources stored in Kubernetes etcd to their latest API version.</li>
<li>The <code>flux debug</code> command gains a new <code>--show-history</code> flag to display the reconciliation history of Flux objects.</li>
<li>The <code>flux diff</code> command now handles the <code>kustomize.toolkit.fluxcd.io/force: Enabled</code> annotation.</li>
<li>The <code>flux create hr</code> command gains a new <code>--storage-namespace</code> flag for changing the namespace of Helm storage objects.</li>
<li>New commands were added for <code>ImagePolicy</code> resources:
<ul>
<li><code>flux reconcile image policy</code></li>
<li><code>flux suspend image policy</code></li>
<li><code>flux resume image policy</code></li>
</ul>
</li>
<li>New commands were added for <code>ArtifactGenerator</code> resources:
<ul>
<li><code>flux get artifact generator</code></li>
<li><code>flux export artifact generator</code></li>
<li><code>flux tree artifact generator</code></li>
<li><code>flux events --for ArtifactGenerator/&lt;name&gt;</code></li>
</ul>
</li>
</ul>
<h2 id="artifact-generators">Artifact Generators</h2>
<p>Flux v2.7 comes with a new component that can be enabled at bootstrap time
with the <code>--components-extra=source-watcher</code> flag.</p>
<p>The
<a href="https://github.com/fluxcd/source-watcher" target="_blank">source-watcher</a> controller
implements the
<a href="https://fluxcd.io/flux/components/source/artifactgenerators/">ArtifactGenerator</a> API which allows Flux users to:</p>
<ul>
<li><strong>Compose</strong> multiple Flux sources (GitRepository, OCIRepository, Bucket) into a single deployable artifact</li>
<li><strong>Decompose</strong> monorepos into multiple independent artifacts with separate deployment lifecycles</li>
<li><strong>Optimize</strong> reconciliation by only triggering updates when specific paths change</li>
<li><strong>Structure</strong> complex deployments from distributed sources maintained by different teams</li>
</ul>
<h3 id="multiple-source-composition">Multiple Source Composition</h3>
<p>The <code>ArtifactGenerator</code> can be used to combine multiple sources into a single deployable artifact,
for example, you can combine upstream Helm charts from OCI registries
with your organization&rsquo;s custom values and configuration overrides stored in Git:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>source.extensions.fluxcd.io/v1beta1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ArtifactGenerator<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>podinfo<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">alias</span>:<span style="color: #bbb;"> </span>chart<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>OCIRepository<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>podinfo-chart<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">alias</span>:<span style="color: #bbb;"> </span>repo<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>GitRepository<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>podinfo-values<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">artifacts</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>podinfo-composite<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">originRevision</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@chart"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">copy</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">from</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@chart/"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">to</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@artifact/"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">from</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@repo/charts/podinfo/values.yaml"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">to</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@artifact/podinfo/values.yaml"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">strategy</span>:<span style="color: #bbb;"> </span>Overwrite<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">from</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@repo/charts/podinfo/values-prod.yaml"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">to</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@artifact/podinfo/values.yaml"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">strategy</span>:<span style="color: #bbb;"> </span>Merge<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #0e84b5; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>helm.toolkit.fluxcd.io/v2<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>HelmRelease<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>podinfo<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">interval</span>:<span style="color: #bbb;"> </span>15m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">releaseName</span>:<span style="color: #bbb;"> </span>podinfo<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">chartRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ExternalArtifact<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>podinfo-composite<span style="color: #bbb;">
</span></span></span></code></pre></div><h3 id="monorepo-decomposition">Monorepo Decomposition</h3>
<p>The <code>ArtifactGenerator</code> can be used to decompose a monorepo into multiple independent artifacts
with separate deployment lifecycles. For example:</p>
<div class="highlight"><pre tabindex="0"><code class="language-yaml"><span style="display: flex;"><span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>source.extensions.fluxcd.io/v1beta1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ArtifactGenerator<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>app-decomposer<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sources</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">alias</span>:<span style="color: #bbb;"> </span>git<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>GitRepository<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>monorepo<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">artifacts</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>frontend<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">originRevision</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@git"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">copy</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">from</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@git/deploy/frontend/**"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">to</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@artifact/"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>backend<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">originRevision</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@git"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">copy</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span>- <span style="color: #062873; font-weight: bold;">from</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@git/deploy/backend/**"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">to</span>:<span style="color: #bbb;"> </span><span style="color: #4070a0;">"@artifact/"</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #0e84b5; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>kustomize.toolkit.fluxcd.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Kustomization<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>frontend-service<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">interval</span>:<span style="color: #bbb;"> </span>15m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">prune</span>:<span style="color: #bbb;"> </span><span style="color: #007020; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sourceRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ExternalArtifact<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>frontend<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">path</span>:<span style="color: #bbb;"> </span>./<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #0e84b5; font-weight: bold;">---</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">apiVersion</span>:<span style="color: #bbb;"> </span>kustomize.toolkit.fluxcd.io/v1<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>Kustomization<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">metadata</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>backend-service<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">namespace</span>:<span style="color: #bbb;"> </span>apps<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"></span><span style="color: #062873; font-weight: bold;">spec</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">interval</span>:<span style="color: #bbb;"> </span>15m<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">prune</span>:<span style="color: #bbb;"> </span><span style="color: #007020; font-weight: bold;">true</span><span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">sourceRef</span>:<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">kind</span>:<span style="color: #bbb;"> </span>ExternalArtifact<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">name</span>:<span style="color: #bbb;"> </span>backend<span style="color: #bbb;">
</span></span></span><span style="display: flex;"><span><span style="color: #bbb;"> </span><span style="color: #062873; font-weight: bold;">path</span>:<span style="color: #bbb;"> </span>./<span style="color: #bbb;">
</span></span></span></code></pre></div><p>Each service gets its own <code>ExternalArtifact</code> with an independent revision.
Changes to <code>deploy/backend/</code> only trigger the reconciliation of the backend-service <code>Kustomization</code>,
leaving other services untouched.</p>
<p>For more details on how to use the <code>ArtifactGenerator</code> API, please refer to the
<a href="https://fluxcd.io/flux/components/source/artifactgenerators/">source-watcher documentation</a>.</p>
<h2 id="supported-versions">Supported Versions</h2>
<p>Flux v2.4 has reached end-of-life and is no longer supported.</p>
<p>Flux v2.7 supports the following Kubernetes versions:</p>
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
<td style="text-align: left;">1.32, 1.33, 1.34</td>
</tr>
<tr>
<td style="text-align: left;">OpenShift</td>
<td style="text-align: left;">4.19</td>
</tr>
</tbody>
</table>
<blockquote>
<p><strong>Enterprise support</strong> Note that the CNCF Flux project offers support only for the latest three minor versions of Kubernetes.
Backwards compatibility with older versions of Kubernetes and OpenShift is offered by vendors such as
<a href="https://control-plane.io/enterprise-for-flux-cd/" target="_blank">ControlPlane</a> that provide enterprise support for Flux.</p>
</blockquote>
<h2 id="upgrade-procedure">Upgrade Procedure</h2>
<p>Note that in Flux v2.7, the following APIs have reached end-of-life and have been removed from the CRDs:</p>
<ul>
<li><code>source.toolkit.fluxcd.io/v1beta1</code></li>
<li><code>kustomize.toolkit.fluxcd.io/v1beta1</code></li>
<li><code>helm.toolkit.fluxcd.io/v2beta1</code></li>
<li><code>image.toolkit.fluxcd.io/v1beta1</code></li>
<li><code>notification.toolkit.fluxcd.io/v1beta1</code></li>
</ul>
<p>Before upgrading to Flux v2.7, make sure to migrate all your resources to the stable APIs
using the
<a href="https://fluxcd.io/flux/cmd/flux_migrate/">flux migrate</a> command.</p>
<div class="alert alert-info">
<h4 class="alert-heading">Upgrade Procedure for Flux v2.7+</h4>
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
