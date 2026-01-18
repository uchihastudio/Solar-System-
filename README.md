<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate 3D Solar System | Uchiha Studio</title>
    <style>
        body { margin: 0; background-color: #000; overflow: hidden; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        #ui {
            position: absolute; top: 25px; left: 25px; color: #00f2ff;
            text-transform: uppercase; letter-spacing: 4px; pointer-events: none;
            background: rgba(0, 10, 20, 0.7); padding: 15px; border-left: 4px solid #00f2ff;
            text-shadow: 0 0 15px #00f2ff; z-index: 100;
        }
        .planet-label {
            color: #fff; font-size: 12px; font-weight: bold; background: rgba(0, 242, 255, 0.2);
            padding: 4px 12px; border-radius: 20px; border: 1px solid rgba(0, 242, 255, 0.5);
            backdrop-filter: blur(8px); pointer-events: none; white-space: nowrap;
            text-shadow: 0 0 5px #000;
        }
    </style>
</head>
<body>
    <div id="ui">
        Cosmic Engine v6.0
        <div style="font-size: 10px; color: #ffffffbb; margin-top: 5px;">
            Full Architecture | Light Structures: Enabled | Reaction Graphics: Max
        </div>
    </div>

    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

        // --- 1. CORE ENGINE SETUP ---
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 10000);
        const renderer = new THREE.WebGLRenderer({ antialias: true, logarithmicDepthBuffer: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.toneMapping = THREE.ACESFilmicToneMapping;
        renderer.outputColorSpace = THREE.SRGBColorSpace;
        document.body.appendChild(renderer.domElement);

        const labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize(window.innerWidth, window.innerHeight);
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        document.body.appendChild(labelRenderer.domElement);

        const controls = new OrbitControls(camera, labelRenderer.domElement);
        camera.position.set(450, 300, 500);
        controls.enableDamping = true;

        // --- 2. LIGHT STRUCTURES & REACTIONS ---
        const sunPointLight = new THREE.PointLight(0xffffff, 80000, 3000, 1.6);
        scene.add(sunPointLight);

        const ambient = new THREE.AmbientLight(0x222233, 0.7); // Subtle space-blue fill
        scene.add(ambient);

        const loader = new THREE.TextureLoader();

        // --- 3. MILKY WAY & STARFIELD ---
        const starGeo = new THREE.BufferGeometry();
        const starCount = 20000;
        const pos = new Float32Array(starCount * 3);
        const cols = new Float32Array(starCount * 3);
        for(let i=0; i < starCount * 3; i++) {
            pos[i] = (Math.random() - 0.5) * 4000;
            cols[i] = 0.5 + Math.random() * 0.5; 
        }
        starGeo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
        starGeo.setAttribute('color', new THREE.BufferAttribute(cols, 3));
        scene.add(new THREE.Points(starGeo, new THREE.PointsMaterial({ size: 1.5, vertexColors: true, transparent: true, opacity: 0.8 })));

        // --- 4. THE SUN (CORONA & GLOW REACTION) ---
        const sunGroup = new THREE.Group();
        const sunMesh = new THREE.Mesh(
            new THREE.SphereGeometry(35, 64, 64),
            new THREE.MeshBasicMaterial({ map: loader.load('https://i.postimg.cc/9M99S9y0/sun.jpg') })
        );
        sunGroup.add(sunMesh);

        const coronaTex = loader.load('https://threejs.org/examples/textures/lensflare/lensflare0.png');
        const coronaMat = new THREE.SpriteMaterial({ 
            map: coronaTex, color: 0xffaa00, transparent: true, blending: THREE.AdditiveBlending, opacity: 0.7 
        });
        const corona = new THREE.Sprite(coronaMat);
        corona.scale.set(180, 180, 1);
        sunGroup.add(corona);
        scene.add(sunGroup);

        // --- 5. INDIVIDUAL PLANET FACTORY ---
        function createPlanet(name, size, tex, dist, speed, tilt, colorGlow, ring = null) {
            const orbitGroup = new THREE.Group();
            scene.add(orbitGroup);

            // Orbit Path Visual
            const pathGeo = new THREE.RingGeometry(dist - 0.5, dist + 0.5, 128);
            const pathMat = new THREE.MeshBasicMaterial({ color: 0xffffff, opacity: 0.1, transparent: true, side: THREE.DoubleSide });
            const path = new THREE.Mesh(pathGeo, pathMat);
            path.rotation.x = Math.PI/2;
            scene.add(path);

            // Planet Body
            const mesh = new THREE.Mesh(
                new THREE.SphereGeometry(size, 32, 32),
                new THREE.MeshStandardMaterial({ 
                    map: loader.load(tex), 
                    roughness: 0.8, 
                    metalness: 0.2,
                    emissive: new THREE.Color(colorGlow),
                    emissiveIntensity: 0.1
                })
            );
            mesh.position.x = dist;
            mesh.rotation.z = tilt;
            orbitGroup.add(mesh);

            // Individual Atmosphere Reaction (Glow)
            const glowMat = new THREE.SpriteMaterial({ 
                map: coronaTex, color: colorGlow, transparent: true, blending: THREE.AdditiveBlending, opacity: 0.3 
            });
            const glow = new THREE.Sprite(glowMat);
            glow.scale.set(size * 4, size * 4, 1);
            mesh.add(glow);

            // Label
            const div = document.createElement('div');
            div.className = 'planet-label';
            div.textContent = name;
            const label = new CSS2DObject(div);
            label.position.set(0, size + 8, 0);
            mesh.add(label);

            if(ring) {
                const rMesh = new THREE.Mesh(
                    new THREE.RingGeometry(ring.inner, ring.outer, 64),
                    new THREE.MeshStandardMaterial({ map: loader.load(ring.tex), side: THREE.DoubleSide, transparent: true })
                );
                rMesh.position.x = dist;
                rMesh.rotation.x = Math.PI/2.2;
                orbitGroup.add(rMesh);
            }

            return { orbitGroup, mesh, speed };
        }

        // --- 6. INITIALIZE FULL ARRAY (8 Planets + Pluto) ---
        const planets = [
            createPlanet("Mercury", 3.5, 'https://i.postimg.cc/8P6S0m9H/mercury.jpg', 70, 0.02, 0.03, 0xaaaaaa),
            createPlanet("Venus", 7, 'https://i.postimg.cc/qR798K8H/venus.jpg', 105, 0.015, 3.0, 0xffaa00),
            createPlanet("Earth", 7.5, 'https://i.postimg.cc/0j7fS6V1/earth.jpg', 145, 0.01, 0.41, 0x0088ff),
            createPlanet("Mars", 5, 'https://i.postimg.cc/6qS70S8P/mars.jpg', 185, 0.008, 0.44, 0xff4400),
            createPlanet("Jupiter", 20, 'https://i.postimg.cc/9M99S9y0/jupiter.jpg', 260, 0.004, 0.05, 0xffcc88),
            createPlanet("Saturn", 17, 'https://i.postimg.cc/P56P86C9/saturn.jpg', 350, 0.002, 0.47, 0xffddaa, {
                inner: 22, outer: 40, tex: 'https://i.postimg.cc/90V7mxYv/saturn-ring.png'
            }),
            createPlanet("Uranus", 11, 'https://i.postimg.cc/0j7fS6V1/uranus.jpg', 430, 0.0015, 1.7, 0x00ffff),
            createPlanet("Neptune", 11, 'https://i.postimg.cc/qR798K8H/neptune.jpg', 500, 0.001, 0.5, 0x3333ff),
            createPlanet("Pluto", 2, 'https://i.postimg.cc/8P6S0m9H/mercury.jpg', 550, 0.0007, 1.0, 0x888888)
        ];

        // --- 7. ANIMATION ---
        function animate() {
            const time = Date.now() * 0.001;
            sunMesh.rotation.y += 0.002;
            
            // Pulsing Sun Reaction
            corona.scale.set(180 + Math.sin(time*2)*10, 180 + Math.sin(time*2)*10, 1);

            planets.forEach(p => {
                p.mesh.rotation.y += 0.01;      // Axis rotation
                p.orbitGroup.rotation.y += p.speed; // Orbital movement
            });

            controls.update();
            renderer.render(scene, camera);
            labelRenderer.render(scene, camera);
            requestAnimationFrame(animate);
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>
