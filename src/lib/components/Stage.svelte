<script lang="ts">
  import { fade } from 'svelte/transition';
  import { game } from '../store.svelte';
  import { pointPos, RING_R, VIEW } from '../geometry';
  import { POINTS } from '../game/board';
  import BoardLines from './BoardLines.svelte';
  import Stone from './Stone.svelte';

  const targets = $derived(new Set(game.targets));

  /** Active (dragged/selected) stone renders last = on top. */
  const ordered = $derived(
    [...game.stones].sort(
      (a, b) =>
        Number(a.id === game.activeStone) - Number(b.id === game.activeStone),
    ),
  );

  const winPath = $derived.by(() => {
    const line = game.state.winLine;
    if (!line) return null;
    const [a, b, c] = line.map((i) => pointPos(i, game.variant));
    // A rim-arc win on the wheel follows the rim, not a chord. Arc mills are
    // stored in ring order, so trace two 45° clockwise segments a → b → c.
    if (game.variant === 'roman' && !line.includes(8)) {
      const r = RING_R;
      return `M ${a.x} ${a.y} A ${r} ${r} 0 0 1 ${b.x} ${b.y} A ${r} ${r} 0 0 1 ${c.x} ${c.y}`;
    }
    return `M ${a.x} ${a.y} L ${b.x} ${b.y} L ${c.x} ${c.y}`;
  });
</script>

<svg
  viewBox="0 0 {VIEW.w} {VIEW.h}"
  preserveAspectRatio="xMidYMid meet"
  aria-label="align3 board"
>
  <defs>
    <!-- chisel wobble: slight, controlled -->
    <filter id="carve" x="-8%" y="-8%" width="116%" height="116%">
      <feTurbulence type="fractalNoise" baseFrequency="0.06 0.09" numOctaves="2" seed="7" result="n" />
      <feDisplacementMap in="SourceGraphic" in2="n" scale="1.1" />
    </filter>
    <!-- chalk wobble: looser hand, two seeds so the re-chalk pass doesn't trace the first -->
    <filter id="chalk" x="-8%" y="-8%" width="116%" height="116%">
      <feTurbulence type="fractalNoise" baseFrequency="0.05 0.08" numOctaves="2" seed="7" result="n" />
      <feDisplacementMap in="SourceGraphic" in2="n" scale="2.2" />
    </filter>
    <filter id="chalk2" x="-8%" y="-8%" width="116%" height="116%">
      <feTurbulence type="fractalNoise" baseFrequency="0.05 0.08" numOctaves="2" seed="13" result="n" />
      <feDisplacementMap in="SourceGraphic" in2="n" scale="2.6" />
    </filter>
  </defs>

  {#key game.variant}
    <g transition:fade={{ duration: 350 }}>
      <BoardLines variant={game.variant} />
    </g>
  {/key}

  <!-- markers share the board's line quality; hit targets stay unfiltered -->
  <g filter={game.variant === 'roman' ? 'url(#carve)' : 'url(#chalk)'}>
    {#each POINTS as i (i)}
      {@const p = pointPos(i, game.variant)}
      <g class="point" style="transform: translate({p.x}px, {p.y}px)">
        {#if game.variant === 'roman'}
          <circle r="3" class="socket" />
          <circle r="1.9" class="dimple" class:glow={targets.has(i)} />
        {:else}
          <circle r="2.5" class="chalk-ring" class:lit={targets.has(i)} />
          <circle r="0.85" class="chalk-dot" class:glow={targets.has(i)} />
        {/if}
      </g>
    {/each}
  </g>

  {#each POINTS as i (i)}
    {@const p = pointPos(i, game.variant)}
    <g class="point" style="transform: translate({p.x}px, {p.y}px)">
      {#if targets.has(i)}
        <circle r="3.4" class="halo" />
      {/if}
      <circle
        r="6"
        fill="transparent"
        role="button"
        tabindex="-1"
        aria-label="board point {i}"
        onclick={() => game.tapPoint(i)}
      />
    </g>
  {/each}

  {#if winPath}
    <path d={winPath} class="win-line" pathLength="1" />
  {/if}

  {#each ordered as stone (stone.id)}
    <Stone {stone} />
  {/each}
</svg>

<style>
  svg {
    display: block;
    width: 100%;
    height: 100%;
    touch-action: none;
  }

  .point {
    transition: transform 0.45s cubic-bezier(0.22, 1, 0.36, 1);
  }

  .socket {
    fill: none;
    stroke: var(--line-soft);
    stroke-width: 0.55;
    opacity: 0.9;
  }

  .dimple {
    fill: var(--dimple);
    transition: fill 0.2s;
  }
  .dimple.glow {
    fill: var(--glow);
  }

  .chalk-ring {
    fill: none;
    stroke: var(--line);
    stroke-width: 0.65;
    opacity: 0.8;
    transition: stroke 0.2s;
  }
  .chalk-ring.lit {
    stroke: var(--glow);
    opacity: 1;
  }

  .chalk-dot {
    fill: var(--line);
    opacity: 0.85;
    transition: fill 0.2s;
  }
  .chalk-dot.glow {
    fill: var(--glow);
  }

  .halo {
    fill: none;
    stroke: var(--glow);
    stroke-width: 0.5;
    opacity: 0.85;
    animation: breathe 1.3s ease-in-out infinite;
  }

  @keyframes breathe {
    0%,
    100% {
      transform: scale(1);
      opacity: 0.85;
    }
    50% {
      transform: scale(1.22);
      opacity: 0.45;
    }
  }

  .win-line {
    fill: none;
    stroke: var(--glow);
    stroke-width: 2.4;
    stroke-linecap: round;
    stroke-dasharray: 1;
    stroke-dashoffset: 0;
    animation: draw 0.7s cubic-bezier(0.22, 1, 0.36, 1);
  }

  @keyframes draw {
    from {
      stroke-dashoffset: 1;
    }
    to {
      stroke-dashoffset: 0;
    }
  }

  @media (prefers-reduced-motion: reduce) {
    .point {
      transition: none;
    }
    .halo {
      animation: none;
    }
    .win-line {
      animation: none;
    }
  }
</style>
