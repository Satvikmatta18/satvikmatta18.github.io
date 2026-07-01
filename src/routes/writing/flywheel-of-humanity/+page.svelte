<script lang="ts">
  import Seo from "$lib/components/Seo.svelte";
  import Markdown from "$lib/components/Markdown.svelte";

  const source = `
## Where we are

Civilizations can be ranked by how much energy they command. A Type I civilization controls its planet, Type II controls its star, and Type III controls its galaxy. Humanity is still around Type 0.7: powerful, but fragile. We do not fully control energy, climate, disease, AI risk, or planetary survival. The mission of the next century is to reach Type I and keep going until humanity is much harder to wipe out.

## The compounding loop

You can already see the shape of how we get there in what Elon Musk is building. Right now it is three things at once: an internet company (Starlink), an AI company (xAI), and a compute and neocloud business (Colossus). The next stage is vertical, going up. The capital from those businesses funds infrastructure in space. That same capital flywheel funds robotics, Optimus, to build a physical labor force. Once the robots become genuinely productive and profitable, their output feeds back into the flywheel.

The final stage is pointing all that compute and capital at advancing humanity itself: scaling xAI models, funding crazy ventures, and pushing research like Neuralink. That loop, internet and AI, capital, space, robots, compute, human advancement, more capital, is one visible example of a much bigger pattern.

That bigger pattern is the flywheel of humanity: each frontier makes the next one easier, and the gains feed back into the whole system. Every frontier below is one turn of that wheel, and each is stuck on a specific bottleneck. Here is where each one is now, what is blocking it, and what comes next.

### 1. AI & Compute

- Bottleneck: scaling laws slowing, lack of memory, agency, reliability, real-world grounding
- Research: agents, continual learning, long-term memory, test-time compute, world models, AI scientists, inference optimization
- Next: chatbots, agents, AI engineers, AI scientists
- Companies: OpenAI, Anthropic, Google DeepMind, Meta, xAI, NVIDIA, AMD, TSMC, CoreWeave, Crusoe, Lambda, Etched, Cerebras, Groq, Tenstorrent

### 2. Energy

- Bottleneck: not enough cheap, clean, 24/7 power
- Research: advanced nuclear, geothermal, fusion, storage, grid AI, space solar
- Next: nuclear/geothermal now, fusion in 2030s, space solar later
- Companies: Oklo, Kairos, X-energy, TerraPower, Fervo, Quaise, Sage, Eavor, Commonwealth Fusion, Helion, Type One, TAE, Zap, Form Energy, Antora, Energy Dome

### 3. Space

- Bottleneck: launch cost, orbital logistics, radiation, life support, habitats
- Research: reusable rockets, orbital refueling, lunar cargo, space manufacturing, orbital data centers, asteroid mining
- Next: cheap launch, Moon bases, orbital industry, asteroid resources, Mars
- Companies: SpaceX, Rocket Lab, Stoke, Blue Origin, Varda, Redwire, Sierra Space, Starcloud, Axiom, Lonestar, Aetherflux, AstroForge, Karman+, TransAstra, Firefly, Intuitive Machines, Astrobotic

### 4. Physical AI / Robotics

- Bottleneck: no internet-scale robot dataset, dexterous hands, reliability, cost
- Research: robot foundation models, VLA models, teleoperation, simulation, tactile sensing, humanoid hardware
- Next: factories, warehouses, data centers, hospitals, homes, space
- Companies: Physical Intelligence, Skild AI, FieldAI, Google DeepMind Robotics, NVIDIA GR00T, Tesla Optimus, Figure, Apptronik, 1X, Agility, Boston Dynamics, Unitree, Machina Labs, Bedrock, Gecko Robotics

### 5. Automated Science

- Bottleneck: research loop is too slow; experiments take months and data is messy
- Research: AI scientists, self-driving labs, robotic wet labs, cloud labs, automated materials discovery, AI-designed proteins/materials
- Next: AI proposes, robots test, data improves model, better experiments
- Companies: Periodic Labs, Lila Sciences, Medra, Chemify, Atinary, Kebotix, Emerald Cloud Lab, Strateos, Automata, Opentrons, Citrine, Orbital Materials, CuspAI, SandboxAQ, Isomorphic, Recursion, Xaira, Generate:Biomedicines

### 6. Longevity & Health

- Bottleneck: aging is slow to measure; biomarkers are not fully proven
- Research: partial reprogramming, senolytics, autophagy, immune rejuvenation, stem cells, aging clocks, dog longevity
- Next: age-related disease treatments, organ-specific rejuvenation, whole-body aging control
- Companies: Altos Labs, NewLimit, Retro Biosciences, Life Biosciences, Rubedo, BioAge, Calico, Cambrian, Loyal, Rejuvenate Bio, Turn Bio

### 7. Gene Editing

- Bottleneck: delivery, off-target edits, immune response, long-term safety
- Research: CRISPR, base editing, prime editing, RNA editing, in-vivo editing, AI-designed editors, embryo screening
- Next: single-gene cures, in-vivo editing, personalized rare disease cures, inherited disease prevention
- Companies: CRISPR Therapeutics, Vertex, Intellia, Editas, Beam, Prime Medicine, Verve, Mammoth, Scribe, Arbor, Metagenomi, Profluent, Dyno, Aera, Orchid, Nucleus, Genomic Prediction, Preventive

### 8. Regenerative Medicine & Organ Replacement

- Bottleneck: organ shortages, immune rejection, vascularization, scale
- Research: gene-edited pig organs, stem-cell therapies, bioprinting, iPSC cells, organ preservation, tissue engineering
- Next: better preservation, pig organs, cell therapies, tissue patches, custom organs
- Companies: eGenesis, United Therapeutics, Revivicor, Aspect Biosystems, OrganOx, TransMedics, Paragonix, BlueRock, Sana, Vertex Cell Therapy, LyGenesis, bit.bio, BICO / CELLINK

### 9. Brain-Computer Interfaces

- Bottleneck: bandwidth vs safety; invasive is powerful but risky, non-invasive is safer but weaker
- Research: thought-to-text, cursor control, speech restoration, robotic arms, vision restoration, neural decoding, brain mapping
- Next: medical restoration, robotic limbs, vision, cognitive prosthetics, human-AI link
- Companies: Neuralink, Synchron, Paradromics, Precision Neuroscience, Blackrock Neurotech, Science Corp, Forest Neurotech, Motif, E11 Bio, Cortical Labs, FinalSpark

### 10. Biosecurity & Resilience

- Bottleneck: coordination, funding, and deployment before disaster happens
- Research: pathogen surveillance, DNA synthesis screening, wastewater monitoring, rapid vaccines, AI threat detection, broad antivirals, planetary defense
- Next: biosecurity becomes like cybersecurity: always-on monitoring and fast response
- Companies / orgs: Perimeter, Red Queen Bio, Valthos, Portal Biotech, SecureDNA, Blueprint Biosecurity, BlueDot, Karius, Mammoth Diagnostics, Sherlock Biosciences, Moderna, BioNTech, HelixNano, Centivax, NASA DART, ESA Hera

### 11. Food, Climate, and the Biosphere

- Bottleneck: climate volatility, water scarcity, soil degradation, fragile food supply, biodiversity loss
- Research: CRISPR crops, microbial fertilizer, precision fermentation, cultivated meat, carbon removal, direct air capture, enhanced weathering, genetic rescue
- Next: resilient food, autonomous farming, synthetic proteins, carbon removal, ecosystem repair
- Companies: Pairwise, Inari, Cibus, Tropic Biosciences, Pivot Bio, Loam Bio, Andes, Sound Agriculture, Perfect Day, The EVERY Company, Formo, Remilk, UPSIDE Foods, GOOD Meat, Wildtype, BlueNalu, Heirloom, Climeworks, Charm, Mati Carbon, Lithos, Vaulted Deep, Colossal, Revive & Restore

Across every frontier, the same issue appears:

The world changes slowly because experiments are slow.

The unlock is a closed-loop civilization:

AI generates ideas -> labs test them -> robots build infrastructure -> energy powers it -> space expands it -> biology keeps humans alive -> safety systems prevent collapse -> data feeds back into better AI.

The future of humanity is a stack:

- Energy gives power.
- Compute gives intelligence.
- Robots give intelligence bodies.
- Automated science speeds discovery.
- Longevity and gene editing make humans harder to kill.
- BCI preserves human agency.
- Biosecurity protects civilization.
- Climate tech stabilizes Earth.
- Space removes Earth as our only home.
`;
</script>

<Seo
  title="Satvik Matta – Flywheel of Humanity"
  description="A framework for the flywheel that can make humanity harder to wipe out."
/>

<section class="layout-md py-12 essay-page">
  <header class="essay-header">
    <p class="essay-date">June 30, 2026</p>
    <h2 class="heading2">Flywheel of Humanity</h2>
  </header>

  <Markdown {source} />
</section>

<style lang="postcss">
  .essay-page {
    @apply text-[17px] leading-7 text-neutral-700;
  }

  .essay-header {
    @apply mb-8 border-b border-neutral-200 pb-5;
  }

  .essay-date {
    @apply mb-2 text-sm text-neutral-500;
  }

  :global(.essay-page .md-output h2) {
    @apply mt-10 mb-3 text-xl font-semibold text-black;
  }

  :global(.essay-page .md-output h3) {
    @apply mt-8 mb-3 text-lg font-semibold text-black;
  }

  :global(.essay-page .md-output ul) {
    @apply mb-6 ml-5 list-disc space-y-1;
  }

  :global(.essay-page .md-output li) {
    @apply pl-1;
  }

  :global(.essay-page .md-output p) {
    @apply mb-5;
  }
</style>
