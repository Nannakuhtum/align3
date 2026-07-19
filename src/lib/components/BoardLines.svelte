<script lang="ts">
  import type { Variant } from '../game/types';
  import { pointPos, RING_R, VIEW } from '../geometry';

  let { variant }: { variant: Variant } = $props();

  const p = (i: number) => pointPos(i, variant);
  const seg = (a: number, b: number) =>
    `M ${p(a).x} ${p(a).y} L ${p(b).x} ${p(b).y}`;

  /** Kolam flourishes sit just outside the chalked square. */
  const CORNERS: [number, [number, number]][] = [
    [0, [-1, -1]],
    [2, [1, -1]],
    [4, [1, 1]],
    [6, [-1, 1]],
  ];
  const EDGES: [number, [number, number]][] = [
    [1, [0, -1]],
    [3, [1, 0]],
    [5, [0, 1]],
    [7, [-1, 0]],
  ];

  /** Chalk line ending: leaves the corner on the diagonal, hooks sideways. */
  const curl = (c: number, [dx, dy]: [number, number]) => {
    const { x, y } = p(c);
    const cx = x + dx * 4.6;
    const cy = y + dy * 4.6;
    return `M ${x + dx * 1.7} ${y + dy * 1.7} Q ${cx} ${cy} ${cx - dy * 2.4} ${cy + dx * 2.4}`;
  };
</script>

{#snippet wheel()}
  <circle cx={VIEW.cx} cy={VIEW.cy} r={RING_R} />
  <path d={seg(0, 4)} />
  <path d={seg(1, 5)} />
  <path d={seg(2, 6)} />
  <path d={seg(3, 7)} />
{/snippet}

{#snippet kattam()}
  <path d="{seg(0, 2)} {seg(2, 4)} {seg(4, 6)} {seg(6, 0)}" />
  <path d={seg(1, 5)} />
  <path d={seg(7, 3)} />
  <path d={seg(0, 4)} />
  <path d={seg(2, 6)} />
{/snippet}

{#if variant === 'roman'}
  <g class="carved" filter="url(#carve)">
    <circle cx={VIEW.cx} cy={VIEW.cy} r={RING_R + 6} class="worn" />
    <path class="crack" d="M 7 121 Q 15 113 20 104 T 28 90" />
    <path class="nick" d="M 24 45 l 3 2" />
    <path class="nick" d="M 79 99.5 l -2.7 1.8" />
    <path class="nick" d="M 71 41.5 l 2.4 -1.5" />
    <!-- groove = light under-edge offset toward the light, then the incision -->
    <g class="hi" transform="translate(0.38, 0.52)">{@render wheel()}</g>
    <g class="groove">{@render wheel()}</g>
    <circle cx={VIEW.cx} cy={VIEW.cy} r="4.9" class="hub" />
  </g>
{:else}
  <g class="chalked">
    <g class="ghost" filter="url(#chalk2)" transform="translate(-0.55, 0.45)">
      {@render kattam()}
    </g>
    <g class="chalk" filter="url(#chalk)">
      {@render kattam()}
      {#each CORNERS as [c, d] (c)}
        <path class="curl" d={curl(c, d)} />
      {/each}
      {#each EDGES as [e, [dx, dy]] (e)}
        <circle class="pulli" cx={p(e).x + dx * 6.2} cy={p(e).y + dy * 6.2} r="0.75" />
      {/each}
    </g>
  </g>
{/if}

<style>
  .carved circle,
  .carved path,
  .chalked circle,
  .chalked path {
    fill: none;
    stroke-linecap: round;
  }

  .worn {
    stroke: var(--line-soft);
    stroke-width: 0.8;
    opacity: 0.75;
  }
  .crack {
    stroke: var(--line-soft);
    stroke-width: 0.5;
    opacity: 0.6;
  }
  .nick {
    stroke: var(--line-soft);
    stroke-width: 0.55;
    opacity: 0.7;
  }
  .hi circle,
  .hi path {
    stroke: var(--line-hi);
    stroke-width: 1.55;
  }
  .groove circle,
  .groove path {
    stroke: var(--line);
    stroke-width: 1.5;
  }
  .hub {
    stroke: var(--line-soft);
    stroke-width: 0.6;
    opacity: 0.9;
  }

  .ghost {
    opacity: 0.28;
  }
  .ghost circle,
  .ghost path,
  .chalk circle,
  .chalk path {
    stroke: var(--line);
    stroke-width: 1.7;
  }
  .ghost circle,
  .ghost path {
    stroke-width: 1;
  }
  .chalk .curl {
    stroke-width: 1.15;
    opacity: 0.75;
  }
  .chalk .pulli {
    fill: var(--line);
    stroke: none;
    opacity: 0.8;
  }
</style>
