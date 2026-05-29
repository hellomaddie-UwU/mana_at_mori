<script lang="ts">
	import { resolve } from '$app/paths';
	import { onMount } from 'svelte';

	const colors = [
		{ label: 'Primary', value: 'var(--ds-color-primary)', hex: '#b85f5b' },
		{ label: 'Secondary', value: 'var(--ds-color-secondary)', hex: '#c5d8c6' },
		{ label: 'Accent', value: 'var(--ds-color-danger)', hex: '#4f2c2e' },
		{ label: 'Background', value: 'var(--ds-color-bg)', hex: '#f7f2e7' },
		{ label: 'Surface', value: 'var(--ds-color-surface)', hex: '#ffffff' },
		{ label: 'Text Primary', value: 'var(--ds-color-text)', hex: '#1E2A34' },
		{ label: 'Text Muted', value: 'var(--ds-color-text-muted)', hex: '#7B8D9B' },
		{ label: 'Border', value: 'var(--ds-color-border)', hex: '#c5d8c6' },
		{ label: 'Code BG', value: 'var(--ds-color-code-bg)', hex: '#1E2A34' }
	];

	const radii = [
		{ label: '--radius-none', value: '0px' },
		{ label: '--radius-sm', value: '4px' },
		{ label: '--radius-md', value: '8px' },
		{ label: '--radius-lg', value: '12px' },
		{ label: '--radius-xl', value: '20px' },
		{ label: '--radius-full', value: '9999px' }
	];

	const transitions = [
		{ label: '--ease-fast', value: '0.10s ease' },
		{ label: '--ease-base', value: '0.20s ease' },
		{ label: '--ease-smooth', value: '0.30s cubic-bezier(0.4, 0, 0.2, 1)' },
		{ label: '--ease-bounce', value: '0.40s cubic-bezier(0.34, 1.56, 0.64, 1)' },
		{ label: '--ease-slow', value: '0.60s ease' }
	];

	const badgeTones = [
		{ label: 'Default', class: 'badge-default' },
		{ label: 'Primary', class: 'badge-primary' },
		{ label: 'Accent', class: 'badge-accent' },
		{ label: 'Success', class: 'badge-success' },
		{ label: 'Warning', class: 'badge-warning' },
		{ label: 'Error', class: 'badge-error' }
	];

	const sections = [
		{ id: 'colors', label: 'Colors' },
		{ id: 'typography-tokens', label: 'Typography' },
		{ id: 'spacing', label: 'Spacing' },
		{ id: 'radii', label: 'Border Radius' },
		{ id: 'shadows', label: 'Shadows' },
		{ id: 'transitions', label: 'Transitions' },
		{ id: 'type-styles', label: 'Type Styles' },
		{ id: 'buttons', label: 'Buttons' },
		{ id: 'dividers', label: 'Dividers' },
		{ id: 'swatches', label: 'Color Swatches' },
		{ id: 'badges', label: 'Badges & Tags' },
		{ id: 'forms', label: 'Form Elements' },
		{ id: 'cards', label: 'Cards' },
		{ id: 'alerts', label: 'Alerts & Toasts' },
		{ id: 'tables', label: 'Tables' },
		{ id: 'codeblocks', label: 'Code Blocks' },
		{ id: 'breadcrumbs', label: 'Breadcrumbs' },
		{ id: 'avatars', label: 'Avatars' },
		{ id: 'tooltips', label: 'Tooltips' },
		{ id: 'progress', label: 'Progress & Steps' }
	];

	let activeSection = $state('colors');

	onMount(() => {
		const sectionNodes = document.querySelectorAll<HTMLElement>('.ds-section');
		const navLinks = document.querySelectorAll<HTMLAnchorElement>('.ds-nav a');

		if (!sectionNodes.length || !navLinks.length) return;

		const observer = new IntersectionObserver(
			(entries) => {
				entries.forEach((entry) => {
					if (!entry.isIntersecting) return;
					activeSection = entry.target.id;
					navLinks.forEach((link) => link.classList.remove('active'));
					const active = document.querySelector<HTMLAnchorElement>(
						`.ds-nav a[href="#${entry.target.id}"]`
					);
					active?.classList.add('active');
				});
			},
			{ rootMargin: '-20% 0px -70% 0px' }
		);

		sectionNodes.forEach((section) => observer.observe(section));

		return () => observer.disconnect();
	});
</script>

<svelte:head>
	<title>Mana at Mori Design System</title>
</svelte:head>

<nav class="ds-topbar">
	<div class="container-lg d-flex align-items-center justify-content-between">
		<a href={resolve('/design-system')} class="brand">Mana at Mori <span>/ Design System</span></a>
		<div class="d-flex align-items-center gap-3">
			<span class="ds-version">v1.0.0</span>
			<span class="badge-ds badge-accent">Sage + Red</span>
		</div>
	</div>
</nav>

<div class="container-lg py-5">
	<div class="row g-5">
		<div class="col-lg-2 d-none d-lg-block">
			<nav class="ds-sidebar ds-nav">
				<div class="ds-nav-label">Tokens</div>
				{#each sections.slice(0, 6) as section (section.id)}
					<a href={`#${section.id}`} class:active={activeSection === section.id}>{section.label}</a>
				{/each}
				<div class="ds-nav-label">Components</div>
				{#each sections.slice(6) as section (section.id)}
					<a href={`#${section.id}`} class:active={activeSection === section.id}>{section.label}</a>
				{/each}
			</nav>
		</div>

		<div class="col-lg-10">
			<header class="ds-header">
				<p class="eyebrow">Design Tokens & Components</p>
				<h1>Mana at Mori Design System</h1>
				<p class="mt-3 hero-copy">
					A reusable foundation of CSS variables and UI components for Mana at Mori projects. The
					page follows the provided HTML reference while using the red-and-sage palette from the
					image.
				</p>
			</header>

			<section class="ds-section" id="colors">
				<h2 class="ds-section-title">Colors</h2>
				<div class="ds-card">
					<div class="swatch-wrap">
						{#each colors as color (color.label)}
							<div class="swatch">
								<div class="swatch-block" style={`background:${color.hex};`}></div>
								<span class="swatch-name">{color.label}</span>
								<span class="swatch-value">{color.value}<br />{color.hex}</span>
							</div>
						{/each}
					</div>
				</div>
			</section>

			<section class="ds-section" id="typography-tokens">
				<h2 class="ds-section-title">Typography Tokens</h2>
				<div class="ds-card">
					<table class="token-table">
						<thead>
							<tr>
								<th>Token</th>
								<th>Value</th>
								<th>Usage</th>
							</tr>
						</thead>
						<tbody>
							<tr><td><code>--font-serif</code></td><td>Serif display</td><td>Headers</td></tr>
							<tr><td><code>--font-sans</code></td><td>UI sans</td><td>Body and controls</td></tr>
							<tr><td><code>--font-mono</code></td><td>Monospace</td><td>Code and tokens</td></tr>
							<tr
								><td><code>--text-xs … --text-4xl</code></td><td>0.75rem → 3rem</td><td
									>Type scale</td
								></tr
							>
						</tbody>
					</table>
				</div>
			</section>

			<section class="ds-section" id="spacing">
				<h2 class="ds-section-title">Spacing Scale</h2>
				<div class="ds-card">
					<table class="token-table">
						<thead><tr><th>Token</th><th>Value</th><th>px equiv.</th></tr></thead>
						<tbody>
							<tr><td><code>--space-1</code></td><td>0.25rem</td><td>4px</td></tr>
							<tr><td><code>--space-2</code></td><td>0.5rem</td><td>8px</td></tr>
							<tr><td><code>--space-3</code></td><td>0.75rem</td><td>12px</td></tr>
							<tr><td><code>--space-4</code></td><td>1rem</td><td>16px</td></tr>
						</tbody>
					</table>
				</div>
			</section>

			<section class="ds-section" id="radii">
				<h2 class="ds-section-title">Border Radius</h2>
				<div class="ds-card">
					<div class="radius-demo">
						{#each radii as radius (radius.label)}
							<div class="d-flex flex-column align-items-center gap-2">
								<div class="radius-item" style={`border-radius:${radius.value};`}></div>
								<span class="swatch-value">{radius.label}<br />{radius.value}</span>
							</div>
						{/each}
					</div>
				</div>
			</section>

			<section class="ds-section" id="shadows">
				<h2 class="ds-section-title">Box Shadows</h2>
				<div class="ds-card">
					<div class="shadow-demo">
						<div class="shadow-item" style="box-shadow: var(--ds-shadow-sm);">
							<span class="fw-medium">Aa</span>
							<div class="shadow-label">--shadow-sm</div>
						</div>
						<div class="shadow-item" style="box-shadow: var(--ds-shadow-md);">
							<span class="fw-medium">Aa</span>
							<div class="shadow-label">--shadow-md</div>
						</div>
					</div>
				</div>
			</section>

			<section class="ds-section" id="transitions">
				<h2 class="ds-section-title">Transition Timing</h2>
				<div class="ds-card">
					<p class="text-caption mb-4">Hover each row to preview the easing.</p>
					<div class="ease-demo">
						{#each transitions as item (item.label)}
							<div class="ease-bar-wrap">
								<span class="ease-label">{item.label}</span>
								<div class="ease-bar" style={`transition: width ${item.value};`}></div>
								<span class="text-caption ms-3">{item.value}</span>
							</div>
						{/each}
					</div>
				</div>
			</section>

			<hr class="divider-thick" style="margin: var(--space-12) 0;" />

			<section class="ds-section" id="type-styles">
				<h2 class="ds-section-title">Type Styles</h2>
				<div class="ds-card">
					<div class="type-row">
						<div class="type-meta"><strong>h1</strong>Serif / 48px / 500</div>
						<h1 class="mb-0 demo-heading">Heading One</h1>
					</div>
					<div class="type-row">
						<div class="type-meta"><strong>h2</strong>Serif / 36px / 500</div>
						<h2 class="mb-0 demo-heading">Heading Two</h2>
					</div>
					<div class="type-row">
						<div class="type-meta"><strong>h3</strong>Serif / 30px / 500</div>
						<h3 class="mb-0 demo-heading">Heading Three</h3>
					</div>
					<div class="type-row">
						<div class="type-meta"><strong>body</strong>Sans / 16px / 400 / 1.6</div>
						<p class="mb-0">Body copy for paragraphs across the app.</p>
					</div>
					<div class="type-row">
						<div class="type-meta"><strong>caption</strong>Sans / 14px / muted</div>
						<p class="text-caption mb-0">Caption text for metadata and helper labels.</p>
					</div>
					<div class="type-row">
						<div class="type-meta"><strong>overline</strong>Sans / 12px / uppercase</div>
						<span class="text-overline">Section Label</span>
					</div>
				</div>
			</section>

			<section class="ds-section" id="buttons">
				<h2 class="ds-section-title">Buttons</h2>
				<div class="ds-card mb-4">
					<p class="text-overline mb-4">Variants</p>
					<div class="d-flex flex-wrap gap-3 align-items-center">
						<button class="btn btn-primary-ds">Primary</button>
						<button class="btn btn-secondary-ds">Secondary</button>
						<button class="btn btn-ghost-ds">Ghost</button>
						<button class="btn btn-accent-ds">Accent</button>
					</div>
				</div>
				<div class="ds-card">
					<p class="text-overline mb-4">Sizes</p>
					<div class="d-flex flex-wrap gap-3 align-items-center">
						<button class="btn btn-primary-ds btn-sm-ds">Small</button>
						<button class="btn btn-primary-ds">Default</button>
						<button class="btn btn-primary-ds btn-lg-ds">Large</button>
					</div>
				</div>
			</section>

			<section class="ds-section" id="dividers">
				<h2 class="ds-section-title">Dividers</h2>
				<div class="ds-card">
					<hr class="divider" />
					<hr class="divider-thick" />
					<hr class="divider-accent" />
					<hr class="divider-dashed" />
				</div>
			</section>

			<section class="ds-section" id="swatches">
				<h2 class="ds-section-title">Color Swatches</h2>
				<div class="ds-card">
					<div class="swatch-wrap">
						{#each colors.slice(0, 4) as color (color.label)}
							<div class="swatch">
								<div class="swatch-block" style={`background:${color.hex};`}></div>
								<span class="swatch-name">{color.label}</span>
								<span class="swatch-value">{color.hex}</span>
							</div>
						{/each}
					</div>
				</div>
			</section>

			<section class="ds-section" id="badges">
				<h2 class="ds-section-title">Badges &amp; Tags</h2>
				<div class="ds-card mb-4">
					<div class="d-flex flex-wrap gap-2 align-items-center">
						{#each badgeTones as badge (badge.class)}
							<span class={`badge-ds ${badge.class}`}>{badge.label}</span>
						{/each}
					</div>
				</div>
				<div class="ds-card">
					<div class="d-flex flex-wrap gap-2 align-items-center">
						<span class="badge-ds badge-outline">Outline</span>
						<span class="badge-ds badge-muted">Muted</span>
						<span class="badge-ds badge-success">✓ Approved</span>
						<span class="badge-ds badge-warning">⚠ Review</span>
						<span class="badge-ds badge-error">✕ Rejected</span>
					</div>
				</div>
			</section>

			<section class="ds-section" id="forms">
				<h2 class="ds-section-title">Form Elements</h2>
				<div class="ds-card mb-4">
					<div class="row g-4">
						<div class="col-md-6">
							<label class="form-label-ds" for="ds-input-default">Default</label>
							<input
								id="ds-input-default"
								class="input-ds"
								type="text"
								placeholder="Placeholder text"
							/>
						</div>
						<div class="col-md-6">
							<label class="form-label-ds" for="ds-input-error">Error State</label>
							<input
								id="ds-input-error"
								class="input-ds input-error"
								type="text"
								value="Invalid input"
							/>
							<p class="form-error-msg">This field is required.</p>
						</div>
						<div class="col-md-6">
							<label class="form-label-ds" for="ds-input-disabled">Disabled</label>
							<input
								id="ds-input-disabled"
								class="input-ds"
								type="text"
								value="Cannot edit this"
								disabled
							/>
						</div>
						<div class="col-md-6">
							<label class="form-label-ds" for="ds-input-textarea">Textarea</label>
							<textarea
								id="ds-input-textarea"
								class="input-ds"
								rows="3"
								placeholder="Enter a longer description..."
							></textarea>
						</div>
					</div>
				</div>
			</section>

			<section class="ds-section" id="cards">
				<h2 class="ds-section-title">Cards</h2>
				<div class="row g-4">
					<div class="col-md-4">
						<div class="card-ds">
							<div class="card-ds-body">
								<h5 class="mb-2">Card Title</h5>
								<p class="text-caption mb-3">A simple card body for content grouping.</p>
							</div>
						</div>
					</div>
					<div class="col-md-4">
						<div class="card-ds">
							<div class="card-ds-header">Card Header</div>
							<div class="card-ds-body">
								<p class="text-caption mb-0">Card body content goes here.</p>
							</div>
							<div class="card-ds-footer">Updated 2 days ago</div>
						</div>
					</div>
					<div class="col-md-4">
						<div class="card-ds card-ds-horizontal" style="height: 130px;">
							<div class="card-ds-img-placeholder"><span>IMG</span></div>
							<div class="card-ds-body d-flex flex-column justify-content-center">
								<h6 class="mb-1">Audit Case Study</h6>
								<p class="text-caption mb-0">Short description for this item.</p>
							</div>
						</div>
					</div>
				</div>
			</section>

			<section class="ds-section" id="alerts">
				<h2 class="ds-section-title">Alerts &amp; Toasts</h2>
				<div class="ds-card mb-4">
					<div class="d-flex flex-column gap-3">
						<div class="alert-ds alert-info">
							<span class="alert-ds-icon">ℹ</span>
							<div>
								<div class="alert-ds-title">Info</div>
								Your audit report will be ready within 3–5 business days.
							</div>
						</div>
						<div class="alert-ds alert-success">
							<span class="alert-ds-icon">✓</span>
							<div>
								<div class="alert-ds-title">Success</div>
								Payment received. Your project has been confirmed.
							</div>
						</div>
						<div class="alert-ds alert-warning">
							<span class="alert-ds-icon">⚠</span>
							<div>
								<div class="alert-ds-title">Warning</div>
								Your onboarding checklist is incomplete.
							</div>
						</div>
						<div class="alert-ds alert-error">
							<span class="alert-ds-icon">✕</span>
							<div>
								<div class="alert-ds-title">Error</div>
								Something went wrong. Please refresh and try again.
							</div>
						</div>
					</div>
				</div>
				<div class="ds-card">
					<div class="d-flex flex-wrap gap-4">
						<div class="toast-ds">
							<div class="toast-dot toast-dot-success"></div>
							<div>
								<div class="alert-ds-title" style="font-size: var(--text-sm);">Report exported</div>
								<span style="font-size: var(--text-xs); color: var(--color-text-muted);"
									>audit-report-v2.pdf is ready to download.</span
								>
							</div>
						</div>
						<div class="toast-ds">
							<div class="toast-dot toast-dot-warning"></div>
							<div>
								<div class="alert-ds-title" style="font-size: var(--text-sm);">
									Session expiring
								</div>
								<span style="font-size: var(--text-xs); color: var(--color-text-muted);"
									>You&apos;ll be logged out in 5 minutes.</span
								>
							</div>
						</div>
					</div>
				</div>
			</section>

			<section class="ds-section" id="tables">
				<h2 class="ds-section-title">Tables</h2>
				<div class="ds-card">
					<table class="table-ds table-striped">
						<thead><tr><th>Page</th><th>Issues Found</th><th>Priority</th></tr></thead>
						<tbody>
							<tr><td>Dashboard</td><td>12</td><td>High</td></tr>
							<tr><td>Onboarding</td><td>8</td><td>High</td></tr>
							<tr><td>Settings</td><td>4</td><td>Medium</td></tr>
						</tbody>
					</table>
				</div>
			</section>

			<section class="ds-section" id="codeblocks">
				<h2 class="ds-section-title">Code Blocks</h2>
				<div class="d-flex flex-column gap-4">
					<div>
						<p class="text-overline mb-3">CSS</p>
						<div class="codeblock-ds">
							<div class="codeblock-ds-bar">
								<div class="codeblock-ds-dots">
									<span style="background:#FF5F57;"></span><span style="background:#FFBD2E;"
									></span><span style="background:#28C840;"></span>
								</div>
								<span class="codeblock-ds-lang">css</span>
								<button class="codeblock-copy" type="button">Copy</button>
							</div>
							<pre><code
									><span class="code-comment">/* Apply primary button styles */</span>
<span class="code-key">.btn-primary-ds</span> &#123;
  <span class="code-key">background-color</span>: <span class="code-val"
										>var(--ds-color-primary)</span
									>;
  <span class="code-key">color</span>: <span class="code-str">#FFFFFF</span>;
  <span class="code-key">border-radius</span>: <span class="code-val">var(--ds-radius-md)</span>;
  <span class="code-key">transition</span>: <span class="code-val">var(--ds-motion-base)</span>;
&#125;</code
								></pre>
						</div>
					</div>

					<div>
						<p class="text-overline mb-3">HTML</p>
						<div class="codeblock-ds">
							<div class="codeblock-ds-bar">
								<div class="codeblock-ds-dots">
									<span style="background:#FF5F57;"></span><span style="background:#FFBD2E;"
									></span><span style="background:#28C840;"></span>
								</div>
								<span class="codeblock-ds-lang">html</span>
								<button class="codeblock-copy" type="button">Copy</button>
							</div>
							<pre><code
									><span class="code-comment">&lt;!-- Audit scope card --&gt;</span>
&lt;<span class="code-key">div</span> <span class="code-val">class</span>=<span class="code-str"
										>"card-ds"</span
									>&gt;
  &lt;<span class="code-key">div</span> <span class="code-val">class</span>=<span class="code-str"
										>"card-ds-header"</span
									>&gt;Web App Audit&lt;/<span class="code-key">div</span>&gt;
  &lt;<span class="code-key">div</span> <span class="code-val">class</span>=<span class="code-str"
										>"card-ds-body"</span
									>&gt;
    &lt;<span class="code-key">p</span>&gt;11-phase heuristic audit workflow.&lt;/<span
										class="code-key">p</span
									>&gt;
  &lt;/<span class="code-key">div</span>&gt;
&lt;/<span class="code-key">div</span>&gt;</code
								></pre>
						</div>
					</div>

					<div>
						<p class="text-overline mb-3">JavaScript</p>
						<div class="codeblock-ds">
							<div class="codeblock-ds-bar">
								<div class="codeblock-ds-dots">
									<span style="background:#FF5F57;"></span><span style="background:#FFBD2E;"
									></span><span style="background:#28C840;"></span>
								</div>
								<span class="codeblock-ds-lang">javascript</span>
								<button class="codeblock-copy" type="button">Copy</button>
							</div>
							<pre><code
									><span class="code-comment">// Scroll-spy: highlight active nav link</span>
<span class="code-key">const</span> sections = document.<span class="code-val"
										>querySelectorAll</span
									>(<span class="code-str">'.ds-section'</span>);
<span class="code-key">const</span> navLinks = document.<span class="code-val"
										>querySelectorAll</span
									>(<span class="code-str">'.ds-nav a'</span>);

<span class="code-key">const</span> observer = <span class="code-key">new</span> <span
										class="code-val">IntersectionObserver</span
									>((entries) =&gt; &#123;
  entries.<span class="code-val">forEach</span>((entry) =&gt; &#123;
    <span class="code-key">if</span> (entry.isIntersecting) &#123;
      navLinks.<span class="code-val">forEach</span>((a) =&gt; a.classList.<span class="code-val"
										>remove</span
									>(<span class="code-str">'active'</span>));
      <span class="code-key">const</span> active = document.<span class="code-val"
										>querySelector</span
									>(<span class="code-str">'.ds-nav a[href="#' + entry.target.id + '"]'</span>);
      <span class="code-key">if</span> (active) active.classList.<span class="code-val">add</span
									>(<span class="code-str">'active'</span>);
    &#125;
  &#125;);
&#125;, &#123; rootMargin: <span class="code-str">'-20% 0px -70% 0px'</span> &#125;);</code
								></pre>
						</div>
					</div>
				</div>
			</section>

			<section class="ds-section" id="breadcrumbs">
				<h2 class="ds-section-title">Breadcrumbs</h2>
				<div class="ds-card">
					<ol class="breadcrumb-ds">
						<li><a href={resolve('/')}>Home</a></li>
						<li><span class="sep">/</span><a href={resolve('/design-system')}>Portfolio</a></li>
						<li><span class="sep">/</span><span class="current">Mana at Mori</span></li>
					</ol>
				</div>
			</section>

			<section class="ds-section" id="avatars">
				<h2 class="ds-section-title">Avatars</h2>
				<div class="ds-card">
					<div class="avatar-group">
						<div
							class="avatar-ds avatar-circle avatar-md"
							style="background: var(--ds-color-danger); color:#fff; border-color: var(--ds-color-danger);"
						>
							BM
						</div>
						<div
							class="avatar-ds avatar-circle avatar-md"
							style="background: var(--ds-color-primary); color:#fff; border-color: var(--ds-color-primary);"
						>
							MA
						</div>
						<div class="avatar-ds avatar-circle avatar-md">+4</div>
					</div>
				</div>
			</section>

			<section class="ds-section" id="tooltips">
				<h2 class="ds-section-title">Tooltips</h2>
				<div class="ds-card">
					<div class="d-flex flex-wrap gap-5 align-items-start">
						<div class="tooltip-ds-wrap">
							<button class="btn btn-secondary-ds">Top</button>
							<div class="tooltip-ds">Appears above the element</div>
						</div>
						<div class="tooltip-ds-wrap tooltip-bottom">
							<button class="btn btn-secondary-ds">Bottom</button>
							<div class="tooltip-ds">Appears below the element</div>
						</div>
					</div>
				</div>
			</section>

			<section class="ds-section" id="progress">
				<h2 class="ds-section-title">Progress &amp; Steps</h2>
				<div class="ds-card mb-4">
					<div class="d-flex flex-column gap-4">
						<div class="progress-ds-wrap">
							<div class="progress-ds-label"><span>Audit Complete</span><span>72%</span></div>
							<div class="progress-ds-track">
								<div class="progress-ds-bar" style="width:72%;"></div>
							</div>
						</div>
						<div class="progress-ds-wrap">
							<div class="progress-ds-label"><span>Discovery</span><span>100%</span></div>
							<div class="progress-ds-track">
								<div class="progress-ds-bar bar-success" style="width:100%;"></div>
							</div>
						</div>
					</div>
				</div>
				<div class="ds-card">
					<div class="steps-ds">
						<div class="step-ds complete">
							<div class="step-node">✓</div>
							<div class="step-label">Discovery</div>
						</div>
						<div class="step-ds complete">
							<div class="step-node">✓</div>
							<div class="step-label">Audit</div>
						</div>
						<div class="step-ds active">
							<div class="step-node">3</div>
							<div class="step-label">PRD</div>
						</div>
						<div class="step-ds">
							<div class="step-node">4</div>
							<div class="step-label">Review</div>
						</div>
					</div>
				</div>
			</section>

			<footer class="ds-footer">
				<div class="d-flex justify-content-between align-items-center flex-wrap gap-3">
					<span class="text-caption">Mana at Mori Design System · v1.0.0</span>
					<div class="d-flex gap-2 flex-wrap">
						<span class="badge-ds badge-default">Sage + Red</span>
						<span class="badge-ds badge-default">Lora + Jost</span>
						<span class="badge-ds badge-default">CSS Variables</span>
					</div>
				</div>
			</footer>
		</div>
	</div>
</div>

<style>
	.ds-topbar {
		background: var(--ds-color-surface);
		border-bottom: 1px solid var(--ds-color-border);
		padding: var(--ds-space-4) 0;
		position: sticky;
		top: 0;
		z-index: 100;
		box-shadow: var(--ds-shadow-sm);
	}

	.brand {
		font-family: Georgia, 'Times New Roman', serif;
		font-size: 1.1rem;
		font-weight: 600;
		color: var(--ds-color-text);
		text-decoration: none;
	}

	.brand span {
		color: var(--ds-color-danger);
	}

	.ds-version {
		font-size: var(--ds-text-xs);
		font-family: var(--ds-font-mono);
		color: var(--ds-color-text-muted);
		background: var(--ds-color-surface-2);
		padding: 0.2em 0.6em;
		border-radius: var(--ds-radius-sm);
	}

	.ds-sidebar {
		position: sticky;
		top: var(--ds-space-8);
		height: calc(100vh - var(--ds-space-16));
		overflow-y: auto;
		padding-right: var(--ds-space-4);
	}

	.ds-sidebar::-webkit-scrollbar {
		width: 4px;
	}
	.ds-sidebar::-webkit-scrollbar-track {
		background: transparent;
	}
	.ds-sidebar::-webkit-scrollbar-thumb {
		background: var(--ds-color-border);
		border-radius: 999px;
	}

	.ds-nav-label {
		font-size: var(--ds-text-xs);
		font-weight: 700;
		letter-spacing: 0.1em;
		text-transform: uppercase;
		color: var(--ds-color-text-muted);
		margin-bottom: var(--ds-space-2);
		padding: var(--ds-space-3) 0 var(--ds-space-1) 0;
	}

	.ds-nav a {
		display: block;
		font-size: var(--ds-text-sm);
		color: var(--ds-color-text-muted);
		text-decoration: none;
		padding: var(--ds-space-1) 0 var(--ds-space-1) var(--ds-space-3);
		border-left: 2px solid transparent;
		transition: all 0.2s ease;
		line-height: var(--ds-line-normal);
	}

	.ds-nav a:hover,
	.ds-nav a.active {
		color: var(--ds-color-text);
		border-color: var(--ds-color-danger);
	}

	.ds-header {
		border-bottom: 1px solid var(--ds-color-border);
		padding-bottom: var(--ds-space-8);
		margin-bottom: var(--ds-space-12);
	}

	.eyebrow {
		font-size: var(--ds-text-xs);
		font-weight: 700;
		letter-spacing: 0.12em;
		text-transform: uppercase;
		color: var(--ds-color-danger);
		margin-bottom: var(--ds-space-3);
	}

	.hero-copy {
		max-width: 560px;
		color: var(--ds-color-text-muted);
	}

	.ds-section {
		margin-bottom: var(--ds-space-7);
		scroll-margin-top: var(--ds-space-8);
	}

	.ds-section-title {
		font-family: var(--ds-font-sans);
		font-size: var(--ds-text-xs);
		font-weight: 700;
		letter-spacing: 0.12em;
		text-transform: uppercase;
		color: var(--ds-color-danger);
		margin-bottom: var(--ds-space-6);
		display: flex;
		align-items: center;
		gap: var(--ds-space-3);
	}

	.ds-section-title::after {
		content: '';
		flex: 1;
		height: 1px;
		background: var(--ds-color-border);
	}

	.ds-card {
		background: var(--ds-color-surface);
		border: 1px solid var(--ds-color-border);
		border-radius: var(--ds-radius-lg);
		padding: var(--ds-space-6);
		box-shadow: var(--ds-shadow-sm);
	}

	.ds-card + .ds-card {
		margin-top: var(--ds-space-4);
	}

	.token-table,
	.table-ds {
		width: 100%;
		border-collapse: collapse;
		font-size: var(--ds-text-sm);
	}

	.token-table th,
	.table-ds thead th {
		font-weight: 700;
		font-size: var(--ds-text-xs);
		letter-spacing: 0.06em;
		text-transform: uppercase;
		color: var(--ds-color-text-muted);
		padding: var(--ds-space-3) var(--ds-space-4);
		text-align: left;
		border-bottom: 2px solid var(--ds-color-border);
		background: var(--ds-color-bg);
	}

	.token-table td,
	.table-ds tbody td {
		padding: var(--ds-space-3) var(--ds-space-4);
		border-bottom: 1px solid var(--ds-color-border);
		color: var(--ds-color-text);
		vertical-align: middle;
	}

	.token-table tr:last-child td,
	.table-ds tbody tr:last-child td {
		border-bottom: none;
	}

	.swatch-wrap {
		display: flex;
		flex-wrap: wrap;
		gap: var(--ds-space-4);
	}

	.swatch {
		display: flex;
		flex-direction: column;
		gap: var(--ds-space-2);
		min-width: 120px;
	}

	.swatch-block {
		width: 100%;
		height: 72px;
		border-radius: var(--ds-radius-md);
		box-shadow: var(--ds-shadow-sm);
		border: 1px solid rgba(0, 0, 0, 0.05);
	}

	.swatch-name {
		font-size: var(--ds-text-xs);
		font-weight: 700;
		color: var(--ds-color-text);
		letter-spacing: 0.04em;
	}

	.swatch-value {
		font-family: var(--ds-font-mono);
		font-size: var(--ds-text-xs);
		color: var(--ds-color-text-muted);
	}

	.radius-demo {
		display: flex;
		gap: var(--ds-space-4);
		flex-wrap: wrap;
		align-items: center;
	}

	.radius-item {
		width: 64px;
		height: 64px;
		background: var(--ds-color-surface-2);
		border: 1.5px solid var(--ds-color-border);
	}

	.shadow-demo {
		display: flex;
		gap: var(--ds-space-4);
		flex-wrap: wrap;
	}

	.shadow-item {
		background: var(--ds-color-surface);
		border-radius: var(--ds-radius-md);
		padding: var(--ds-space-4) var(--ds-space-5);
		min-width: 110px;
		text-align: center;
	}

	.shadow-label {
		font-size: var(--ds-text-xs);
		font-family: var(--ds-font-mono);
		color: var(--ds-color-text-muted);
		margin-top: var(--ds-space-3);
	}

	.type-row {
		display: flex;
		align-items: baseline;
		gap: var(--ds-space-4);
		padding: var(--ds-space-4) 0;
		border-bottom: 1px solid var(--ds-color-border);
	}

	.type-row:last-child {
		border-bottom: none;
	}

	.type-meta {
		min-width: 160px;
		font-size: var(--ds-text-xs);
		color: var(--ds-color-text-muted);
		font-family: var(--ds-font-mono);
		flex-shrink: 0;
	}

	.type-meta strong {
		display: block;
		font-family: var(--ds-font-sans);
		font-weight: 600;
		color: var(--ds-color-text);
		margin-bottom: var(--ds-space-1);
	}

	.demo-heading {
		font-family: Georgia, 'Times New Roman', serif;
	}

	.ease-demo {
		display: flex;
		flex-direction: column;
		gap: var(--ds-space-3);
	}

	.ease-bar-wrap {
		display: flex;
		align-items: center;
		gap: var(--ds-space-3);
	}

	.ease-label {
		font-family: var(--ds-font-mono);
		font-size: var(--ds-text-xs);
		color: var(--ds-color-text-muted);
		min-width: 100px;
	}

	.ease-bar {
		height: 8px;
		width: 40px;
		background: var(--ds-color-danger);
		border-radius: var(--ds-radius-pill);
		cursor: pointer;
	}

	.ease-bar-wrap:hover .ease-bar {
		width: 240px;
	}

	.divider,
	.divider-thick,
	.divider-accent,
	.divider-dashed {
		border: none;
		margin: var(--ds-space-4) 0;
	}

	.divider {
		border-top: 1px solid var(--ds-color-border);
	}
	.divider-thick {
		border-top: 2px solid var(--ds-color-text);
	}
	.divider-accent {
		border-top: 2px solid var(--ds-color-danger);
	}
	.divider-dashed {
		border-top: 1.5px dashed var(--ds-color-border);
	}

	.btn {
		font-family: var(--ds-font-sans);
		font-weight: 600;
		border-radius: var(--ds-radius-md);
		padding: 0.6rem 1.35rem;
		font-size: var(--ds-text-sm);
		line-height: 1;
		letter-spacing: 0.01em;
		border: 1.5px solid transparent;
		cursor: pointer;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		gap: var(--ds-space-2);
		transition: all 0.22s ease;
	}

	.btn-primary-ds {
		background-color: var(--ds-color-primary);
		color: #fff;
		border-color: var(--ds-color-primary);
		box-shadow: var(--ds-shadow-sm);
	}

	.btn-primary-ds:hover {
		background-color: var(--ds-color-primary-hover);
		border-color: var(--ds-color-primary-hover);
		box-shadow: var(--ds-shadow-md);
		transform: translateY(-1px);
	}

	.btn-secondary-ds {
		background-color: var(--ds-color-surface);
		color: var(--ds-color-text);
		border-color: var(--ds-color-border);
		box-shadow: var(--ds-shadow-sm);
	}

	.btn-secondary-ds:hover {
		border-color: var(--ds-color-text-muted);
		color: var(--ds-color-text);
		transform: translateY(-1px);
	}

	.btn-ghost-ds {
		background-color: transparent;
		color: var(--ds-color-text);
		border-color: transparent;
	}

	.btn-ghost-ds:hover {
		background-color: var(--ds-color-surface-2);
		color: var(--ds-color-text);
	}

	.btn-accent-ds {
		background-color: var(--ds-color-danger);
		color: #fff;
		border-color: var(--ds-color-danger);
		box-shadow: var(--ds-shadow-sm);
	}

	.btn-accent-ds:hover {
		background-color: #a1514e;
		border-color: #a1514e;
		box-shadow: var(--ds-shadow-md);
		transform: translateY(-1px);
	}

	.btn-sm-ds {
		padding: 0.4rem 0.9rem;
		font-size: var(--ds-text-xs);
	}

	.btn-lg-ds {
		padding: 0.8rem 1.75rem;
		font-size: var(--ds-text-md);
	}

	.badge-ds {
		display: inline-flex;
		align-items: center;
		gap: var(--ds-space-1);
		font-family: var(--ds-font-sans);
		font-size: var(--ds-text-xs);
		font-weight: 700;
		letter-spacing: 0.06em;
		text-transform: uppercase;
		padding: 0.25em 0.7em;
		border-radius: var(--ds-radius-pill);
		border: 1.5px solid transparent;
	}

	.badge-default {
		background: var(--ds-color-surface-2);
		color: var(--ds-color-text);
		border-color: var(--ds-color-border);
	}
	.badge-primary {
		background: var(--ds-color-primary);
		color: #fff;
	}
	.badge-accent {
		background: var(--ds-color-danger);
		color: #fff;
	}
	.badge-success {
		background: #d4ecd0;
		color: #2a6b24;
		border-color: #b0dca9;
	}
	.badge-warning {
		background: #fff0d1;
		color: #855600;
		border-color: #f5d589;
	}
	.badge-error {
		background: #fddede;
		color: #8b2020;
		border-color: #f5aaaa;
	}
	.badge-outline {
		background: transparent;
		color: var(--ds-color-text);
		border-color: var(--ds-color-text);
	}
	.badge-muted {
		background: transparent;
		color: var(--ds-color-text-muted);
		border-color: var(--ds-color-border);
	}

	.form-label-ds {
		display: block;
		font-family: var(--ds-font-sans);
		font-size: var(--ds-text-sm);
		font-weight: 600;
		margin-bottom: var(--ds-space-2);
	}

	.input-ds {
		font-family: var(--ds-font-sans);
		font-size: var(--ds-text-sm);
		color: var(--ds-color-text);
		background: var(--ds-color-surface);
		border: 1.5px solid var(--ds-color-border);
		border-radius: var(--ds-radius-md);
		padding: 0.55rem 0.85rem;
		width: 100%;
	}

	.input-ds::placeholder {
		color: var(--ds-color-text-muted);
	}
	.input-ds:focus {
		border-color: var(--ds-color-danger);
		box-shadow: 0 0 0 3px rgba(184, 95, 91, 0.15);
		outline: none;
	}
	.input-ds:disabled {
		background: var(--ds-color-surface-2);
		color: var(--ds-color-text-muted);
		cursor: not-allowed;
		opacity: 0.7;
	}
	.input-ds.input-error {
		border-color: #cc3333;
		box-shadow: 0 0 0 3px rgba(204, 51, 51, 0.12);
	}
	.form-error-msg {
		font-size: var(--ds-text-xs);
		color: #8b2020;
		margin-top: var(--ds-space-1);
	}

	.card-ds {
		background: var(--ds-color-surface);
		border: 1px solid var(--ds-color-border);
		border-radius: var(--ds-radius-lg);
		box-shadow: var(--ds-shadow-sm);
		overflow: hidden;
	}

	.card-ds-body {
		padding: var(--ds-space-6);
	}
	.card-ds-header {
		padding: var(--ds-space-4) var(--ds-space-6);
		border-bottom: 1px solid var(--ds-color-border);
		background: var(--ds-color-bg);
		font-weight: 700;
	}
	.card-ds-footer {
		padding: var(--ds-space-4) var(--ds-space-6);
		border-top: 1px solid var(--ds-color-border);
		background: var(--ds-color-bg);
	}
	.card-ds-horizontal {
		display: flex;
		align-items: stretch;
	}
	.card-ds-img-placeholder {
		width: 120px;
		background: var(--ds-color-surface-2);
		display: flex;
		align-items: center;
		justify-content: center;
		color: var(--ds-color-text-muted);
		font-size: var(--ds-text-xs);
		flex-shrink: 0;
	}

	.alert-ds {
		display: flex;
		align-items: flex-start;
		gap: var(--ds-space-3);
		padding: var(--ds-space-4) var(--ds-space-5);
		border-radius: var(--ds-radius-md);
		border: 1.5px solid transparent;
		font-size: var(--ds-text-sm);
		line-height: var(--ds-line-normal);
	}

	.alert-ds-icon {
		font-size: 1rem;
		flex-shrink: 0;
		margin-top: 1px;
	}
	.alert-ds-title {
		font-weight: 700;
		margin-bottom: var(--ds-space-1);
	}
	.alert-info {
		background: #e8f1fb;
		border-color: #adc9ef;
		color: #1a3f6f;
	}
	.alert-success {
		background: #e6f4e5;
		border-color: #a8d5a2;
		color: #1e5219;
	}
	.alert-warning {
		background: #fff5dc;
		border-color: #f5d589;
		color: #6b4200;
	}
	.alert-error {
		background: #fdecec;
		border-color: #f0aaaa;
		color: #6b1a1a;
	}

	.toast-ds {
		display: flex;
		align-items: flex-start;
		gap: var(--ds-space-3);
		padding: var(--ds-space-4) var(--ds-space-5);
		border-radius: var(--ds-radius-lg);
		border: 1px solid var(--ds-color-border);
		background: var(--ds-color-surface);
		box-shadow: var(--ds-shadow-md);
		font-size: var(--ds-text-sm);
		max-width: 360px;
		position: relative;
	}
	.toast-ds-close {
		position: absolute;
		top: var(--ds-space-3);
		right: var(--ds-space-4);
		background: none;
		border: none;
		cursor: pointer;
		color: var(--ds-color-text-muted);
		font-size: var(--ds-text-base);
		line-height: 1;
		padding: 0;
	}
	.toast-dot {
		width: 8px;
		height: 8px;
		border-radius: 50%;
		flex-shrink: 0;
		margin-top: 5px;
	}
	.toast-dot-success {
		background: #3a8a33;
	}
	.toast-dot-warning {
		background: #cc8800;
	}

	.codeblock-ds {
		position: relative;
		background: var(--ds-color-text);
		border-radius: var(--ds-radius-lg);
		overflow: hidden;
	}

	.codeblock-ds-bar {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: var(--ds-space-3) var(--ds-space-5);
		background: rgba(255, 255, 255, 0.05);
		border-bottom: 1px solid rgba(255, 255, 255, 0.08);
	}

	.codeblock-ds-lang {
		font-family: var(--ds-font-mono);
		font-size: var(--ds-text-xs);
		color: rgba(255, 255, 255, 0.45);
		letter-spacing: 0.06em;
	}

	.codeblock-ds-dots {
		display: flex;
		gap: 6px;
	}

	.codeblock-ds-dots span {
		width: 10px;
		height: 10px;
		border-radius: 50%;
	}

	.codeblock-ds pre {
		margin: 0;
		padding: var(--ds-space-5) var(--ds-space-6);
		overflow-x: auto;
		font-family: var(--ds-font-mono);
		font-size: var(--ds-text-sm);
		line-height: 1.8;
		color: #e8e0d8;
	}

	.codeblock-ds pre code {
		background: none;
		color: inherit;
		padding: 0;
		font-size: inherit;
		border-radius: 0;
	}

	.code-comment {
		color: rgba(255, 255, 255, 0.32);
	}

	.code-key {
		color: #b85f5b;
	}

	.code-val {
		color: #c5d8c6;
	}

	.code-str {
		color: #f3d9af;
	}

	.codeblock-copy {
		font-family: var(--ds-font-sans);
		font-size: var(--ds-text-xs);
		color: rgba(255, 255, 255, 0.5);
		background: none;
		border: 1px solid rgba(255, 255, 255, 0.16);
		border-radius: var(--ds-radius-sm);
		padding: 0.2em 0.6em;
		cursor: pointer;
		transition: all 0.2s ease;
	}

	.codeblock-copy:hover {
		color: #fff;
		border-color: rgba(255, 255, 255, 0.4);
	}

	.breadcrumb-ds {
		display: flex;
		align-items: center;
		flex-wrap: wrap;
		gap: var(--ds-space-1);
		list-style: none;
		padding: 0;
		margin: 0;
		font-size: var(--ds-text-sm);
	}
	.breadcrumb-ds li {
		display: flex;
		align-items: center;
		gap: var(--ds-space-1);
	}
	.breadcrumb-ds a {
		color: var(--ds-color-text-muted);
		text-decoration: none;
	}
	.breadcrumb-ds a:hover {
		color: var(--ds-color-danger);
	}
	.breadcrumb-ds .sep {
		color: var(--ds-color-border);
		font-size: var(--ds-text-xs);
	}
	.breadcrumb-ds .current {
		color: var(--ds-color-text);
		font-weight: 600;
	}

	.avatar-ds {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		font-family: var(--ds-font-sans);
		font-weight: 700;
		background: var(--ds-color-surface-2);
		color: var(--ds-color-primary);
		border: 1.5px solid var(--ds-color-border);
		overflow: hidden;
		flex-shrink: 0;
	}
	.avatar-circle {
		border-radius: 50%;
	}
	.avatar-md {
		width: 40px;
		height: 40px;
		font-size: var(--ds-text-sm);
	}
	.avatar-group {
		display: flex;
	}
	.avatar-group .avatar-ds {
		margin-left: -8px;
		border: 2px solid var(--ds-color-surface);
		box-shadow: var(--ds-shadow-sm);
	}
	.avatar-group .avatar-ds:first-child {
		margin-left: 0;
	}

	.tooltip-ds-wrap {
		position: relative;
		display: inline-flex;
	}
	.tooltip-ds {
		position: absolute;
		bottom: calc(100% + 8px);
		left: 50%;
		transform: translateX(-50%);
		background: var(--ds-color-text);
		color: #fff;
		font-size: var(--ds-text-xs);
		font-family: var(--ds-font-sans);
		padding: 0.3em 0.7em;
		border-radius: var(--ds-radius-sm);
		white-space: nowrap;
		pointer-events: none;
		opacity: 0;
		transition:
			opacity 0.15s ease,
			transform 0.15s ease;
		transform: translateX(-50%) translateY(4px);
		z-index: 200;
	}
	.tooltip-ds::after {
		content: '';
		position: absolute;
		top: 100%;
		left: 50%;
		transform: translateX(-50%);
		border: 5px solid transparent;
		border-top-color: var(--ds-color-text);
	}
	.tooltip-ds-wrap:hover .tooltip-ds {
		opacity: 1;
		transform: translateX(-50%) translateY(0);
	}
	.tooltip-bottom .tooltip-ds {
		bottom: auto;
		top: calc(100% + 8px);
		transform: translateX(-50%) translateY(-4px);
	}
	.tooltip-bottom .tooltip-ds::after {
		top: auto;
		bottom: 100%;
		border-top-color: transparent;
		border-bottom-color: var(--ds-color-text);
	}

	.progress-ds-wrap {
		display: flex;
		flex-direction: column;
		gap: var(--ds-space-2);
	}
	.progress-ds-label {
		display: flex;
		justify-content: space-between;
		font-size: var(--ds-text-xs);
		font-weight: 600;
		color: var(--ds-color-text);
	}
	.progress-ds-track {
		width: 100%;
		height: 8px;
		background: var(--ds-color-surface-2);
		border-radius: var(--ds-radius-pill);
		overflow: hidden;
	}
	.progress-ds-bar {
		height: 100%;
		border-radius: var(--ds-radius-pill);
		background: var(--ds-color-danger);
		transition: width 0.6s cubic-bezier(0.4, 0, 0.2, 1);
	}
	.progress-ds-bar.bar-success {
		background: #3a8a33;
	}

	.steps-ds {
		display: flex;
		align-items: flex-start;
		gap: 0;
		position: relative;
	}
	.step-ds {
		display: flex;
		flex-direction: column;
		align-items: center;
		flex: 1;
		position: relative;
	}
	.step-ds:not(:last-child)::after {
		content: '';
		position: absolute;
		top: 15px;
		left: 50%;
		width: 100%;
		height: 2px;
		background: var(--ds-color-border);
		z-index: 0;
	}
	.step-ds.complete:not(:last-child)::after {
		background: var(--ds-color-danger);
	}
	.step-node {
		width: 30px;
		height: 30px;
		border-radius: 50%;
		border: 2px solid var(--ds-color-border);
		background: var(--ds-color-surface);
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: var(--ds-text-xs);
		font-weight: 700;
		color: var(--ds-color-text-muted);
		position: relative;
		z-index: 1;
	}
	.step-ds.complete .step-node {
		background: var(--ds-color-danger);
		border-color: var(--ds-color-danger);
		color: #fff;
	}
	.step-ds.active .step-node {
		border-color: var(--ds-color-danger);
		color: var(--ds-color-danger);
		box-shadow: 0 0 0 4px rgba(184, 95, 91, 0.2);
	}
	.step-label {
		margin-top: var(--ds-space-2);
		font-size: var(--ds-text-xs);
		font-weight: 600;
		color: var(--ds-color-text-muted);
		text-align: center;
	}
	.step-ds.active .step-label {
		color: var(--ds-color-danger);
	}
	.step-ds.complete .step-label {
		color: var(--ds-color-text);
	}

	.ds-footer {
		border-top: 1px solid var(--ds-color-border);
		padding-top: var(--ds-space-8);
		margin-top: var(--ds-space-8);
	}

	@media (max-width: 991px) {
		.ds-section-title::after {
			display: none;
		}
	}
</style>
