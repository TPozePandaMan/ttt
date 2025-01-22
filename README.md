<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Camera App with Video, Photo, and Gallery</title>
  <link href="https://fonts.googleapis.com/css2?family=Material+Icons&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 20px;
      background-color: #f3f4f6;
      color: #333;
    }
    h1 {
      margin-bottom: 20px;
    }
    video, canvas {
      width: 100%;
      max-width: 640px;
      margin-bottom: 20px;
      border-radius: 10px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }
    .button-container {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 10px;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      margin: 5px;
      cursor: pointer;
      background-color: #007BFF;
      color: white;
      border: none;
      border-radius: 5px;
      transition: background-color 0.3s;
      display: flex;
      align-items: center;
      gap: 5px;
    }
    button:hover {
      background-color: #0056b3;
    }
    button:disabled {
      background-color: #ccc;
      cursor: not-allowed;
    }
    a {
      display: inline-block;
      margin: 10px 5px;
      text-decoration: none;
      color: white;
      background-color: #28a745;
      padding: 10px 20px;
      border-radius: 5px;
      transition: background-color 0.3s;
    }
    a:hover {
      background-color: #1e7e34;
    }
    a:disabled {
      background-color: #ccc;
      pointer-events: none;
    }
    .material-icons {
      font-size: 18px;
    }
    .gallery {
      margin-top: 20px;
    }
    .gallery img {
      width: 100px;
      height: 100px;
      object-fit: cover;
      margin: 5px;
      border-radius: 5px;
      cursor: pointer;
      transition: transform 0.3s;
    }
    .gallery img:hover {
      transform: scale(1.1);
    }
    .fullscreen {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.8);
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 1000;
    }
    .fullscreen img {
      max-width: 90%;
      max-height: 90%;
      border-radius: 10px;
    }
  </style>
</head>
<body>
  <h1>Camera App with Video, Photo, and Gallery</h1>
  <video id="video" autoplay playsinline></video>
  <canvas id="canvas" style="display: none;"></canvas>
  <div class="button-container">
    <button id="start">
      <span class="material-icons">videocam</span>
      Start Recording
    </button>
    <button id="stop" disabled>
      <span class="material-icons">stop</span>
      Stop Recording
    </button>
    <button id="take-photo">
      <span class="material-icons">camera_alt</span>
      Take Photo
    </button>
  </div>
  <div>
    <a id="download" style="display: none;">Download Video</a>
    <a id="photo-link" style="display: none;">Download Photo</a>
  </div>

  <div class="gallery" id="gallery">
    <h2>Your Gallery</h2>
  </div>

  <div class="fullscreen" id="fullscreen" style="display: none;">
    <img id="fullscreen-img" src="" alt="Fullscreen View">
  </div>

  <script>
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const startButton = document.getElementById('start');
    const stopButton = document.getElementById('stop');
    const takePhotoButton = document.getElementById('take-photo');
    const downloadLink = document.getElementById('download');
    const photoLink = document.getElementById('photo-link');
    const gallery = document.getElementById('gallery');
    const fullscreen = document.getElementById('fullscreen');
    const fullscreenImg = document.getElementById('fullscreen-img');

    let mediaRecorder;
    let recordedChunks = [];

    async function setupCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        video.srcObject = stream;
        return stream;
      } catch (error) {
        console.error('Error accessing the camera:', error);
      }
    }

    startButton.addEventListener('click', async () => {
      const stream = await setupCamera();
      recordedChunks = [];

      mediaRecorder = new MediaRecorder(stream);

      mediaRecorder.ondataavailable = (event) => {
        if (event.data.size > 0) {
          recordedChunks.push(event.data);
        }
      };

      mediaRecorder.onstop = () => {
        const blob = new Blob(recordedChunks, { type: 'video/webm' });
        const url = URL.createObjectURL(blob);
        downloadLink.href = url;
        downloadLink.download = 'recorded-video.webm';
        downloadLink.style.display = 'inline';
        downloadLink.textContent = 'Download Video';
      };

      mediaRecorder.start();
      startButton.disabled = true;
      stopButton.disabled = false;
    });

    stopButton.addEventListener('click', () => {
      mediaRecorder.stop();
      startButton.disabled = false;
      stopButton.disabled = true;
    });

    takePhotoButton.addEventListener('click', () => {
      const context = canvas.getContext('2d');
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      const photoUrl = canvas.toDataURL('image/png');
      photoLink.href = photoUrl;
      photoLink.download = 'photo.png';
      photoLink.style.display = 'inline';
      photoLink.textContent = 'Download Photo';

      const img = document.createElement('img');
      img.src = photoUrl;
      img.alt = 'Photo';
      img.addEventListener('click', () => {
        fullscreen.style.display = 'flex';
        fullscreenImg.src = photoUrl;
      });
      gallery.appendChild(img);
    });

    fullscreen.addEventListener('click', () => {
      fullscreen.style.display = 'none';
      fullscreenImg.src = '';
    });

    setupCamera();
  </script>
</body>
</html>
