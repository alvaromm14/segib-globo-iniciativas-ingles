<script>
    import Glow from "$components/Glow.svelte";
    import { isoAlpha2ToNumeric } from "$data/isoAlpha2ToNumeric.js";

    import * as topojson from "topojson-client";
    import { geoOrthographic, geoPath } from "d3-geo";
    import { scaleLinear } from "d3-scale";
    import { max } from "d3-array";
    import { timer } from "d3-timer";
    import { spring } from "svelte/motion";
    import { onMount } from "svelte";
    import { drag } from "d3-drag";
    import { select } from "d3-selection";
    import { geoCentroid } from "d3-geo";

    export let world;
    export let data;
    export let selectedCountryName = "";

    const numericToData = {};
    data.forEach((d) => {
        const numId = isoAlpha2ToNumeric[d.pais_iso];
        if (numId) numericToData[numId] = d;
    });

    let countries = topojson.feature(world, world.objects.countries).features;
    let borders = topojson.mesh(
        world,
        world.objects.countries,
        (a, b) => a !== b,
    );

    countries.forEach((country) => {
        const asPais = numericToData[country.id];
        if (asPais) {
            country.name = asPais.pais;
            country.pais_iso = asPais.pais_iso;
            return;
        }
        const asSocio = data.find(
            (d) => isoAlpha2ToNumeric[d.socio_iso] === country.id,
        );
        if (asSocio) {
            country.name = asSocio.socio;
            country.pais_iso = asSocio.socio_iso;
        }
    });

    $: filteredData = selectedCountryName
        ? data.filter((d) => d.pais === selectedCountryName)
        : data;

    $: iniciativasByNumId = (() => {
        const lookup = {};
        if (selectedCountryName) {
            filteredData.forEach((d) => {
                const numId = isoAlpha2ToNumeric[d.socio_iso];
                if (numId) lookup[numId] = d.iniciativas;
            });
        } else {
            data.forEach((d) => {
                const numId = isoAlpha2ToNumeric[d.socio_iso];
                if (numId) lookup[numId] = (lookup[numId] || 0) + d.iniciativas;
            });
        }
        return lookup;
    })();

    $: colorScale = scaleLinear()
        .domain([
            0,
            max(Object.values(iniciativasByNumId)) / 2,
            max(Object.values(iniciativasByNumId)) ?? 1,
        ])
        .range(["#ffffd9", "#ffda4d", "#ef4423"]);

    $: getCountryColor = (country) => {
        if (country.name === selectedCountryName) return "#212C55";
        const val = iniciativasByNumId[country.id];
        if (!val) return "#e8eaea";
        return colorScale(val);
    };

    let tooltipData;

    $: if (selectedCountryName) {
        const found = countries.find((c) => c.name === selectedCountryName);
        if (found) tooltipData = found;
    }

    $: if (!selectedCountryName) tooltipData = null;

    $: dispatch("tooltipChange", {
        tooltipData,
        iniciativas: iniciativasByNumId[tooltipData?.id],
        colorScale,
    });

    let width = 400;
    $: height = width;

    $: projection = geoOrthographic()
        .scale(width / 2)
        .rotate([$rotation, 0])
        .translate([width / 2, height / 2]);

    $: path = geoPath(projection);

    let rotation = spring(0, { stiffness: 0.08, damping: 0.4 });
    let degreesPerFrame = 0.5;

    const t = timer(() => {
        if (dragging || tooltipData) return;
        $rotation += degreesPerFrame;
    }, 0);

    let globe;
    const DRAG_SENSITIVITY = 0.5;
    let dragging = false;

    onMount(() => {
        const element = select(globe);
        element.call(
            drag()
                .on("drag", (event) => {
                    $rotation = $rotation + event.dx * DRAG_SENSITIVITY;
                    dragging = true;
                })
                .on("end", () => {
                    dragging = false;
                }),
        );
    });

    $: if (tooltipData) {
        const center = geoCentroid(tooltipData);
        $rotation = -center[0];
    }

    import { createEventDispatcher } from "svelte";
    const dispatch = createEventDispatcher();

    $: dispatch("tooltipChange", {
        tooltipData,
        iniciativas: iniciativasByNumId[tooltipData?.id],
        colorScale,
    });
</script>

<div class="globo-wrapper" bind:clientWidth={width}>
    <svg {width} {height} bind:this={globe} class:dragging>
        <Glow />
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <circle
            cx={width / 2}
            cy={height / 2}
            r={width / 2}
            fill={"#d2d5d5"}
            filter="url(#glow)"
            on:click={() => {
                tooltipData = null;
            }}
        />
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        {#each countries as country}
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <path
                d={path(country)}
                fill={getCountryColor(country)}
                stroke="none"
                style:cursor={iniciativasByNumId[country.id]
                    ? "pointer"
                    : "default"}
                on:click={() => {
                    if (iniciativasByNumId[country.id]) tooltipData = country;
                }}
            />
        {/each}
        <path d={path(borders)} fill="none" stroke="#e8eaea" />
        {#if tooltipData}
            <path
                d={path(tooltipData)}
                fill="transparent"
                stroke={tooltipData.name === selectedCountryName
                    ? "#63ffa1"
                    : "#212c55"}
                stroke-width="1.5"
            />
        {/if}
    </svg>
</div>

<style>
    .globo-wrapper {
        max-width: 700px;
        margin: 0 auto;
        position: relative;
        z-index: 0;
    }

    svg {
        overflow: visible;
    }

    .dragging {
        cursor: grabbing;
    }
</style>
