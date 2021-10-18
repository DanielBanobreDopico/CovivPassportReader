<script>
	export let name;

  import { onMount } from "svelte";

	import jsQR from "jsqr";

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

  function toArrayBuffer(array){
    const buffer = new ArrayBuffer(array.length);
    const bufferView = new Uint8Array(buffer);
    array.forEach(
      (num,idx) => bufferView[idx] = num
    )
    return buffer
  }

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
    /*var buffer = new ArrayBuffer(output.length);
    var bufferView = new Uint8Array(buffer);
    output.forEach(
      (num,idx) => bufferView[idx] = num
    )*/
    const buffer = toArrayBuffer(output)
    return buffer
  };


  function decode(QRdata) {
    const b45Decoded = new Uint8Array(base45decode(QRdata.slice(4)));
    const inflated = pako.inflate(b45Decoded);
    const nestedCbor = CBOR.decode(inflated.buffer);
    const payload = CBOR.decode(toArrayBuffer(nestedCbor[2]))
    status = JSON.stringify(payload)
  }

  function tick() {
    status = status == "Mounting..." ? "⌛ Loading video..." : status
    if (video.readyState === video.HAVE_ENOUGH_DATA) {
      canvasElement.height = video.videoHeight;
      canvasElement.width = video.videoWidth;
      canvas.drawImage(video, 0, 0, canvasElement.width, canvasElement.height);
      var imageData = canvas.getImageData(0, 0, canvasElement.width, canvasElement.height);
      var code = jsQR(imageData.data, imageData.width, imageData.height, {
        inversionAttempts: "dontInvert",                                                                                                                                  
      });
      if (code) {
        decode(code.data)
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


/*   function tests() {
    const buffer = base45decode(".QF14E5DC834-M6")
    const int8array = new Int8Array(buffer)
    const text = new TextDecoder().decode(int8array)  
    console.log('A',buffer,int8array,text)

    var initial = { Hello: "World" };
    var encoded = CBOR.encode(initial);
    var decoded = CBOR.decode(encoded);
    console.log(decoded)
  }
  tests() */

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