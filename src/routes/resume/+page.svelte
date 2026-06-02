<script lang="ts">
  import Seo from "$lib/components/Seo.svelte";
</script>

<Seo title="Satvik Matta – Resume" description="Resume for Satvik Matta." />

<section class="layout-md py-12">
  <h2 class="heading2">Resume</h2>

  <div class="text-lg">
    <div class="resume-item">
      <div class="resume-head">
        <h3 class="text-black text-lg">NVIDIA</h3>
        <div class="resume-dates">Incoming</div>
      </div>
      <p class="text-neutral-600">Distributed training @ GEAR</p>
    </div>

    <div class="resume-item">
      <div class="resume-head">
        <h3 class="text-black text-lg">Google DeepMind</h3>
        <div class="resume-dates">Jan 2026 - Present</div>
      </div>
      <p class="text-neutral-600">3D coral reef reconstruction</p>
    </div>

    <div class="resume-item">
      <div class="resume-head">
        <h3 class="text-black text-lg">Hao AI Lab, UCSD</h3>
        <div class="resume-dates">Sep 2025 - Present</div>
      </div>
      <p class="text-neutral-600">World models research</p>
    </div>

    <div class="resume-item">
      <div class="resume-head">
        <h3 class="text-black text-lg">Meta</h3>
        <div class="resume-dates">Jun 2025 - Sep 2025</div>
      </div>
      <p class="text-neutral-600">Ads ranking with RLHF</p>
    </div>

    <div class="resume-item">
      <div class="resume-head">
        <h3 class="text-black text-lg">NVIDIA</h3>
        <div class="resume-dates">Sep 2025 - Dec 2025</div>
      </div>
      <p class="text-neutral-600">Disaggregated serving @ NIMs and Dynamo</p>
    </div>
  </div>
</section>

<style lang="postcss">
  .resume-item {
    @apply mb-5;
  }

  .resume-head {
    @apply flex items-end justify-between gap-3;
  }

  .resume-dates {
    @apply text-neutral-500 mb-0.5 whitespace-nowrap;
  }
</style>
