<script lang="ts">
  import Seo from "$lib/components/Seo.svelte";
  import WritingList from "./WritingList.svelte";
  import writing from "./writing.yaml";

  type WritingItem = {
    title: string;
    date: Date;
    summary: string;
    link: string;
  };

  const essays = writing.essays as WritingItem[];
</script>

<Seo title="Satvik Matta – Writing" description="Writing by Satvik Matta." />

<section class="layout-md py-12">
  <h2 class="heading2">Writing</h2>

  <div class="space-y-4">
    <h3 class="text-2xl text-black">Essays</h3>
    <WritingList data={essays} />
  </div>
</section>
