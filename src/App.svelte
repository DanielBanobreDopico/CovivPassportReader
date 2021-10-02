<script>
	export let name;

  import { onMount } from "svelte";

	import jsQR from "jsqr";

  import base45 from 'base45-js';
  
	var status = 'Mounting...';

  var video, canvasElement, canvas;

  function divmod(a,b) {
    var remainder = a
    var quotient = 0
    if (a >= b) {
      remainder = a % b
    	quotient = (a - remainder) / b
    }
    return [ quotient, remainder ]
  }

  /* var fromCharCode = function fromCharCode(c) {
    return BASE45_CHARSET.charAt(c);
  }; */

  function base45decode(str) {
    const BASE45_CHARSET = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ $%*+-./:"
    var output = []
    var buf = []

    for(var i = 0, length=str.length; i < length; i++) {
       var j = BASE45_CHARSET.indexOf(str[i])
       if (j < 0)
              throw new Error('Base45 decode: unknown character');
       buf.push(j)
    }

    for(var i = 0, length=buf.length; i < length; i+=3) {
       var x = buf[i] + buf[i + 1] * 45
       if (length - i >= 3) {
          var [d, c] = divmod(x + buf[i + 2] * 45 * 45,256)
          output.push(d)
          output.push(c)
       } else {
         output.push(x)
       }
    }
    //return Buffer.from(output);
    var buffer = new ArrayBuffer(output.length);
    var bufferView = new Uint8Array(buffer);
    output.forEach(
      (num,idx) => bufferView[idx] = num
    )
    return buffer
  };

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
        status = base45decode(code.data).toString();
        status
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