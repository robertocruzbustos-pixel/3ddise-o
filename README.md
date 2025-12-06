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
        /* Estilos generales para el cuerpo y el contenedor */
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
            aspect-ratio: 16 / 9; /* Mantiene una relación de aspecto para el 3D */
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

        <!-- Mensaje de estado -->
        <div id="status-message" class="p-3 bg-blue-100 text-blue-700 rounded-lg w-full text-center font-medium">
            Inicializando aplicación y autenticación...
        </div>

        <!-- Contenedor para la escena 3D -->
        <div id="canvas-container"></div>
    </div>

    <!-- Carga los módulos de Firebase -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, onSnapshot, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // =========================================================================
        // PASO 1: CONFIGURACIÓN DE FIREBASE PARA ALOJAMIENTO EXTERNO (IMPORTANTE)
        // *** DEBES REEMPLAZAR TODOS ESTOS VALORES CON LA CONFIGURACIÓN REAL DE TU PROYECTO FIREBASE ***
        // Si dejas los placeholders, la base de datos NO funcionará.
        // =========================================================================
        const firebaseConfig = {
            apiKey: "AIzaSyA1Yy13XMF2oB7KkFJVPOKvp8z01YiuNF0", 
            authDomain: "calculador-de-costos-c4f71.firebaseapp.com",
            projectId: "calculador-de-costos-c4f71",
            storageBucket: "calculador-de-costos-c4f71.firebasestorage.app",
            messagingSenderId: "880952513286",
            appId: "3a6798f1ead2527977c487"
            measurementId: "G-DK98EWYVE9"
        };
        
        // Define un ID de app fijo, ya que __app_id no existe en GitHub Pages
        const appId = "3ddiseno-app-fija"; 
        
        const statusEl = document.getElementById('status-message');
        
        let db, auth;
        let userId;

        // Función para inicializar la escena 3D (parte visual de tu app)
        function initThreeJS() {
            const container = document.getElementById('canvas-container');
            const scene = new THREE.Scene();
            const camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
            const renderer = new THREE.WebGLRenderer({ antialias: true });

            renderer.setSize(container.clientWidth, container.clientHeight);
            container.appendChild(renderer.domElement);

            // Objeto 3D de ejemplo (Cubo)
            const geometry = new THREE.BoxGeometry(1, 1, 1);
            const material = new THREE.MeshPhongMaterial({ color: 0x0056b3 });
            const cube = new THREE.Mesh(geometry, material);
            scene.add(cube);

            // Luces
            const light = new THREE.DirectionalLight(0xffffff, 1);
            light.position.set(5, 5, 5).normalize();
            scene.add(light);
            scene.add(new THREE.AmbientLight(0x404040, 2));

            camera.position.z = 3;

            // Bucle de animación
            function animate() {
                requestAnimationFrame(animate);

                // Animación simple para que se vea activo
                cube.rotation.x += 0.005;
                cube.rotation.y += 0.005;

                renderer.render(scene, camera);
            }
            
            animate();

            // Manejo de redimensionamiento
            window.addEventListener('resize', () => {
                camera.aspect = container.clientWidth / container.clientHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(container.clientWidth, container.clientHeight);
            });
        }

        // Función para inicializar la base de datos y listeners
        function initializeDatabase(dbInstance, authInstance) {
            db = dbInstance;
            auth = authInstance;
            userId = auth.currentUser.uid;
            
            // Muestra el ID de usuario para confirmación (opcional pero útil)
            console.log("Autenticación completada. User ID:", userId);

            // Aquí se inicializan tus listeners de Firestore, por ejemplo:
            // Escucha una colección pública para cargar los diseños 3D
            const designsCollectionRef = collection(db, "artifacts", appId, "public", "data", "designs");
            
            onSnapshot(designsCollectionRef, (snapshot) => {
                statusEl.textContent = `Datos cargados. Diseños encontrados: ${snapshot.docs.length}. ¡Listo!`;
                
                // Aquí iría la lógica para procesar los diseños (snapshot.docs)
                // y actualizar la escena 3D si fuera necesario.

            }, (error) => {
                console.error("Error al escuchar los datos de Firestore:", error);
                statusEl.textContent = `ERROR de Firestore: ${error.message}. Verifica las reglas de seguridad.`;
            });
        }

        // =========================================================================
        // PASO 2: LÓGICA DE INICIALIZACIÓN Y AUTENTICACIÓN FIJA (Se ejecuta automáticamente)
        // =========================================================================
        window.onload = function () {
            initThreeJS(); // Inicia la parte visual inmediatamente

            if (firebaseConfig.apiKey === "TU_API_KEY_AQUI") {
                // Si el usuario no reemplazó la configuración, cargamos solo el 3D
                statusEl.textContent = 'AVISO: Falta la configuración de Firebase. Se carga solo el visor 3D.';
                console.warn("ADVERTENCIA: Debes reemplazar los placeholders de firebaseConfig para que funcione la base de datos.");
                return; 
            }

            try {
                // 1. Inicializar Firebase
                const app = initializeApp(firebaseConfig);
                const fbAuth = getAuth(app);
                const fbDb = getFirestore(app);
                
                setLogLevel('error'); // Establece el nivel de log.

                // 2. Esperar el cambio de estado de autenticación (el "portero")
                onAuthStateChanged(fbAuth, (user) => {
                    if (user) {
                        // Usuario ya autenticado (o sign-in anónimo exitoso)
                        initializeDatabase(fbDb, fbAuth);
                    } else {
                        // No autenticado, intentar sign-in anónimo para obtener un ID temporal
                        statusEl.textContent = 'Autenticando sesión anónimamente...';
                        signInAnonymously(fbAuth)
                            .then(() => {
                                // onAuthStateChanged se disparará de nuevo con el usuario
                            })
                            .catch((error) => {
                                console.error("Error al autenticar anónimamente:", error);
                                statusEl.textContent = `ERROR de Autenticación: ${error.message}`;
                            });
                    }
                });

            } catch (error) {
                console.error("Error durante la inicialización de Firebase:", error);
                statusEl.textContent = `ERROR FATAL: No se pudo inicializar Firebase. ${error.message}`;
            }
        }
    </script>
</body>
</html>
