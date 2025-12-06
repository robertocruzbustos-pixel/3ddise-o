# 3ddise-o
Calculadora de costos Lightbox
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Diseño - Enlace Fijo</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        /* General styles for the body and container */
        body {
            margin: 0;
            padding: 0;
            font-family: 'Inter', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            background-color: #f7f7f7;
        }
        #app-container {
            width: 90%;
            max-width: 1024px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #canvas-container {
            width: 100%;
            aspect-ratio: 16 / 9; /* Maintains an aspect ratio for 3D */
            border-radius: 8px;
            overflow: hidden;
            margin-top: 15px;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.1);
        }
        canvas {
            display: block;
        }
    </style>
</head>
<body>

    <div id="app-container">
        <h1 class="text-3xl font-bold text-gray-800 mb-2">Visor de Diseño 3D (Implementación Fija)</h1>
        <p class="text-sm text-gray-600 mb-4">La conexión a la base de datos se ha ajustado para funcionar en GitHub Pages.</p>

        <!-- Status message -->
        <div id="status-message" class="p-3 bg-blue-100 text-blue-700 rounded-lg w-full text-center font-medium">
            Inicializando aplicación y autenticación...
        </div>

        <!-- Container for the 3D scene -->
        <div id="canvas-container"></div>
    </div>

    <!-- Load Firebase modules -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, onSnapshot, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // =========================================================================
        // STEP 1: FIXED FIREBASE CONFIGURATION
        // =========================================================================
        const firebaseConfig = {
            apiKey: "AIzaSyA1Yy13XMF2oB7KkFJVPOKvp8z01YiuNF0", 
            authDomain: "calculador-de-costos-c4f71.firebaseapp.com",
            projectId: "calculador-de-costos-c4f71",
            storageBucket: "calculador-de-costos-c4f71.firebasestorage.app",
            messagingSenderId: "880952513286",
            appId: "3a6798f1ead2527977c487",
            measurementId: "G-DK98EWYVE9"
        };
        
        // Define a fixed app ID, as __app_id does not exist on GitHub Pages
        const appId = "3ddiseno-app-fija"; 
        
        const statusEl = document.getElementById('status-message');
        
        let db, auth;
        let userId;

        // Function to initialize the 3D scene (visual part of your app)
        function initThreeJS() {
            const container = document.getElementById('canvas-container');
            const scene = new THREE.Scene();
            const camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
            const renderer = new THREE.WebGLRenderer({ antialias: true });

            renderer.setSize(container.clientWidth, container.clientHeight);
            container.appendChild(renderer.domElement);

            // Example 3D object (Cube)
            const geometry = new THREE.BoxGeometry(1, 1, 1);
            const material = new THREE.MeshPhongMaterial({ color: 0x0056b3 });
            const cube = new THREE.Mesh(geometry, material);
            scene.add(cube);

            // Lights
            const light = new THREE.DirectionalLight(0xffffff, 1);
            light.position.set(5, 5, 5).normalize();
            scene.add(light);
            scene.add(new THREE.AmbientLight(0x404040, 2));

            camera.position.z = 3;

            // Animation loop
            function animate() {
                requestAnimationFrame(animate);

                // Simple animation for activity indication
                cube.rotation.x += 0.005;
                cube.rotation.y += 0.005;

                renderer.render(scene, camera);
            }
            
            animate();

            // Handle resizing
            window.addEventListener('resize', () => {
                camera.aspect = container.clientWidth / container.clientHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(container.clientWidth, container.clientHeight);
            });
        }

        // Function to initialize the database and listeners
        function initializeDatabase(dbInstance, authInstance) {
            db = dbInstance;
            auth = authInstance;
            userId = auth.currentUser.uid;
            
            // Show user ID for confirmation (optional but useful)
            console.log("Authentication complete. User ID:", userId);

            // Firestore listeners are initialized here, for example:
            // Listen to a public collection to load 3D designs
            const designsCollectionRef = collection(db, "artifacts", appId, "public", "data", "designs");
            
            onSnapshot(designsCollectionRef, (snapshot) => {
                statusEl.textContent = `Datos cargados. Diseños encontrados: ${snapshot.docs.length}. ¡Listo!`;
                
                // Logic to process the designs (snapshot.docs) would go here
                // and update the 3D scene if necessary.

            }, (error) => {
                console.error("Error listening to Firestore data:", error);
                statusEl.textContent = `ERROR de Firestore: ${error.message}. Verifica las reglas de seguridad.`;
            });
        }

        // =========================================================================
        // STEP 2: FIXED INITIALIZATION AND AUTHENTICATION LOGIC (Runs automatically)
        // =========================================================================
        window.onload = function () {
            initThreeJS(); // Start the visual part immediately

            if (!firebaseConfig.apiKey) {
                statusEl.textContent = 'AVISO: Falta la configuración de Firebase. Se carga solo el visor 3D.';
                console.warn("ADVERTENCIA: Debes reemplazar los placeholders de firebaseConfig para que funcione la base de datos.");
                return; 
            }

            try {
                // 1. Initialize Firebase
                const app = initializeApp(firebaseConfig);
                const fbAuth = getAuth(app);
                const fbDb = getFirestore(app);
                
                setLogLevel('debug'); // <--- LOG LEVEL CHANGED TO DEBUG for better console output

                // 2. Wait for authentication state change (the "doorman")
                onAuthStateChanged(fbAuth, (user) => {
                    if (user) {
                        // User already authenticated (or successful anonymous sign-in)
                        initializeDatabase(fbDb, fbAuth);
                    } else {
                        // Not authenticated, attempt anonymous sign-in to get a temporary ID
                        statusEl.textContent = 'Autenticando sesión anónimamente...';
                        signInAnonymously(fbAuth)
                            .then(() => {
                                // onAuthStateChanged will trigger again with the user
                            })
                            .catch((error) => {
                                console.error("Error during anonymous authentication:", error);
                                statusEl.textContent = `ERROR de Autenticación: ${error.message}`;
                            });
                    }
                });

            } catch (error) {
                console.error("Error during Firebase initialization:", error);
                statusEl.textContent = `ERROR FATAL: No se pudo inicializar Firebase. ${error.message}`;
            }
        }
    </script>
</body>
</html>
