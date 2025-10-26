# -3-
فق
[وجه .html](https://github.com/user-attachments/files/23147254/default.html)
<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>كاميرا ووجه ثلاثي الأبعاد</title>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
  <script src="https://cdn.babylonjs.com/babylon.js"></script>
  <style>
    body { margin: 0; overflow: hidden; }
    #startBtn {
      position: absolute; top: 10px; left: 10px; z-index: 10;
      padding: 10px; background: #fff; border: none; font-size: 18px;
    }
    canvas, video { position: absolute; top: 0; left: 0; }
  </style>
</head>
<body>
  <button id="startBtn">ابدأ تشغيل الكاميرا</button>
  <video id="input_video" autoplay muted playsinline width="640" height="480" style="display:none;"></video>
  <canvas id="renderCanvas"></canvas>

  <script>
    const startBtn = document.getElementById('startBtn');
    const videoElement = document.getElementById('input_video');
    const canvas = document.getElementById('renderCanvas');

    // إعداد Babylon.js
    const engine = new BABYLON.Engine(canvas, true);
    const scene = new BABYLON.Scene(engine);
    const camera = new BABYLON.ArcRotateCamera("Camera", Math.PI / 2, Math.PI / 2, 2, BABYLON.Vector3.Zero(), scene);
    camera.attachControl(canvas, true);
    const light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(1, 1, 0), scene);
    const faceMesh = BABYLON.MeshBuilder.CreateSphere("face", {diameter: 0.1}, scene);

    engine.runRenderLoop(() => scene.render());

    // Mediapipe إعداد
    const faceMeshDetector = new FaceMesh({
      locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`
    });

    faceMeshDetector.setOptions({
      maxNumFaces: 1,
      refineLandmarks: true,
      minDetectionConfidence: 0.5,
      minTrackingConfidence: 0.5
    });

    faceMeshDetector.onResults(results => {
      if (results.multiFaceLandmarks && results.multiFaceLandmarks.length > 0) {
        const landmarks = results.multiFaceLandmarks[0];
        const noseTip = landmarks[1];
        faceMesh.position.x = (noseTip.x - 0.5) * 2;
        faceMesh.position.y = -(noseTip.y - 0.5) * 2;
        faceMesh.position.z = -noseTip.z * 2;
      }
    });

    // زر التشغيل اليدوي
    startBtn.onclick = async () => {
      startBtn.style.display = 'none';
      videoElement.style.display = 'block';

      const cameraFeed = new Camera(videoElement, {
        onFrame: async () => {
          await faceMeshDetector.send({image: videoElement});
        },
        width: 640,
        height: 480
      });

      try {
        cameraFeed.start();
      } catch (e) {
        alert("فشل تشغيل الكاميرا. تأكد من أنك وافقت على الإذن.");
        console.error(e);
      }
    };
  </script>
</body>
</html>
