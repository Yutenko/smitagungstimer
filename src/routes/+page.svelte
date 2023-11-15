<script>
    import { readable } from "svelte/store";
    import { tweened } from "svelte/motion";
    import { onMount } from "svelte";
    import { fade } from "svelte/transition";

    let timer = 5; // seconds
    let start;
    let mstime;

    let kermits = [];

    let display;
    let bg;

    function randomIntFromInterval(min, max) {
        // min and max included
        return Math.floor(Math.random() * (max - min + 1) + min);
    }

    onMount(() => {
        mstime = readable(new Date().getTime(), (set) => {
            let animationFrame;
            const next = () => {
                set(new Date().getTime());
                animationFrame = requestAnimationFrame(next);
            };
            if (window.requestAnimationFrame) {
                next();
                return () => {
                    cancelAnimationFrame(animationFrame);
                };
            }
        });

        start = new Date().getTime();
    });

    $: time = Math.floor(($mstime - start) / 1000);
    $: toWait = timer - time > 0 ? timer - time : 0;
    $: minutes = Math.floor(toWait / 60);
    $: seconds = toWait - minutes * 60;

    $: {
        if (start && minutes === 0 && seconds === 0) {
            setTimeout(() => {
                addKermit(randomIntFromInterval(1, 1));
            }, 1000);
        }
    }

    const progress = tweened(0, { duration: 1000 });
    $: {
        $progress = 1 - toWait / timer;
    }

    function addKermit(timer) {
        setTimeout(() => {
            display.innerHTML = "ENDE";
            const screenWidth = window.innerWidth;
            const screenHeight = window.innerHeight;

            // Berechnen Sie die Breite und Höhe des Objekts als 25% der Bildschirmgröße
            const rand = randomIntFromInterval(5, 35);
            const objectWidth = screenWidth * (rand / 100);
            const objectHeight = objectWidth;

            // Berechnen Sie die maximale X- und Y-Koordinaten, um sicherzustellen, dass das Objekt auf dem Bildschirm erscheint
            const maxX = screenWidth - objectWidth;
            const maxY = screenHeight - objectHeight;

            // Generieren Sie Zufalls-X- und Y-Koordinaten innerhalb der berechneten Grenzen
            const randomX = Math.random() * maxX;
            const randomY = Math.random() * maxY;

            // Erstellen Sie ein Objekt mit den Zufallskoordinaten und den Abmessungen des Objekts
            const k = {
                x: randomX,
                y: randomY,
                width: objectWidth,
                height: objectHeight,
            };
            kermits = [...kermits, k];

            addKermit(randomIntFromInterval(1, 6));
            setTimeout(shakeScreen, randomDelay());
        }, timer * 1000);
    }

    function randomDelay() {
        return Math.random() * 3000 + 1000; // Random delay between 1 and 4 seconds
    }

    function randomDuration() {
        return Math.random() * 2000 + 500; // Random duration between 0.5 and 2.5 seconds
    }

    function shakeScreen() {
        bg.classList.add("shake");
        setTimeout(() => {
            bg.classList.remove("shake");
            setTimeout(shakeScreen, randomDelay());
        }, randomDuration());
    }
</script>

{#each kermits as kermit}
    <div
        class="muppet"
        transition:fade
        style="top:{kermit.y}px;left:{kermit.x}px;width:{kermit.width}px;height:{kermit.height}px"
    />
{/each}

<section class="app" bind:this={bg}>
    <svg
        viewBox="0 0 10 10"
        style="background: hsl({120 * (1 - $progress)}deg, 50%, {100 -
            50 * $progress}%)"
        preserveAspectRatio="none"
    >
        <rect
            y={10 * $progress}
            width="10"
            height={10 * (1 - $progress)}
            fill="hsl({120 * (1 - $progress)}deg, 50%, 50%)"
        />
    </svg>
    <div class="timer-value">
        <div class="tt" bind:this={display}>{minutes}m {seconds}s</div>
    </div>
</section>

<style>
    svg {
        position: absolute;
        left: 0;
        right: 0;
        width: 100%;
        height: 100%;
    }
    :global(body) {
        color: #333;
        padding: 0;
        margin: 0;
        box-sizing: border-box;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
            Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif;
    }
    .tt {
        width: 100vw;
        text-align: center;
    }
    .app {
        overflow: hidden;
        display: flex;
        align-items: flex-end;
        height: 100vh;
    }

    .timer-value {
        position: absolute;
        display: flex;
        mix-blend-mode: difference;
        color: #333;
        flex-direction: row;
        align-items: center;
        justify-content: center;
        font-size: 35vmin;
        height: 100vh;
        width: 100vw;
    }

    .muppet {
        display: block;
        max-width: 100%;
        height: auto;
        border-radius: 50%;
        height: 25vmax;
        width: 25vmax;
        margin-left: auto;
        margin-right: auto;

        background-image: url(/muppets-muppet-show.gif);
        background-position: center center;
        background-repeat: no-repeat;
        background-size: cover;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 1;
    }

    .shake {
        animation: shake 0.5s ease-in-out infinite;
        animation-play-state: paused;
    }

    @keyframes shake {
        0% {
            transform: translate(0, 0);
        }
        25% {
            transform: translate(-5px, -5px);
        }
        50% {
            transform: translate(5px, 5px);
        }
        75% {
            transform: translate(-5px, -5px);
        }
        100% {
            transform: translate(5px, 5px);
        }
    }
</style>
