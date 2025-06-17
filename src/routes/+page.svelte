<script>
    import testImage from "$lib/test_img.png";
    import { onMount } from "svelte";
    import isolateHighlights from "$lib/isolate-highlights";

    let imageWidth = 1920;
    let imageHeight = 1080;
    let imageUrl = testImage;
    let threshold = 200;
    let startPoint = { x: 960, y: 540 };
    let endPoint = { x: 960, y: 540 };
    let endScale = 2;
    let resolution = 100;

    
    let /** @type {HTMLCanvasElement} */ baseCanvas,
        /** @type {HTMLCanvasElement} */ thresholdCanvas,
        /** @type {HTMLCanvasElement} */ raysCanvas,
        /** @type {HTMLCanvasElement} */ outputCanvas;

    /**
     * Load image from url
     * @param {string} src image url
     * @returns {Promise<HTMLImageElement>}
     */
    async function loadImage(src) {
        return new Promise((resolve, reject) => {
            const img = new Image();
            img.addEventListener("load", () => {
                resolve(img);
            });
            img.addEventListener("error", (e) => {
                reject(e);
            });
            img.src = src;
        });
    }

    async function render() {
        let img = await loadImage(imageUrl);
        imageWidth = img.width;
        imageHeight = img.height;

        baseCanvas.width = imageWidth;
        baseCanvas.height = imageHeight;
        thresholdCanvas.width = imageWidth;
        thresholdCanvas.height = imageHeight;
        raysCanvas.width = imageWidth;
        raysCanvas.height = imageHeight;
        outputCanvas.width = imageWidth;
        outputCanvas.height = imageHeight;

        let baseCtx = baseCanvas.getContext("2d");
        baseCtx.drawImage(img, 0, 0, imageWidth, imageHeight);

        let thresholdCtx = thresholdCanvas.getContext("2d");
        thresholdCtx.drawImage(baseCanvas, 0, 0, imageWidth, imageHeight);
        isolateHighlights(thresholdCtx, threshold);

        let raysCtx = raysCanvas.getContext("2d");
        raysCtx.globalCompositeOperation = "lighten";
        raysCtx.save();
        for (let i = 0; i < resolution; i++) {
            let scale = 1 + (endScale - 1) * (i / resolution);
            let brightness = 1 - (i / resolution);
            let x = startPoint.x + (endPoint.x - startPoint.x) * i / resolution;
            let y = startPoint.y + (endPoint.y - startPoint.y) * i / resolution;
            let width = imageWidth * scale;
            let height = imageHeight * scale;
            raysCtx.translate(x, y);
            raysCtx.filter = `brightness(${brightness})`;
            raysCtx.drawImage(thresholdCanvas, -width / 2, -height / 2, width, height);
            raysCtx.restore();
            raysCtx.save();
        }

        let outputCtx = outputCanvas.getContext("2d");
        outputCtx.drawImage(baseCanvas, 0, 0, imageWidth, imageHeight);
        outputCtx.save();
        outputCtx.globalCompositeOperation = "screen";
        outputCtx.drawImage(raysCanvas, 0, 0, imageWidth, imageHeight);
        outputCtx.restore();
    }

    onMount(() => {
        render();
    });
</script>

<canvas bind:this={baseCanvas} width={imageWidth} height={imageHeight} />
<canvas bind:this={thresholdCanvas} width={imageWidth} height={imageHeight} />
<canvas bind:this={raysCanvas} width={imageWidth} height={imageHeight} />
<canvas bind:this={outputCanvas} width={imageWidth} height={imageHeight} />

<style>
    canvas {
        width: 300px;
    }
</style>