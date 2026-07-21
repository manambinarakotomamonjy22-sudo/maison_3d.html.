<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plan 3D de la Maison - 215 m²</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        #info {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0,0,0,0.7);
            color: white;
            padding: 15px 25px;
            border-radius: 8px;
            pointer-events: none;
            z-index: 100;
            border-left: 5px solid #ffaa00;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        }
        #info h1 { margin: 0; font-size: 1.4rem; font-weight: 400; }
        #info h1 span { font-weight: 700; color: #ffaa00; }
        #info p { margin: 5px 0 0 0; font-size: 0.9rem; opacity: 0.8; }
        #legend {
            position: absolute;
            bottom: 30px;
            left: 30px;
            background: rgba(0,0,0,0.6);
            color: #eee;
            padding: 12px 18px;
            border-radius: 6px;
            font-size: 0.8rem;
            backdrop-filter: blur(4px);
            border: 1px solid #444;
            pointer-events: none;
            z-index: 100;
        }
        #legend span { display: inline-block; width: 12px; height: 12px; margin-right: 6px; border-radius: 2px; }
    </style>
</head>
<body>
    <div id="info">
        <h1>🏠 <span>MODÈLE 3D</span> · 215 m² (20m x 11m)</h1>
        <p>Mesures : 20m (façade) · 11m (profondeur) · Cuisine 3m x 6m</p>
    </div>
    <div id="legend">
        <div><span style="background:#4a90e2;"></span> Séjour (36m²)</div>
        <div><span style="background:#f5a623;"></span> Cuisine (18m²)</div>
        <div><span style="background:#7ed321;"></span> Chambres (20-30m²)</div>
        <div><span style="background:#d0021b;"></span> Salles de bain (9m²)</div>
        <div><span style="background:#9b9b9b;"></span> Couloir / Buanderie</div>
    </div>

    <!-- Importation de Three.js et OrbitControls -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.126.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.126.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        // --- 1. Initialisation de la scène, caméra, rendu ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x111122); // Fond nuit

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(25, 18, 25);
        camera.lookAt(10, 0, 5);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

        // --- 2. Contrôles ---
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.target.set(10, 1.5, 5);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.autoRotate = false;
        controls.update();

        // --- 3. Lumières ---
        // Lumière ambiante
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);

        // Lumière directionnelle (pour les ombres)
        const dirLight = new THREE.DirectionalLight(0xffeedd, 1);
        dirLight.position.set(15, 20, 10);
        dirLight.castShadow = true;
        dirLight.shadow.mapSize.width = 1024;
        dirLight.shadow.mapSize.height = 1024;
        const d = 25;
        dirLight.shadow.camera.left = -d;
        dirLight.shadow.camera.right = d;
        dirLight.shadow.camera.top = d;
        dirLight.shadow.camera.bottom = -d;
        dirLight.shadow.camera.near = 1;
        dirLight.shadow.camera.far = 50;
        scene.add(dirLight);
        
        const fillLight = new THREE.DirectionalLight(0x4466ff, 0.3);
        fillLight.position.set(-10, 5, -10);
        scene.add(fillLight);

        // Lumière du sol (pour éclairer les dessous)
        const backLight = new THREE.PointLight(0x4466ff, 0.2);
        backLight.position.set(10, -5, 5);
        scene.add(backLight);

        // --- 4. Fonctions utilitaires ---

        // Créer une pièce (sol coloré + murs transparents)
        function createRoom(x, z, w, d, h, color, label, labelOffsetY = 0) {
            const group = new THREE.Group();
            
            // Sol
            const floorGeo = new THREE.BoxGeometry(w, 0.1, d);
            const floorMat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.7, metalness: 0.1 });
            const floor = new THREE.Mesh(floorGeo, floorMat);
            floor.position.set(w/2, 0.05, d/2);
            floor.receiveShadow = true;
            floor.castShadow = true;
            group.add(floor);

            // Murs (bordures transparentes pour délimiter)
            const wallMat = new THREE.MeshStandardMaterial({ color: 0xcccccc, transparent: true, opacity: 0.15, side: THREE.DoubleSide });
            const wallHeight = h;
            // Mur avant (z = 0)
            const wallFront = new THREE.Mesh(new THREE.BoxGeometry(w, wallHeight, 0.05), wallMat);
            wallFront.position.set(w/2, wallHeight/2, 0);
            group.add(wallFront);
            // Mur arrière (z = d)
            const wallBack = new THREE.Mesh(new THREE.BoxGeometry(w, wallHeight, 0.05), wallMat);
            wallBack.position.set(w/2, wallHeight/2, d);
            group.add(wallBack);
            // Mur gauche (x = 0)
            const wallLeft = new THREE.Mesh(new THREE.BoxGeometry(0.05, wallHeight, d), wallMat);
            wallLeft.position.set(0, wallHeight/2, d/2);
            group.add(wallLeft);
            // Mur droit (x = w)
            const wallRight = new THREE.Mesh(new THREE.BoxGeometry(0.05, wallHeight, d), wallMat);
            wallRight.position.set(w, wallHeight/2, d/2);
            group.add(wallRight);

            group.position.set(x, 0, z);
            scene.add(group);

            // Étiquette (Sprite)
            const canvas = document.createElement('canvas');
            canvas.width = 256;
            canvas.height = 128;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = 'rgba(0,0,0,0.6)';
            ctx.shadowColor = 'black';
            ctx.shadowBlur = 10;
            ctx.roundRect(10, 10, 236, 108, 15);
            ctx.fill();
            ctx.shadowBlur = 0;
            ctx.font = 'Bold 28px "Segoe UI"';
            ctx.fillStyle = '#ffffff';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(label, 128, 55);
            ctx.font = '20px "Segoe UI"';
            ctx.fillStyle = '#dddddd';
            ctx.fillText(`${(w*d).toFixed(0)} m²`, 128, 95);

            const texture = new THREE.CanvasTexture(canvas);
            const spriteMat = new THREE.SpriteMaterial({ map: texture, depthTest: false, depthWrite: false });
            const sprite = new THREE.Sprite(spriteMat);
            sprite.scale.set(3, 1.5, 1);
            sprite.position.set(x + w/2, h + 0.8 + labelOffsetY, z + d/2);
            scene.add(sprite);
        }

        // Fonction pour créer les murs porteurs (plus épais)
        function createWall(x, z, w, d, h, color = 0x888888) {
            const geo = new THREE.BoxGeometry(w, h, d);
            const mat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.5, metalness: 0.3 });
            const wall = new THREE.Mesh(geo, mat);
            wall.position.set(x, h/2, z);
            wall.castShadow = true;
            wall.receiveShadow = true;
            scene.add(wall);
            return wall;
        }

        // Dessiner une ligne de mesure
        function createDimensionLine(x1, z1, x2, z2, y, label, offset = 0.5) {
            const points = [
                new THREE.Vector3(x1, y, z1),
                new THREE.Vector3(x2, y, z2)
            ];
            const geometry = new THREE.BufferGeometry().setFromPoints(points);
            const material = new THREE.LineBasicMaterial({ color: 0xffaa00 });
            const line = new THREE.Line(geometry, material);
            scene.add(line);

            // Petits repères
            const markerMat = new THREE.MeshStandardMaterial({ color: 0xffaa00 });
            const marker1 = new THREE.Mesh(new THREE.SphereGeometry(0.1, 8, 8), markerMat);
            marker1.position.set(x1, y, z1);
            scene.add(marker1);
            const marker2 = new THREE.Mesh(new THREE.SphereGeometry(0.1, 8, 8), markerMat);
            marker2.position.set(x2, y, z2);
            scene.add(marker2);

            // Étiquette de la mesure
            const midX = (x1 + x2) / 2;
            const midZ = (z1 + z2) / 2;
            
            const canvas = document.createElement('canvas');
            canvas.width = 128;
            canvas.height = 64;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = 'rgba(0,0,0,0.7)';
            ctx.roundRect(5, 5, 118, 54, 10);
            ctx.fill();
            ctx.font = 'Bold 22px "Segoe UI"';
            ctx.fillStyle = '#ffaa00';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(label, 64, 35);

            const texture = new THREE.CanvasTexture(canvas);
            const spriteMat = new THREE.SpriteMaterial({ map: texture, depthTest: false, depthWrite: false });
            const sprite = new THREE.Sprite(spriteMat);
            sprite.scale.set(2, 1, 1);
            sprite.position.set(midX, y + 0.8, midZ);
            scene.add(sprite);
        }

        // --- 5. Construction de la maison ---

        // Dimensions globales
        const W = 20; // Largeur (axe X)
        const D = 11; // Profondeur (axe Z)
        const H = 3;  // Hauteur sous plafond

        // 5.1 Sol principal
        const groundGeo = new THREE.PlaneGeometry(W, D);
        const groundMat = new THREE.MeshStandardMaterial({ color: 0x2a2a3a, roughness: 0.9, metalness: 0.1, side: THREE.DoubleSide });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.position.set(W/2, -0.05, D/2);
        ground.receiveShadow = true;
        scene.add(ground);

        // Grille au sol (repères)
        const gridHelper = new THREE.GridHelper(30, 20, 0x44aaff, 0x335588);
        gridHelper.position.set(W/2, 0, D/2);
        scene.add(gridHelper);

        // 5.2 Murs extérieurs (épaisseur 0.2m)
        // Mur avant (z=0)
        createWall(W/2, 0, W, 0.2, H, 0x556677);
        // Mur arrière (z=D)
        createWall(W/2, D, W, 0.2, H, 0x556677);
        // Mur gauche (x=0)
        createWall(0, D/2, 0.2, D, H, 0x556677);
        // Mur droit (x=W)
        createWall(W, D/2, 0.2, D, H, 0x556677);

        // 5.3 Définition des pièces (x, z, w, d, couleur, nom)
        const rooms = [
            // Séjour (bas gauche)
            { x: 0, z: 0, w: 6, d: 6, color: 0x4a90e2, name: 'Séjour' },
            // Cuisine (bas milieu)
            { x: 6, z: 0, w: 3, d: 6, color: 0xf5a623, name: 'Cuisine' },
            // Couloir (bas milieu droit)
            { x: 9, z: 0, w: 2, d: 11, color: 0x9b9b9b, name: 'Couloir' },
            // Chambre 1 (Master) (bas droite)
            { x: 11, z: 0, w: 5, d: 6, color: 0x7ed321, name: 'Chambre 1' },
            // Salle de bain 1 (droite, bas)
            { x: 16, z: 0, w: 3, d: 3, color: 0xd0021b, name: 'Salle de bain 1' },
            // Salle de bain 2 (droite, milieu)
            { x: 16, z: 3, w: 3, d: 3, color: 0xd0021b, name: 'Salle de bain 2' },
            // Chambre 2 (haut gauche)
            { x: 11, z: 6, w: 4, d: 5, color: 0x7ed321, name: 'Chambre 2' },
            // Chambre 3 (haut droite)
            { x: 15, z: 6, w: 4, d: 5, color: 0x7ed321, name: 'Chambre 3' },
            // Buanderie / Dégagement (haut droite extrémité)
            { x: 19, z: 6, w: 1, d: 5, color: 0x9b9b9b, name: 'Buanderie' }
        ];

        // Ajout des pièces
        rooms.forEach(room => {
            createRoom(room.x, room.z, room.w, room.d, H, room.color, room.name);
        });

        // 5.4 Murs intérieurs (pour séparer visuellement les pièces)
        function addInternalWall(x1, z1, x2, z2) {
            const dx = x2 - x1;
            const dz = z2 - z1;
            const length = Math.sqrt(dx*dx + dz*dz);
            const angle = Math.atan2(dx, dz);
            const midX = (x1 + x2) / 2;
            const midZ = (z1 + z2) / 2;
            
            const wall = new THREE.Mesh(
                new THREE.BoxGeometry(0.15, H, length),
                new THREE.MeshStandardMaterial({ color: 0x99aabb, roughness: 0.4, metalness: 0.2 })
            );
            wall.position.set(midX, H/2, midZ);
            wall.rotation.y = angle;
            wall.castShadow = true;
            wall.receiveShadow = true;
            scene.add(wall);
        }

        // Murs intérieurs basés sur la grille
        // Séparation Séjour / Cuisine
        addInternalWall(6, 0, 6, 6);
        // Séparation Cuisine / Couloir
        addInternalWall(9, 0, 9, 6);
        // Séparation Couloir / Chambre 1
        addInternalWall(11, 0, 11, 6);
        // Séparation Chambre 1 / SdB1
        addInternalWall(16, 0, 16, 3);
        // Séparation SdB1 / SdB2
        addInternalWall(16, 3, 16, 6);
        // Séparation Couloir / Chambre 2
        addInternalWall(11, 6, 11, 11);
        // Séparation Chambre 2 / Chambre 3
        addInternalWall(15, 6, 15, 11);
        // Séparation Chambre 3 / Buanderie
        addInternalWall(19, 6, 19, 11);

        // 5.5 Toit (transparent)
        const roofMat = new THREE.MeshStandardMaterial({ 
            color: 0x88aaff, 
            transparent: true, 
            opacity: 0.15, 
            side: THREE.DoubleSide,
            roughness: 0.1,
            metalness: 0.8
        });
        const roof = new THREE.Mesh(new THREE.BoxGeometry(W, 0.1, D), roofMat);
        roof.position.set(W/2, H + 0.05, D/2);
        roof.castShadow = false;
        roof.receiveShadow = false;
        scene.add(roof);

        // Bordures du toit
        const roofEdgeMat = new THREE.LineBasicMaterial({ color: 0x88ccff });
        const roofEdgePoints = [
            new THREE.Vector3(0, H+0.1, 0),
            new THREE.Vector3(W, H+0.1, 0),
            new THREE.Vector3(W, H+0.1, D),
            new THREE.Vector3(0, H+0.1, D),
            new THREE.Vector3(0, H+0.1, 0)
        ];
        const roofEdgeGeo = new THREE.BufferGeometry().setFromPoints(roofEdgePoints);
        const roofEdge = new THREE.Line(roofEdgeGeo, roofEdgeMat);
        scene.add(roofEdge);

        // 5.6 Affichage des mesures clés en 3D
        // Mesure façade (20m)
        createDimensionLine(0, -0.5, W, -0.5, -0.2, '20 m');
        // Mesure profondeur (11m)
        createDimensionLine(-0.5, 0, -0.5, D, -0.2, '11 m');
        // Mesure cuisine (3m x 6m)
        createDimensionLine(6, -0.5, 9, -0.5, 0.5, '3 m');
        createDimensionLine(6, -0.5, 6, 6.5, 0.5, '6 m');

        // 5.7 Axes (optionnel, pour l'orientation)
        // const axesHelper = new THREE.AxesHelper(15);
        // scene.add(axesHelper);

        // --- 6. Animation et rendu ---
        function animate() {
            requestAnimationFrame(animate);
            controls.update(); // pour le damping
            renderer.render(scene, camera);
        }
        animate();

        // --- 7. Gestion du redimensionnement ---
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // Polyfill pour roundRect (si navigateur ancien)
        if (!CanvasRenderingContext2D.prototype.roundRect) {
            CanvasRenderingContext2D.prototype.roundRect = function (x, y, w, h, r) {
                if (r > w/2) r = w/2;
                if (r > h/2) r = h/2;
                this.moveTo(x + r, y);
                this.lineTo(x + w - r, y);
                this.quadraticCurveTo(x + w, y, x + w, y + r);
                this.lineTo(x + w, y + h - r);
                this.quadraticCurveTo(x + w, y + h, x + w - r, y + h);
                this.lineTo(x + r, y + h);
                this.quadraticCurveTo(x, y + h, x, y + h - r);
                this.lineTo(x, y + r);
                this.quadraticCurveTo(x, y, x + r, y);
                return this;
            };
        }

        console.log('🏠 Modèle 3D chargé avec succès !');
    </script>
</body>
</html>
