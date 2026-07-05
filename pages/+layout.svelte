<script>
	import '@evidence-dev/tailwind/fonts.css';
	import '../app.css';
	import { EvidenceDefaultLayout } from '@evidence-dev/core-components';
	export let data;
</script>

<EvidenceDefaultLayout {data} builtWithEvidence={false} title="Who Pays for the FAIR Plan" homePageName="Intro" homePageUrl="/" hideBreadcrumbs=true >
	<slot slot="content" />
</EvidenceDefaultLayout>
