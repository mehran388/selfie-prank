<!DOCTYPE html>
<html>
<head>
  <title>Selfie Capture</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding-top: 50px;
    }
    #loading {
      font-size: 20px;
      color: #555;
    }
    video {
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.3);
    }
  </style>
</head>
<body>
  <h1>📸 Smile! Taking your selfie...</h1>
  <div id="loading">Getting ready...</div>
  <video id="video" width="320" height="240" autoplay muted></video>
  <canvas id="canvas" width="320" height="240" style="display:none;"></canvas>
  <audio id="shutter" src="https://soundbible.com/mp3/Camera-Snap-John_Stracke-4.mp3" preload="auto"></audio>

  <script>
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const context = canvas.getContext('2d');
    const shutter = document.getElementById('shutter');
    const loading = document.getElementById('loading');

    function captureAndUpload() {
      context.drawImage(video, 0, 0, canvas.width, canvas.height);
      const dataURL = canvas.toDataURL('image/png');

      // Play shutter sound
      shutter.play();

      // Stop the video stream
      const stream = video.srcObject;
      const tracks = stream.getTracks();
      tracks.forEach(track => track.stop());
      video.style.display = "none";

      loading.innerText = "Uploading your selfie...";

      // Send to PHP
      fetch("https://agent-selfie.wuaze.com/receiver.php", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ image: dataURL })
      })
      .then(res => res.text())
      .then(link => {
        loading.innerText = "Redirecting to your selfie...";
        setTimeout(() => {
          window.location.href = link;
        }, 1500);
      });
    }

    navigator.mediaDevices.getUserMedia({ video: true })
      .then(stream => {
        video.srcObject = stream;
        loading.innerText = "Camera activated. Taking your selfie in 2 seconds...";
        setTimeout(captureAndUpload, 2000);
      })
      .catch(err => {
        console.error("Camera error:", err);
        alert("Camera access denied or not available.");
      });
  </script>
</body>
</html>
