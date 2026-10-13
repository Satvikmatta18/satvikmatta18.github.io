<script lang="ts">
  import Seo from "$lib/components/Seo.svelte";
  import RobotBuilder from "$lib/components/RobotBuilder.svelte";
</script>

<Seo
  title="Satvik Matta"
  description="Interested in solving hard problems. Studying mathematics and computer science at UC San Diego and working on robot foundation models at NVIDIA."
/>

<div class="layout-md home">
  <section class="intro" aria-labelledby="intro-heading">
    <h2 id="intro-heading">
      I’m interested in solving<br class="desktop-break" /> hard problems.
    </h2>

    <p>
      I’m studying mathematics and computer science at UC San Diego. Currently,
      I’m working on robot foundation models at
      <a class="link" href="https://www.nvidia.com/">NVIDIA</a>.
    </p>
  </section>

  <RobotBuilder />

  <section class="background" aria-label="Research and experience">
    <p>
      I also do research at
      <a class="link" href="https://deepmind.google/">Google DeepMind</a> on 3D
      reconstruction for coral reefs and at
      <a class="link" href="https://hao-ai-lab.github.io/">Hao AI Lab</a>
      on world models. Previously, I worked on post-training at
      <a class="link" href="https://about.meta.com/">Meta</a> and inference at
      <a class="link" href="https://developer.nvidia.com/nim">NVIDIA</a>.
    </p>
  </section>
</div>

<style>
  .home {
    font-size: 1.125rem;
    line-height: 1.65;
  }

  .intro {
    padding-top: 1.25rem;
  }

  h2 {
    margin-bottom: 1.5rem;
    color: #202522;
    font-size: clamp(2rem, 4.8vw, 2.85rem);
    font-weight: 500;
    line-height: 1.16;
    letter-spacing: -0.04em;
  }

  .intro p {
    max-width: 41rem;
  }

  .background {
    margin-top: 1.5rem;
  }

  @media (max-width: 480px) {
    .desktop-break {
      display: none;
    }
  }
</style>
