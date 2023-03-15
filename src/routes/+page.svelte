<script lang="ts">
    import { onMount } from 'svelte';

    // const recordedChunks: Blob[] = [];
    // let audioURLs: string[] = [];

    let canvas: HTMLCanvasElement;
    let canvasCtx: CanvasRenderingContext2D;

    let audioCtx: AudioContext;
    let analyser: AnalyserNode;
    let bufferLength: number;
    let dataArray: Uint8Array;
    
    let recording = false;

    let drawPosition = 0;
    const drawWidth = 1;
    
    onMount(() => {
        setup();
    });

    function setup() {
        canvasCtx = canvas.getContext('2d') as CanvasRenderingContext2D;
        canvasCtx.fillRect(0, 0, canvas.width, canvas.height);
        if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
            navigator.mediaDevices.getUserMedia({ audio: true }).then(
                (stream) => {
                    audioCtx = new AudioContext();
                    analyser = audioCtx.createAnalyser();
                    analyser.fftSize = 1024;
                    bufferLength = analyser.frequencyBinCount;
                    dataArray = new Uint8Array(bufferLength);
                    const source = audioCtx.createMediaStreamSource(stream);
                    source.connect(analyser);
                }
            ).catch(
                (err) => {
                    console.log("The following error occurred: " + err);
                }
            )
        }
        else {
            console.log("getUserMedia not supported");
        }
    }

    function startRecording() {
        // mediaRecorder.start();
        recording = true;
        draw();
    }
    function stopRecording() {
        // mediaRecorder.stop();
        recording = false;
    }

    function draw() {
        if (recording) {
            requestAnimationFrame(draw);
        }
        analyser.getByteFrequencyData(dataArray);

        // canvasCtx.fillStyle = 'rgb(0, 0, 0)';
        // canvasCtx.fillRect(0, 0, canvas.width, canvas.height);

        const barHeight = canvas.height / bufferLength;
        let y = 0;
        for (let amp of dataArray) {
            canvasCtx.fillStyle = `rgb(${amp**5 / 1e5},${amp ** 3 / 1e4},${amp})`;
            canvasCtx.fillRect(drawPosition, y, drawWidth, barHeight);
            // canvasCtx.fillRect(0, 0, canvas.width, canvas.height);

            y += barHeight;
        }
        if (drawPosition >= canvas.width) {
            drawPosition = 0;
        }
        else {
            drawPosition += drawWidth;
        }
    }

</script>

<div>{(recording) ? "Recording..." : "Stopped"}</div>

<button on:click={startRecording}>Start</button>
<button on:click={stopRecording}>Stop</button>

<div>
    <canvas bind:this={canvas} width=1600 height=900 />
</div>