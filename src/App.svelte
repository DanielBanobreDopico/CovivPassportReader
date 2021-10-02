<script>
	export let name;

  import { onMount } from "svelte";

	import jsQR from "jsqr";
  
	var status = 'Mounting...';

  var video, canvasElement, canvas;

  function tick() {
    status = "⌛ Loading video..."
    if (video.readyState === video.HAVE_ENOUGH_DATA) {
      canvasElement.height = video.videoHeight;
      canvasElement.width = video.videoWidth;
      canvas.drawImage(video, 0, 0, canvasElement.width, canvasElement.height);
      var imageData = canvas.getImageData(0, 0, canvasElement.width, canvasElement.height);
      var code = jsQR(imageData.data, imageData.width, imageData.height, {
        inversionAttempts: "dontInvert",
      });
      if (code) {
        status = code.data;
        console.log('QR code!')
      }
    }
    requestAnimationFrame(tick);
  }

  onMount(
    () => {
      video = document.getElementById("camView");
      canvasElement = document.getElementById("canvas");
      canvas = canvasElement.getContext("2d");

      navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } }).then(function(stream) { // Use facingMode: environment to attemt to get the front camera on phones

          video.srcObject = stream;
          video.setAttribute("playsinline", true); // required to tell iOS safari we don't want fullscreen
          video.play();
          requestAnimationFrame(tick);
      });
    }
  )

</script>

<main>
	<h1>Hello {name}!</h1>
	<video id="camView"></video>
	<canvas id="canvas" hidden></canvas>
	<p>{status}</p>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>