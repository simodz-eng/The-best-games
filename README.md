<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>Happy Game: BMW Turbo Race</title>
  <style>
    body { margin: 0; overflow: hidden; font-family: sans-serif; background: #000; }
    #instructions {
      position: absolute;
      top: 0; left: 0; right: 0; bottom: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      color: white;
      font-size: 24px;
      background: rgba(0,0,0,0.7);
      cursor: pointer;
      z-index: 10;
    }
    #hud {
      position: absolute;
      top: 10px;
      left: 10px;
      color: white;
      font-size: 18px;
    }
  </style>
</head>
<body>
  <div id="instructions">اضغط للبدء 🚗🔥</div>
  <div id="hud">السرعة: 0 km/h</div>
  <audio id="turboSound" src="https://freesound.org/data/previews/320/320181_4939433-lq.mp3"></audio>

  <script src="https://cdn.jsdelivr.net/npm/three@0.160.1/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.1/examples/js/controls/PointerLockControls.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.1/examples/js/loaders/GLTFLoader.min.js"></script>

  <script>
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x101010);

    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const controls = new THREE.PointerLockControls(camera, document.body);

    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10, 20, 10);
    scene.add(light);

    // أرضية (طريق)
    const roadGeo = new THREE.PlaneGeometry(10, 1000);
    const roadMat = new THREE.MeshStandardMaterial({ color: 0x333333 });
    const road = new THREE.Mesh(roadGeo, roadMat);
    road.rotation.x = -Math.PI / 2;
    scene.add(road);

    // لهب
    const flameGeo = new THREE.ConeGeometry(0.1, 0.5, 8);
    const flameMat = new THREE.MeshBasicMaterial({ color: 0xff3300 });
    const flame = new THREE.Mesh(flameGeo, flameMat);
    flame.rotation.x = Math.PI;
    flame.visible = false;
    scene.add(flame);

    const turboSound = document.getElementById("turboSound");
    const hud = document.getElementById("hud");
    let speed = 0;

    // تحميل BMW
    const loader = new THREE.GLTFLoader();
    loader.load(
      'bmw_e30/scene.gltf', // ضع مسار مجسمك بعد التحميل
      (gltf) => {
        const car = gltf.scene;
        car.scale.set(0.5, 0.5, 0.5);
        car.position.set(0, -1, 0);
        scene.add(car);
        camera.position.set(0, 1, 0.5);
        car.add(camera);
      },
      undefined,
      (err) => console.error("خطأ تحميل BMW:", err)
    );

    function updateFlamePos() {
      flame.position.set(camera.position.x, camera.position.y - 0.3, camera.position.z - 1);
    }

    document.addEventListener("keydown", (e) => {
      if (e.code === "Space") {
        flame.visible = true;
        turboSound.currentTime = 0;
        turboSound.play();
        speed += 50;
        setTimeout(() => {
          flame.visible = false;
        }, 500);
      }
    });

    // HUD
    function updateHUD() {
      hud.textContent = `السرعة: ${speed} km/h`;
    }

    // منظور
    const instructions = document.getElementById("instructions");
    instructions.addEventListener("click", () => {
      controls.lock();
    });

    controls.addEventListener("lock", () => {
      instructions.style.display = "none";
    });
    controls.addEventListener("unlock", () => {
      instructions.style.display = "flex";
    });

    window.addEventListener("resize", () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // حركة بسيطة
    function animate() {
      requestAnimationFrame(animate);
      updateFlamePos();
      if (speed > 0) {
        controls.getObject().position.z -= speed * 0.001;
        speed *= 0.99; // تقليل السرعة تدريجيا
      }
      updateHUD();
      renderer.render(scene, camera);
    }
    animate();
  </script>
</body>
</html>
