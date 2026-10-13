<script lang="ts">
  import { onMount } from "svelte";

  type Point = { x: number; y: number };
  const colors = [
    { top: "#80a18a", front: "#56775f", side: "#678b71", stud: "#a2bcaa" },
    { top: "#e0a875", front: "#b87949", side: "#c99160", stud: "#efc59f" },
    { top: "#829ebc", front: "#536f8e", side: "#6885a4", stud: "#adc1d5" },
    { top: "#d9c891", front: "#ac975d", side: "#c0ad72", stud: "#eee1b7" },
  ];
  const source = { x: 360, y: 287 };
  const shoulder = { x: 247, y: 219 };
  const home = { x: 348, y: 190 };
  const stepDuration = 3600;
  const assemblyDuration = stepDuration * 8;
  const cycleDuration = assemblyDuration + 2800;
  const studs = Array.from({ length: 8 }, (_, index) => {
    const u = (index % 4) + 0.5;
    const v = Math.floor(index / 4) + 0.5;
    return { x: (u - v) * 13 - 13, y: (u + v) * 6.5 - 19.5 };
  });
  const bricks = Array.from({ length: 8 }, (_, index) => ({
    x: index % 2 === 0 ? 478 : 504,
    y: (index % 2 === 0 ? 274 : 261) - (Math.floor(index / 2) + 1) * 14,
    color: colors[Math.floor(index / 2) % colors.length],
  }));

  let container: HTMLElement;
  let elapsed = stepDuration * 2.54;
  let paused = false;
  let reducedMotion = false;
  let syncAnimation = () => {};

  const smooth = (value: number) => {
    const t = Math.max(0, Math.min(1, value));
    return t * t * (3 - 2 * t);
  };
  const mix = (a: Point, b: Point, value: number): Point => {
    const t = smooth(value);
    return { x: a.x + (b.x - a.x) * t, y: a.y + (b.y - a.y) * t };
  };
  const above = (point: Point): Point => ({ x: point.x, y: point.y - 68 });
  const grip = (point: Point): Point => ({ x: point.x, y: point.y - 16 });

  function getScene(time: number) {
    const cycle = time % cycleDuration;
    const index = Math.min(7, Math.floor(cycle / stepDuration));
    const phase = (cycle % stepDuration) / stepDuration;
    const destination = bricks[index];
    const completed = cycle >= assemblyDuration;
    let wrist = home;
    let opening = 1;
    if (completed) {
      wrist = mix(above(bricks[7]), home, (cycle - assemblyDuration) / 700);
    } else if (phase < 0.18) {
      wrist = mix(
        index === 0 ? home : above(bricks[index - 1]),
        above(source),
        phase / 0.18
      );
    } else if (phase < 0.3) {
      wrist = mix(above(source), grip(source), (phase - 0.18) / 0.12);
    } else if (phase < 0.36) {
      wrist = grip(source);
      opening = 1 - smooth((phase - 0.3) / 0.06);
    } else if (phase < 0.49) {
      wrist = mix(grip(source), above(source), (phase - 0.36) / 0.13);
      opening = 0;
    } else if (phase < 0.72) {
      wrist = mix(above(source), above(destination), (phase - 0.49) / 0.23);
      wrist.y -= Math.sin(((phase - 0.49) / 0.23) * Math.PI) * 14;
      opening = 0;
    } else if (phase < 0.84) {
      wrist = mix(above(destination), grip(destination), (phase - 0.72) / 0.12);
      opening = 0;
    } else if (phase < 0.91) {
      wrist = grip(destination);
      opening = smooth((phase - 0.84) / 0.07);
    } else {
      wrist = mix(grip(destination), above(destination), (phase - 0.91) / 0.09);
    }
    return {
      wrist,
      opening,
      index,
      count: completed ? 8 : index + (phase >= 0.865 ? 1 : 0),
      holding: !completed && phase >= 0.33 && phase < 0.865,
      sourceVisible: !completed && phase < 0.33,
      fade: completed ? 1 - smooth((cycle - assemblyDuration - 2100) / 600) : 1,
    };
  }

  function elbowFor(wrist: Point): Point {
    const dx = wrist.x - shoulder.x;
    const dy = wrist.y - shoulder.y;
    const distance = Math.hypot(dx, dy);
    const upper = 143;
    const forearm = 133;
    const along =
      (upper * upper - forearm * forearm + distance * distance) /
      (2 * distance);
    const height = Math.sqrt(Math.max(0, upper * upper - along * along));
    return {
      x: shoulder.x + (along * dx + height * dy) / distance,
      y: shoulder.y + (along * dy - height * dx) / distance,
    };
  }

  $: scene = getScene(reducedMotion ? assemblyDuration + 900 : elapsed);
  $: elbow = elbowFor(scene.wrist);
  $: visibleBricks = [
    ...bricks.slice(0, scene.count),
    ...(scene.sourceVisible
      ? [{ ...source, color: bricks[scene.index].color }]
      : []),
    ...(scene.holding
      ? [
          {
            x: scene.wrist.x,
            y: scene.wrist.y + 16,
            color: bricks[scene.index].color,
          },
        ]
      : []),
  ];

  function togglePause() {
    paused = !paused;
    syncAnimation();
  }

  function replay() {
    elapsed = 0;
    paused = false;
    syncAnimation();
  }

  onMount(() => {
    const preference = window.matchMedia("(prefers-reduced-motion: reduce)");
    let frame = 0;
    let previous = 0;
    let inView = true;
    let disposed = false;
    reducedMotion = preference.matches;

    const tick = (now: number) => {
      if (previous) elapsed += Math.min(now - previous, 64);
      previous = now;
      frame = requestAnimationFrame(tick);
    };
    syncAnimation = () => {
      cancelAnimationFrame(frame);
      previous = 0;
      if (
        !disposed &&
        !paused &&
        !reducedMotion &&
        inView &&
        !document.hidden
      ) {
        frame = requestAnimationFrame(tick);
      }
    };
    const onPreference = () => {
      reducedMotion = preference.matches;
      syncAnimation();
    };
    const observer = new IntersectionObserver(
      ([entry]) => {
        inView = entry.isIntersecting;
        syncAnimation();
      },
      { threshold: 0.05 }
    );
    observer.observe(container);
    preference.addEventListener("change", onPreference);
    document.addEventListener("visibilitychange", syncAnimation);
    syncAnimation();

    return () => {
      disposed = true;
      cancelAnimationFrame(frame);
      observer.disconnect();
      preference.removeEventListener("change", onPreference);
      document.removeEventListener("visibilitychange", syncAnimation);
    };
  });
</script>

<figure class="robot-builder" bind:this={container}>
  <div class="scene">
    <svg
      viewBox="0 0 720 360"
      role="img"
      aria-labelledby="robot-title robot-description"
    >
      <title id="robot-title">One brick at a time</title>
      <desc id="robot-description"
        >An animated illustration of a robot arm picking up colorful, studded
        building bricks and carefully assembling a small tower.</desc
      >
      <defs>
        <linearGradient id="robot-arm" x1="0" y1="0" x2="0.7" y2="1">
          <stop offset="0" stop-color="#64716d" />
          <stop offset="1" stop-color="#394641" />
        </linearGradient>
        <linearGradient id="robot-floor" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0" stop-color="#f5f6f2" />
          <stop offset="1" stop-color="#edf0e9" />
        </linearGradient>
        <radialGradient id="robot-shadow">
          <stop offset="0" stop-color="#293b2d" stop-opacity="0.15" />
          <stop offset="1" stop-color="#293b2d" stop-opacity="0" />
        </radialGradient>
        <clipPath id="robot-floor-clip">
          <path d="M112 244 393 104 649 232 368 372Z" />
        </clipPath>
      </defs>

      <path
        d="M112 244 393 104 649 232 368 372Z"
        fill="url(#robot-floor)"
        opacity="0.58"
      />
      <g
        clip-path="url(#robot-floor-clip)"
        stroke="#dce2d8"
        stroke-width="0.8"
        opacity="0.7"
      >
        {#each Array.from({ length: 20 }, (_, index) => index) as index}
          <path d={`M${20 + index * 28} 100 l560 280`} />
          <path d={`M${120 + index * 28} 90 l-560 280`} />
        {/each}
      </g>
      <ellipse cx="250" cy="285" rx="87" ry="30" fill="url(#robot-shadow)" />
      <ellipse cx="495" cy="277" rx="77" ry="29" fill="url(#robot-shadow)" />
      <ellipse cx="360" cy="306" rx="52" ry="18" fill="url(#robot-shadow)" />

      <!-- The two work areas are etched into the same isometric ground plane. -->
      <path
        d="M310 286 361 260 411 285 360 311Z"
        fill="none"
        stroke="#bdc9ba"
        stroke-dasharray="3 5"
        stroke-linecap="round"
      />
      <path
        d="M421 268 490 233 561 269 492 303Z"
        fill="#e1e6dc"
        stroke="#d0d9ca"
        stroke-width="1"
      />
      <path d="M421 268 492 303 561 269v5l-69 35-71-36Z" fill="#d1d9ca" />

      <!-- Fixed pedestal and shoulder. -->
      <path
        d="M202 276 246 254 291 276 247 298Z"
        fill="#b7c1b8"
        stroke="#9da99e"
      />
      <path d="M202 276v8l45 23v-9Z" fill="#8d9c90" />
      <path d="m247 298 44-22v8l-44 23Z" fill="#6f8173" />
      <path d="M229 231h36v45c0 12-36 12-36 0Z" fill="url(#robot-arm)" />
      <ellipse cx="247" cy="232" rx="18" ry="9" fill="#8c9890" />
      <path
        d="M235 246v24"
        stroke="#86958a"
        stroke-width="3"
        stroke-linecap="round"
      />
      <rect x="246" y="266" width="8" height="3" rx="1.5" fill="#b6d99b" />

      <!-- Every joint is solved from the gripper's position. -->
      <path
        d={`M${shoulder.x} ${shoulder.y} L${elbow.x} ${elbow.y}`}
        fill="none"
        stroke="#35433b"
        stroke-width="30"
        stroke-linecap="round"
      />
      <path
        d={`M${shoulder.x - 2} ${shoulder.y - 3} L${elbow.x - 2} ${
          elbow.y - 3
        }`}
        fill="none"
        stroke="#6c7a70"
        stroke-width="19"
        stroke-linecap="round"
      />
      <path
        d={`M${elbow.x} ${elbow.y} L${scene.wrist.x} ${scene.wrist.y - 6}`}
        fill="none"
        stroke="#3d4b43"
        stroke-width="23"
        stroke-linecap="round"
      />
      <path
        d={`M${elbow.x} ${elbow.y - 3} L${scene.wrist.x} ${scene.wrist.y - 9}`}
        fill="none"
        stroke="#7d8980"
        stroke-width="13"
        stroke-linecap="round"
      />
      <circle cx={shoulder.x} cy={shoulder.y} r="18" fill="#3a4840" />
      <circle cx={shoulder.x} cy={shoulder.y} r="10" fill="#99a59c" />
      <circle cx={shoulder.x} cy={shoulder.y} r="4" fill="#65746a" />
      <circle cx={elbow.x} cy={elbow.y} r="17" fill="#35433b" />
      <circle cx={elbow.x} cy={elbow.y} r="10" fill="#a6b0a7" />
      <circle cx={elbow.x} cy={elbow.y} r="4" fill="#6c7d6d" />

      <g opacity={scene.fade}>
        {#each visibleBricks as brick}
          <g transform={`translate(${brick.x} ${brick.y})`}>
            <path d="M-39-6.5 13 19.5v14L-39 7.5Z" fill={brick.color.front} />
            <path d="m13 19.5 26-13v14l-26 13.5Z" fill={brick.color.side} />
            <path
              d="M-13-19.5 39 6.5 13 19.5-39-6.5Z"
              fill={brick.color.top}
              stroke={brick.color.front}
              stroke-width="0.55"
              stroke-linejoin="round"
            />
            <path
              d="m-38-6 51 25"
              stroke="#fff"
              stroke-opacity="0.18"
              stroke-width="1"
            />
            {#each studs as stud}
              <path
                d={`M${stud.x - 4.8} ${
                  stud.y - 2.5
                }v2.5a4.8 2.4 0 0 0 9.6 0v-2.5Z`}
                fill={brick.color.side}
              />
              <ellipse
                cx={stud.x}
                cy={stud.y - 2.5}
                rx="4.8"
                ry="2.4"
                fill={brick.color.stud}
                stroke={brick.color.top}
                stroke-width="0.5"
              />
            {/each}
          </g>
        {/each}
      </g>

      <!-- Open jaws lower around the brick; closed jaws move with it. -->
      <g transform={`translate(${scene.wrist.x} ${scene.wrist.y})`}>
        <rect x="-12" y="-13" width="24" height="18" rx="6" fill="#3e4c43" />
        <path
          d={`M-5 0h${-25 - scene.opening * 8}v23h7 M5 0h${
            25 + scene.opening * 8
          }v23h-7`}
          fill="none"
          stroke="#344139"
          stroke-width="5"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
        <rect x="-5" y="-8" width="10" height="4" rx="2" fill="#9fc283" />
      </g>
    </svg>
  </div>
  <figcaption>
    <span class="caption"><span class="status-dot" /> One brick at a time.</span
    >
    <div class="controls">
      {#if reducedMotion}
        <span class="motion-note">Motion reduced</span>
      {:else}
        <button
          type="button"
          on:click={togglePause}
          aria-label={paused ? "Play robot animation" : "Pause robot animation"}
        >
          <svg viewBox="0 0 16 16" aria-hidden="true">
            {#if paused}<path
                d="m5 3 7 5-7 5Z"
                fill="currentColor"
              />{:else}<path
                d="M5 4v8M11 4v8"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
              />{/if}
          </svg>
          {paused ? "Play" : "Pause"}
        </button>
        <button
          type="button"
          class="replay"
          on:click={replay}
          aria-label="Replay robot animation"
          title="Replay animation"
        >
          <svg viewBox="0 0 16 16" fill="none" aria-hidden="true"
            ><path
              d="M3 6a5 5 0 1 1 0 4M3 2v4h4"
              stroke="currentColor"
              stroke-width="1.5"
              stroke-linecap="round"
              stroke-linejoin="round"
            /></svg
          >
        </button>
      {/if}
    </div>
  </figcaption>
</figure>

<style>
  .robot-builder {
    width: 100%;
    max-width: 720px;
    margin: 2.25rem auto 0;
    overflow: hidden;
    border: 1px solid #e7e9e2;
    border-radius: 16px;
    background: #fafbf8;
  }
  .scene {
    overflow: hidden;
  }
  .scene > svg {
    display: block;
    width: 100%;
    height: auto;
  }
  figcaption {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    min-height: 54px;
    padding: 6px 22px;
    color: #657063;
    background: #fafbf8;
    font-size: 12px;
    line-height: 1.4;
    letter-spacing: 0.01em;
  }
  .caption {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  .status-dot {
    width: 5px;
    height: 5px;
    flex-shrink: 0;
    border-radius: 50%;
    background: #93a687;
  }
  .controls {
    display: flex;
    align-items: center;
    gap: 5px;
  }
  button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 5px;
    min-height: 36px;
    padding: 7px 9px;
    border: 0;
    border-radius: 6px;
    background: transparent;
    color: #657063;
    font: inherit;
    cursor: pointer;
    transition: background 150ms, color 150ms;
  }
  button:hover {
    background: #edf0e8;
    color: #283b2b;
  }
  button:focus-visible {
    outline: 2px solid #6b8666;
    outline-offset: 2px;
  }
  button svg {
    width: 14px;
    height: 14px;
  }
  .replay {
    min-width: 36px;
  }
  .motion-note {
    font-size: 11px;
  }
  @media (max-width: 480px) {
    .robot-builder {
      border-radius: 12px;
      margin-top: 1.75rem;
    }
    .scene > svg {
      width: 125%;
      max-width: none;
      margin-left: -12.5%;
    }
    figcaption {
      padding: 6px 13px;
      font-size: 11px;
    }
  }
</style>
