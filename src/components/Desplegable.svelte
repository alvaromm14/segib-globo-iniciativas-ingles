<script>
    import { fade } from "svelte/transition";
    import { onMount } from "svelte";

    export let options = [];
    export let value = "";
    export let placeholder = "Seleccionar";
    export let formatCount = null; // Se mantiene por compatibilidad de props, aunque ya no se usa en el HTML

    let dropdownOpen = false;

    onMount(() => {
        const handleClickOutside = (event) => {
            if (!event.target.closest(".select-wrapper")) {
                dropdownOpen = false;
            }
        };
        document.addEventListener("click", handleClickOutside);
        return () => document.removeEventListener("click", handleClickOutside);
    });
</script>

<div class="select-wrapper" class:open={dropdownOpen}>
    <button
        class="select-trigger"
        on:click={() => (dropdownOpen = !dropdownOpen)}
    >
        <span class={value ? "selected" : "placeholder"}>
            {value || placeholder}
        </span>
        <div class="trigger-right">
            <span class="separator" />
            <svg class="chevron" viewBox="0 0 10 6" width="10" height="6">
                <path
                    d="M1 1l4 4 4-4"
                    stroke="currentColor"
                    stroke-width="1.5"
                    fill="none"
                    stroke-linecap="round"
                />
            </svg>
        </div>
    </button>

    {#if dropdownOpen}
        <ul class="dropdown" transition:fade={{ duration: 150 }}>
            {#each options as option}
                <li
                    on:click={() => {
                        value = option.label;
                        dropdownOpen = false;
                    }}
                >
                    {option.label}
                </li>
            {/each}
        </ul>
    {/if}
</div>

<nav class="side-menu">
    <ul>
        {#each options as option}
            <li
                class:active={value === option.label}
                on:click={() => (value = option.label)}
            >
                {option.label}
            </li>
        {/each}
    </ul>
</nav>

<style>
    /* ── DROPDOWN (Móvil por defecto) ── */
    .select-wrapper {
        display: block;
        position: relative;
        margin: 10px auto 15px auto;
        width: 100%;
        max-width: 300px;
    }

    .select-trigger {
        width: 100%;
        display: flex;
        transition: 100ms;
        justify-content: space-between;
        align-items: center;
        padding: 6px 14px;
        border: 1px solid #d0d0d0;
        border-radius: 4px;
        background-color: rgb(255, 255, 255);
        color: #3d3935;
        text-align: left;
        cursor: pointer;
    }

    .select-trigger:hover {
        border-color: #aaa;
    }
    .select-trigger:focus {
        outline: none;
    }
    .select-wrapper.open .select-trigger {
        outline: 2px solid #2684ff;
        outline-offset: -1px;
    }

    .trigger-right {
        display: flex;
        align-items: center;
        gap: 10px;
        flex-shrink: 0;
    }

    .separator {
        width: 1px;
        height: 18px;
        background: #d0d0d0;
    }

    .chevron {
        transition: transform 200ms ease;
        color: #888;
    }

    .select-wrapper.open .chevron {
        transform: rotate(180deg);
    }
    .placeholder {
        color: #aaa;
    }
    .selected {
        color: #3d3935;
    }

    .dropdown {
        position: absolute;
        top: 100%;
        left: 0;
        right: 0;
        margin: 6px 0 0 0;
        background: white;
        border: 1px solid #dadada;
        border-radius: 6px;
        max-height: 220px;
        overflow-y: auto;
        list-style: none;
        padding: 0;
        z-index: 100;
    }

    .dropdown li {
        padding: 9px 14px;
        color: #3d3935;
        cursor: pointer;
        user-select: none;
    }

    .dropdown li:hover {
        background: #deebff;
    }

    /* ── MENÚ LATERAL (Desktop) ── */
    .side-menu {
        display: none;
        flex-shrink: 0;
        margin-left: 10px;
    }

    .side-menu ul {
        list-style: none;
        margin: 0;
        padding: 0;
    }

    .side-menu li {
        padding: 8px 10px;
        border-radius: 4px;
        color: #3d3935;
        cursor: pointer;
        user-select: none;
        font-size: 0.9rem;
        border-left: 4px solid transparent;
        transition:
            background 100ms,
            border-color 100ms;
    }

    .side-menu li:hover {
        background: #deebff;
    }
    .side-menu li.active {
        background: #212c55;
        border-left-color: #63ffa1;
        color: white;
        font-weight: 600;
    }

    /* ── MEDIA QUERIES ── */
    @media (min-width: 900px) {
        .select-wrapper {
            display: none;
        }

        .side-menu {
            display: block;
        }
    }
</style>
