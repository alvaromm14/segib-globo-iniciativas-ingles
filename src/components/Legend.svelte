<script>
    export let colorScale;
    export let iniciativas;
    export let data;

    import { format } from "d3-format";
    import { fade } from "svelte/transition";

    const formatNum = (d) => format(",")(d).replace(/,/g, ",");

    $: [minColor, midColor, maxColor] = colorScale.range();
    $: [minValue, , maxValue] = colorScale.domain();

    $: percentOfMax = iniciativas ? (iniciativas / maxValue) * 100 : null;
</script>

<div class="legend">
    <div
        class="bar"
        style:background="linear-gradient(to right, {minColor} 0%, {midColor} 50%,
        {maxColor} 100%)"
    >
        {#if percentOfMax}
            <div transition:fade class="marker" style:left={percentOfMax + "%"}>
                <span class="line" />
                <div class="marker-tooltip">
                    <span class="marker-name">{data.name}</span>
                    <span class="marker-count">{formatNum(iniciativas)}</span>
                </div>
            </div>
        {/if}
    </div>
</div>

<style>
    .legend {
        display: flex;
        flex-direction: row;
        align-items: center;
        gap: 8px;
        width: 100%;
        max-width: 300px;
        margin: 0 auto;
        margin-bottom: 20px;
        z-index: 1;
        position: relative;
    }

    .bar {
        height: 12px;
        flex: 1;
        position: relative;
        border-radius: 4px;
        border: 1px solid #d0d0d0;
        overflow: visible;
    }

    .marker {
        position: absolute;
        top: 50%;
        transform: translate(-50%, -50%);
        display: flex;
        flex-direction: column;
        align-items: center;
        transition: left 500ms cubic-bezier(1, 0, 0, 1);
    }

    .marker-tooltip {
        position: absolute;
        top: calc(100% + 5px);
        display: flex;
        flex-direction: column;
        align-items: center;
        color: #3d3935;
        font-size: 0.9rem;
        white-space: nowrap;
        pointer-events: none;
        gap: 3px;
    }

    .marker-name {
        font-weight: 700;
        text-shadow:
            -2px 0 0 white,
            2px 0 0 white,
            0 -2px 0 white,
            0 2px 0 white,
            -2px -2px 0 white,
            2px -2px 0 white,
            -2px 2px 0 white,
            2px 2px 0 white;
    }

    .marker-count {
        font-weight: 400;
        text-shadow:
            -2px 0 0 white,
            2px 0 0 white,
            0 -2px 0 white,
            0 2px 0 white,
            -2px -2px 0 white,
            2px -2px 0 white,
            -2px 2px 0 white,
            2px 2px 0 white;
    }

    .line {
        width: 6px;
        height: 17px;
        background: white;
        border: 1.5px solid black;
        border-radius: 4px;
    }
</style>
