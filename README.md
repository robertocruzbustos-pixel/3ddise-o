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
            /* Ensures the container has a visible height */
            height: 400px; 
            border-radius: 8px;
            overflow: hidden;
            margin-top: 15px;
            margin-bottom: 15px;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.1);
            background-color: #e0e0e0; /* Added background for visibility */
        }
        canvas {
            display: block;
        }
    </style>
</head>
<body>

    <div id="app-container">
        <h1 class="text-3xl font-bold text-gray-800 mb-2">Visor de Diseño 3D (Implementación Fija)</h1>
        <p class="text-sm text-gray-600 mb-4">¡Conexión a Firebase Exitosa! Haz clic en el botón para guardar el primer diseño.</p>

        <!-- Status message -->
        <div id="status-message" class="p-3 bg-blue-100 text-blue-700 rounded-lg w-full text-center font-medium transition-colors duration-300">
            Inicializando aplicación y autenticación...
        </div>

        <!-- Button to save a test design -->
        <button id="save-button" class="mt-4 px-6 py-2 bg-indigo-600 text-white font-semibold rounded-lg shadow-md hover:bg-indigo-700 transition-all duration-200">
            Guardar Diseño de Prueba
        </button>

        <!-- Container for the 3D scene -->
        <div id="canvas-container"></div>
    </div>

    <!-- Load Firebase modules -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, addDoc, onSnapshot, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // =========================================================================
        // STEP 1: FIXED FIREBASE CONFIGURATION (Using your current configuration)
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
        
        const appId = "3ddiseno-app-fija"; 
        const statusEl = document.getElementById('status-message');
        const saveButton = document.getElementById('save-button');
        
        let db, auth;
        let userId;
        let scene, camera, renderer;
        let currentObject = null; // Reference to the currently displayed 3D object

        // --- CORE THREE.JS FUNCTIONS ---

        // Function to create and add the placeholder object
        function createPlaceholderCube() {
            const geometry = new THREE.BoxGeometry(1, 1, 1);
            const material = new THREE.MeshPhongMaterial({ color: 0xff4500 }); // Orange-Red
            return new THREE.Mesh(geometry, material);
        }

        // Function to create a 3D object from Firestore data
        function createObjectFromData(data) {
            let geometry;
            if (data.shape === 'Sphere') {
                geometry = new THREE.SphereGeometry(1, 32, 32);
            } else {
                // Default to Box
                geometry = new THREE.BoxGeometry(1, 1, 1);
            }
            const material = new THREE.MeshPhongMaterial({ color: data.color || 0x00ff00 });
            return new THREE.Mesh(geometry, material);
        }

        // Function to set up the 3D scene (runs only once)
        function initThreeJS() {
            const container = document.getElementById('canvas-container');
            
            if (container.clientWidth === 0 || container.clientHeight === 0) {
                 console.error("Three.js: Container has zero size. Check CSS height/width.");
                 return;
            }

            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
            renderer = new THREE.WebGLRenderer({ antialias: true });

            renderer.setSize(container.clientWidth, container.clientHeight);
            container.appendChild(renderer.domElement);
            
            console.log("Three.js initialization successful. Renderer added to container.");

            // Add Lights
            const light = new THREE.DirectionalLight(0xffffff, 1);
            light.position.set(5, 5, 5).normalize();
            scene.add(light);
            scene.add(new THREE.AmbientLight(0x404040, 2));

            camera.position.z = 3;

            // Start animation loop
            animate();

            // Handle resizing
            window.addEventListener('resize', () => {
                camera.aspect = container.clientWidth / container.clientHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(container.clientWidth, container.clientHeight);
            });
        }
        
        // Animation loop
        function animate() {
            requestAnimationFrame(animate);

            if (currentObject) {
                // Rotate the currently loaded object
                currentObject.rotation.x += 0.005;
                currentObject.rotation.y += 0.005;
            }

            renderer.render(scene, camera);
        }
        
        // --- FIREBASE WRITE FUNCTION ---
        
        async function saveDesignToFirestore() {
            if (!db) {
                console.error("Firestore no está inicializado.");
                statusEl.textContent = "ERROR: La base de datos no está lista para guardar.";
                return;
            }
            
            saveButton.disabled = true;
            saveButton.textContent = "Guardando...";
            statusEl.classList.remove('bg-blue-100');
            statusEl.classList.add('bg-yellow-100', 'text-yellow-700');

            // Define the simple design data to save
            const testDesignData = {
                shape: 'Sphere',
                color: '#00cc00', // Green color
                name: 'Esfera de Prueba',
                created: new Date().toISOString()
            };

            try {
                const designsCollectionRef = collection(db, "artifacts", appId, "public", "data", "designs");
                await addDoc(designsCollectionRef, testDesignData);
                
                statusEl.textContent = `¡Diseño guardado exitosamente! Recargando datos...`;
                statusEl.classList.remove('bg-yellow-100', 'text-yellow-700');
                statusEl.classList.add('bg-green-100', 'text-green-700');
            } catch (error) {
                console.error("Error al guardar el diseño:", error);
                statusEl.textContent = `ERROR al guardar: ${error.message}`;
                statusEl.classList.remove('bg-yellow-100');
                statusEl.classList.add('bg-red-100', 'text-red-700');
            } finally {
                saveButton.disabled = false;
                saveButton.textContent = "Guardar Diseño de Prueba";
            }
        }


        // --- FIREBASE READ AND RENDER FUNCTION ---

        function initializeDatabase(dbInstance, authInstance) {
            db = dbInstance;
            auth = authInstance;
            userId = auth.currentUser.uid;
            
            console.log("Authentication complete. User ID:", userId);

            // Add the initial placeholder cube
            currentObject = createPlaceholderCube();
            scene.add(currentObject);

            // Attach save handler
            saveButton.addEventListener('click', saveDesignToFirestore);


            const designsCollectionRef = collection(db, "artifacts", appId, "public", "data", "designs");
            
            onSnapshot(designsCollectionRef, (snapshot) => {
                const numDesigns = snapshot.docs.length;
                statusEl.textContent = `Datos cargados. Diseños encontrados: ${numDesigns}. ¡Listo!`;
                statusEl.classList.remove('bg-blue-100', 'bg-red-100', 'bg-yellow-100');
                statusEl.classList.add('bg-green-100', 'text-green-700');

                if (numDesigns > 0) {
                    const latestDesign = snapshot.docs[0].data();
                    
                    // 1. Remove the current object
                    scene.remove(currentObject);
                    
                    // 2. Create the new object from data
                    currentObject = createObjectFromData(latestDesign);
                    
                    // 3. Add the new object to the scene
                    scene.add(currentObject);

                    statusEl.textContent = `Diseño cargado: ${latestDesign.name || latestDesign.shape}. Total: ${numDesigns}`;

                } else {
                    // If no designs, ensure the placeholder cube is showing
                    if (currentObject.geometry.type !== 'BoxGeometry') {
                         scene.remove(currentObject);
                         currentObject = createPlaceholderCube();
                         scene.add(currentObject);
                    }
                    statusEl.textContent = `Datos cargados. Diseños encontrados: 0. Cubo de muestra visible.`;
                    statusEl.classList.remove('bg-green-100', 'text-green-700');
                    statusEl.classList.add('bg-blue-100', 'text-blue-700');
                }

            }, (error) => {
                console.error("Error listening to Firestore data:", error);
                statusEl.textContent = `ERROR de Firestore: ${error.message}. Verifica las reglas de seguridad.`;
                statusEl.classList.remove('bg-blue-100', 'bg-green-100');
                statusEl.classList.add('bg-red-100', 'text-red-700');
            });
        }

        // =========================================================================
        // STEP 2: FIXED INITIALIZATION AND AUTHENTICATION LOGIC
        // =========================================================================
        window.onload = function () {
            // Initialize Three.js first
            initThreeJS(); 

            if (!firebaseConfig.apiKey) {
                // If API key is missing (i.e., still TU_API_KEY_AQUI placeholder)
                statusEl.textContent = 'AVISO: Falta la configuración de Firebase. Se carga solo el visor 3D.';
                console.warn("ADVERTENCIA: Debes reemplazar los placeholders de firebaseConfig para que funcione la base de datos.");
                saveButton.style.display = 'none'; // Hide button if DB is not configured
                return; 
            }

            try {
                // 1. Initialize Firebase
                const app = initializeApp(firebaseConfig);
                const fbAuth = getAuth(app);
                const fbDb = getFirestore(app);
                
                setLogLevel('debug'); 

                // 2. Wait for authentication state change (the "doorman")
                onAuthStateChanged(fbAuth, (user) => {
                    if (user) {
                        // User authenticated
                        initializeDatabase(fbDb, fbAuth);
                    } else {
                        // Attempt anonymous sign-in
                        statusEl.textContent = 'Autenticando sesión anónimamente...';
                        signInAnonymously(fbAuth)
                            .then(() => {}) // onAuthStateChanged will trigger again
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
