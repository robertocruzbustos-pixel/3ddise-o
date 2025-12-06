# 3ddise-o
Calculadora de costos Lightbox
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora de Costos Lightbox 3D (ID Personalizado - FIX V3 - Reverted LED Naming)</title>
    
    <!-- Tailwind CSS CDN para un diseño responsive y moderno -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f9fb;
        }
        .section-header {
            border-left: 4px solid #3b82f6;
            padding-left: 1rem;
        }
        input[type="number"] {
            /* Ocultar flechas en inputs de número (Chrome/Edge/Safari) */
            appearance: none;
            /* Ocultar flechas en inputs de número (Firefox) */
            -moz-appearance: textfield;
        }
        .card {
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.06);
        }
        .calculated-value {
            background-color: #e0f2fe; /* Fondo azul claro para claridad */
            color: #0b69a6;
            padding: 0.5rem;
            border-radius: 0.5rem;
            font-weight: 600;
            margin-top: 0.5rem;
            display: block;
        }
        /* Estilo para el botón de eliminar, usa SVG */
        .remove-btn {
            background: none;
            border: none;
            cursor: pointer;
            padding: 0.25rem;
            border-radius: 0.375rem;
        }
        .remove-btn:hover {
            background-color: #fee2e2;
        }
        .loading-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(255, 255, 255, 0.95); /* Más opaco */
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            z-index: 1000;
        }
    </style>
</head>
<body class="p-4 md:p-8">
    
    <!-- Overlay de carga inicial -->
    <div id="loading-overlay" class="loading-overlay">
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-blue-500"></div>
        <p class="mt-4 text-blue-600 font-semibold">Cargando datos y autenticando sesión...</p>
    </div>

    <div id="app" class="max-w-4xl mx-auto hidden">
        <div class="flex flex-col md:flex-row md:justify-between md:items-start mb-6 border-b pb-4">
            <h1 class="text-3xl font-bold text-gray-800 mb-4 md:mb-0">LightBox 3D: Costos</h1>
            
            <div class="text-left md:text-right text-sm text-gray-700 bg-blue-50 p-3 rounded-lg border border-blue-200 w-full md:w-96">
                <p class="font-semibold text-blue-600 mb-1">ID de Datos Personalizado</p>
                <p class="mb-2 text-xs text-gray-600">Usa un ID corto (ej: "MiTienda") para guardar y cargar tus costos.</p>
                
                <div class="flex flex-col sm:flex-row gap-2">
                    <input type="text" id="custom-id-input" placeholder="Ingresa tu ID corto" class="p-2 border border-gray-300 rounded-lg flex-grow text-sm">
                    <button onclick="loadDataByCustomKey()" id="load-id-button" class="py-1.5 px-3 bg-blue-500 text-white font-semibold rounded-lg hover:bg-blue-600 transition duration-150 ease-in-out whitespace-nowrap">
                        Cargar / Establecer ID
                    </button>
                </div>

                <p id="current-custom-id" class="mt-2 font-mono text-sm text-blue-800 bg-blue-200 p-2 rounded-lg break-all hidden">ID Activo: Cargando...</p>

                <p id="auth-status" class="text-xs mt-2 font-medium text-gray-600">Sesión autenticada. (Establece un ID para guardar)</p>
                
                <button onclick="saveDataToFirestore(true)" id="save-button" class="mt-3 w-full py-1.5 px-3 bg-green-500 text-white font-semibold rounded-lg hover:bg-green-600 transition duration-150 ease-in-out">
                    Guardar Ahora (Guardado Automático OK)
                </button>
            </div>
        </div>

        <!-- Pestañas de Navegación -->
        <div class="flex border-b border-gray-200 mb-6 overflow-x-auto">
            <button onclick="showSection('base-costs')" class="tab-btn p-3 text-sm font-medium text-gray-600 border-b-2 border-transparent hover:border-blue-500 transition duration-150 ease-in-out whitespace-nowrap" data-target="base-costs">
                1. Costos Base (Fijos)
            </button>
            <button onclick="showSection('model-input')" class="tab-btn p-3 text-sm font-medium text-gray-600 border-b-2 border-transparent hover:border-blue-500 transition duration-150 ease-in-out whitespace-nowrap" data-target="model-input">
                2. Consumo por Modelo
            </button>
            <button onclick="showSection('results')" class="tab-btn p-3 text-sm font-medium text-gray-600 border-b-2 border-blue-500 text-blue-600 whitespace-nowrap" data-target="results">
                3. Resultado Final
            </button>
        </div>

        <!-- 1. Sección de Costos Base (Hoja 1 y 2) -->
        <div id="base-costs" class="tab-content hidden space-y-6">
            <div class="card bg-white p-6 rounded-lg">
                <h2 class="text-xl font-semibold text-gray-700 section-header mb-4">A. Materiales Unitarios (Por Medida)</h2>
                <p class="text-sm text-gray-500 mb-4">Ingresa el costo total de compra y la cantidad para calcular el costo por unidad de medida.</p>
                
                <!-- Filamento PLA -->
                <div class="space-y-2 mb-4 p-3 border rounded-lg">
                    <h3 class="font-medium text-gray-700">Filamento PLA (Gramos)</h3>
                    <label class="block text-sm text-gray-600">Costo de Compra (Total ARS):</label>
                    <input type="number" data-key="plaTotal" id="input-pla-total" value="30000" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <label class="block text-sm text-gray-600">Cantidad Comprada (Gramos):</label>
                    <input type="number" data-key="plaQty" id="input-pla-qty" value="1000" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <span class="calculated-value" id="calc-pla-gram">Costo/g: $ 30.00</span>
                </div>

                <!-- LED Tira Blanca -->
                <div class="space-y-2 mb-4 p-3 border rounded-lg">
                    <h3 class="font-medium text-gray-700">LED Tira Blanca (Metros comprados)</h3>
                    <p class="text-xs text-gray-500">Se asume una densidad de 120 LEDs por metro para el cálculo unitario.</p>
                    <label class="block text-sm text-gray-600">Costo de Compra (Total ARS):</label>
                    <input type="number" data-key="ledTotal" id="input-led-total" value="1250" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <label class="block text-sm text-gray-600">Cantidad Comprada (Metros):</label>
                    <input type="number" data-key="ledQty" id="input-led-qty" value="5" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <span class="calculated-value" id="calc-led-m">Costo/LED (120/m): $ X.XX</span>
                </div>

                <!-- LED Tira RGB -->
                <div class="space-y-2 mb-4 p-3 border rounded-lg">
                    <h3 class="font-medium text-gray-700">LED Tira RGB (Metros comprados)</h3>
                    <p class="text-xs text-gray-500">Se asume una densidad de 60 LEDs por metro para el cálculo unitario.</p>
                    <label class="block text-sm text-gray-600">Costo de Compra (Total ARS):</label>
                    <input type="number" data-key="ledRgbTotal" id="input-led-rgb-total" value="1750" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <label class="block text-sm text-gray-600">Cantidad Comprada (Metros):</label>
                    <input type="number" data-key="ledRgbQty" id="input-led-rgb-qty" value="5" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <span class="calculated-value" id="calc-led-rgb-m">Costo/LED (60/m): $ X.XX</span>
                </div>
                
                <!-- Fuente 12V -->
                <div class="space-y-2 mb-4 p-3 border rounded-lg">
                    <h3 class="font-medium text-gray-700">Fuente 12V (Unidades)</h3>
                    <label class="block text-sm text-gray-600">Costo de Compra (Total ARS):</label>
                    <input type="number" data-key="psTotal" id="input-ps-total" value="1200" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <label class="block text-sm text-gray-600">Cantidad Comprada (Unidades):</label>
                    <input type="number" data-key="psQty" id="input-ps-qty" value="1" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <span class="calculated-value" id="calc-ps-unit">Costo/u: $ 1200.00</span>
                </div>
                
                <!-- Embalaje/Caja -->
                <div class="space-y-2 p-3 border rounded-lg">
                    <h3 class="font-medium text-gray-700">Embalaje/Caja (Unidades)</h3>
                    <label class="block text-sm text-gray-600">Costo de Compra (Total ARS):</label>
                    <input type="number" data-key="pkgTotal" id="input-pkg-total" value="500" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <label class="block text-sm text-gray-600">Cantidad Comprada (Unidades):</label>
                    <input type="number" data-key="pkgQty" id="input-pkg-qty" value="1" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <span class="calculated-value" id="calc-pkg-unit">Costo/u: $ 500.00</span>
                </div>
            </div>

            <div class="card bg-white p-6 rounded-lg">
                <h2 class="text-xl font-semibold text-gray-700 section-header mb-4">B. Costos Fijos y de Producción (Derivados)</h2>
                
                <!-- Mano de Obra Directa -->
                <div class="space-y-2 mb-4 p-3 border rounded-lg">
                    <h3 class="font-medium text-gray-700">Mano de Obra Directa (MOD)</h3>
                    <label class="block text-sm text-gray-600">Sueldo Asignado a M.O. (Mensual ARS):</label>
                    <input type="number" data-key="moSalary" id="input-mo-salary" value="400000" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <label class="block text-sm text-gray-600">Horas de Trabajo Mensual:</label>
                    <input type="number" data-key="moHours" id="input-mo-hours" value="160" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <span class="calculated-value" id="calc-mo-min">Costo/min: $ 41.67</span>
                </div>

                <!-- Energía/Electricidad (Impresión) -->
                <div class="space-y-2 mb-4 p-3 border rounded-lg">
                    <h3 class="font-medium text-gray-700">Costo de Energía (Impresión)</h3>
                    <label class="block text-sm text-gray-600">Costo por kWh (ARS):</label>
                    <input type="number" data-key="energyKwh" id="input-energy-kwh" value="30" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <label class="block text-sm text-gray-600">Consumo de Impresora (kW):</label>
                    <input type="number" data-key="energyKw" id="input-energy-kw" value="0.1" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <span class="calculated-value" id="calc-energy-min">Costo/min: $ 0.05</span>
                </div>

                <!-- Costos Fijos Indirectos (CIF) - Desglosado -->
                <div class="space-y-2 p-3 border rounded-lg">
                    <h3 class="font-medium text-gray-700 mb-2">Costos Fijos Asignados (CIF)</h3>
                    <p class="text-xs text-gray-500 mb-2">Ingresa el detalle de tus costos fijos mensuales. El total se calculará automáticamente y se prorrateará.</p>
                    
                    <!-- Detalle de Costos Fijos Mensuales -->
                    <div class="space-y-2 border-l-2 border-indigo-300 pl-3 py-1">
                        <label class="block text-sm text-gray-600">Renta o Hipoteca (ARS):</label>
                        <input type="number" data-key="cifRenta" id="input-cif-renta" value="10000" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                        
                        <label class="block text-sm text-gray-600">Servicios (Agua, Internet, etc. ARS):</label>
                        <input type="number" data-key="cifServicios" id="input-cif-servicios" value="3000" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                        
                        <label class="block text-sm text-gray-600">Impuestos y Licencias (ARS):</label>
                        <input type="number" data-key="cifImpuestos" id="input-cif-impuestos" value="2000" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    </div>

                    <label class="block text-sm text-gray-600 pt-2">Total Costos Fijos Mensuales (ARS):</label>
                    <!-- Campo calculado -->
                    <input type="number" id="input-cif-total" value="15000" disabled class="w-full p-2 border border-blue-300 rounded-lg bg-blue-50 font-bold text-gray-800">

                    <label class="block text-sm text-gray-600">Producción Estimada Mensual (Unidades):</label>
                    <input type="number" data-key="cifProduction" id="input-cif-production" value="100" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                    <span class="calculated-value" id="calc-cif-unit">CIF/Unidad: $ 150.00</span>
                </div>
            </div>
        </div>

        <!-- 2. Sección de Consumo por Modelo (Matriz de Consumo) -->
        <div id="model-input" class="tab-content hidden space-y-6">
            <h2 class="text-2xl font-semibold text-gray-700 section-header mb-4">2. Consumo por Modelo</h2>
            <p class="text-sm text-gray-500 mb-4">Define el consumo específico de cada modelo. Los costos base se toman de la pestaña anterior.</p>

            <div id="model-list" class="space-y-4">
                <!-- Los modelos se renderizarán aquí -->
            </div>

            <button onclick="addModel()" class="w-full py-2 px-4 border border-blue-500 text-blue-500 font-semibold rounded-lg hover:bg-blue-50 transition duration-150 ease-in-out">
                + Agregar Nuevo Modelo
            </button>
        </div>

        <!-- 3. Sección de Resultados (Resumen y Cálculo) -->
        <div id="results" class="tab-content space-y-6">
            <h2 class="text-2xl font-semibold text-gray-700 section-header mb-4">3. Resumen y Precios Finales</h2>

            <div class="card bg-white p-6 rounded-lg space-y-4">
                <label class="block">Modelo a Calcular:</label>
                <select id="select-model" onchange="displayResults()" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500"></select>

                <div class="pt-4 border-t border-gray-200">
                    <label class="block font-medium">Margen de Ganancia Deseado (%)</label>
                    <input type="number" data-key="margin" id="input-margin" value="35" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg mb-4">

                    <label class="block font-medium">Comisiones/Impuestos por Venta (%)</label>
                    <input type="number" data-key="commissions" id="input-commissions" value="15" oninput="updateBaseCostsDisplay()" class="w-full p-2 border border-gray-300 rounded-lg">
                </div>
            </div>

            <div id="result-display" class="card bg-white p-6 rounded-lg">
                <!-- Resultados se muestran aquí -->
                <p class="text-center text-gray-500">Selecciona o agrega un modelo para ver el cálculo.</p>
            </div>
        </div>
    </div>

    <!-- Custom Modal for Alerts (replacing native alert/confirm) -->
    <div id="custom-modal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center p-4 z-50">
        <div class="bg-white p-6 rounded-lg shadow-2xl max-w-sm w-full">
            <h3 id="modal-title" class="text-xl font-semibold text-gray-800 mb-4">Atención</h3>
            <p id="modal-message" class="text-gray-600 mb-6">Mensaje de la aplicación.</p>
            <div class="flex justify-end space-x-3">
                <button onclick="closeModal()" class="px-4 py-2 bg-blue-500 text-white font-medium rounded-lg hover:bg-blue-600 transition duration-150">Cerrar</button>
            </div>
        </div>
    </div>

    <script type="module">
        // Importaciones de Firebase
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Establecer nivel de log para depuración de Firestore
        setLogLevel('Debug');
        
        // --- CONSTANTES GLOBALES Y DE FIREBASE ---
        const LEDS_PER_SEGMENT = 3;    // Los sectores de corte son de 3 LEDs
        const WHITE_LED_DENSITY = 120; // LEDs por metro en tira blanca
        const RGB_LED_DENSITY = 60;    // LEDs por metro en tira RGB

        let db = null;
        let auth = null;
        let sessionUid = null; 
        let dataUid = null;    
        let currentCustomId = null; 
        
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        // Asegurarse de que firebaseConfig sea un objeto, incluso si __firebase_config no existe o es inválido
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        
        const initialModels = [ // Modelos de ejemplo si no hay datos guardados
            {
                id: Date.now(),
                name: "Modelo Básico (Pequeño)",
                ledType: 'white', 
                consumption: { pla_grams: 80, led_quantity: 20, mo_minutes: 15, print_hours: 3, print_minutes: 0, }
            },
            {
                id: Date.now() + 1,
                name: "Modelo Grande (RGB)",
                ledType: 'rgb', 
                consumption: { pla_grams: 200, led_quantity: 24, mo_minutes: 25, print_hours: 8, print_minutes: 0, }
            }
        ];
        
        // Estado local de los modelos
        let models = initialModels;

        // --- FUNCIONES DE PERSISTENCIA DE DATOS (FIREBASE) ---

        /**
         * Retorna la referencia al documento Firestore privado del usuario activo.
         */
        function getDataDocRef(uid) {
            if (!db || !uid) {
                console.error("Firebase no está inicializado o el UID de datos no está definido.");
                return null;
            }
            // Ruta: /artifacts/{appId}/users/{uid}/cost_data/data_doc
            return doc(db, 'artifacts', appId, 'users', uid, 'cost_data', 'data_doc');
        }

        /**
         * Retorna la referencia al documento de mapeo de ID público.
         */
        function getMappingDocRef(customId) {
            if (!db) return null;
            // Ruta: /artifacts/{appId}/public/data/id_mapping/{customId}
            const cleanId = customId.trim().toLowerCase().replace(/[^a-z0-9]/g, '');
            if (!cleanId) return null;
            return doc(db, 'artifacts', appId, 'public', 'data', 'id_mapping', cleanId);
        }

        /**
         * Recolecta todos los valores de los inputs de costo base y los márgenes.
         */
        function gatherDataForSaving() {
            const data = {};
            // Recolectar inputs de Costos Base y Resultados (usando data-key)
            document.querySelectorAll('[data-key]').forEach(input => {
                data[input.dataset.key] = parseFloat(input.value) || 0;
            });

            return {
                baseCostsInputs: data,
                models: models
            };
        }

        /**
         * Guarda los datos actuales (costos base y modelos) en Firestore.
         * @param {boolean} isManual - Indica si el guardado fue forzado por el usuario.
         */
        async function saveDataToFirestore(isManual = false) {
            if (!dataUid || !currentCustomId || !db) {
                if (isManual) customAlert("Asegúrate de que la DB esté conectada y hayas establecido un 'ID de Datos Personalizado'.", "Guardado Deshabilitado");
                return;
            }

            const saveButton = document.getElementById('save-button');
            const originalText = saveButton.innerHTML;
            
            try {
                if (isManual) {
                    saveButton.innerHTML = 'Guardando...';
                    saveButton.disabled = true;
                }

                const dataToSave = gatherDataForSaving();
                const docRef = getDataDocRef(dataUid);
                
                if (docRef) {
                    // 1. Guardar los datos de costos en el documento privado del dataUid
                    await setDoc(docRef, dataToSave);
                    
                    // 2. Asegurar que el mapeo público exista (Custom ID -> dataUid)
                    const mappingRef = getMappingDocRef(currentCustomId);
                    if (mappingRef) {
                         await setDoc(mappingRef, { uid: dataUid, last_saved: new Date() }, { merge: true });
                    }

                    if (isManual) {
                        saveButton.innerHTML = '¡Guardado con Éxito!';
                        setTimeout(() => {
                            saveButton.innerHTML = originalText;
                            saveButton.disabled = false;
                        }, 2000);
                    } else {
                        saveButton.innerHTML = 'Guardado Automático OK';
                    }
                }
            } catch (e) {
                console.error("Error al guardar en Firestore:", e);
                if (isManual) {
                    customAlert(`Error al guardar los datos: ${e.message}`, 'Error de Guardado');
                    saveButton.innerHTML = 'Error de Guardado';
                    setTimeout(() => {
                        saveButton.innerHTML = originalText;
                        saveButton.disabled = false;
                    }, 3000);
                }
            }
        }

        /**
         * Muestra la app y oculta el overlay de carga.
         * @param {string} message - Mensaje de estado a mostrar.
         */
        const hideOverlayAndShowApp = (message) => {
            document.getElementById('auth-status').innerText = message;
            document.getElementById('loading-overlay').classList.add('hidden');
            document.getElementById('app').classList.remove('hidden');
        };


        /**
         * Intenta cargar los datos usando el UID de datos proporcionado.
         * @param {string} uid - El UID de Firestore a usar.
         * @param {boolean} isFirstLoad - Si es la carga inicial después de la autenticación.
         */
        async function loadDataFromUid(uid, isFirstLoad = false) {
            if (!db || !uid) {
                // Esto solo debería pasar si initFirebase falló críticamente, pero lo manejamos
                if (isFirstLoad) hideOverlayAndShowApp('DB no inicializada. Trabajando sin persistencia.');
                return;
            }

            const docRef = getDataDocRef(uid);
            if (!docRef) {
                if (isFirstLoad) hideOverlayAndShowApp('Error interno al preparar la base de datos.');
                return;
            }

            try {
                const docSnap = await getDoc(docRef);

                if (docSnap.exists()) {
                    const data = docSnap.data();
                    applyLoadedData(data);
                    document.getElementById('auth-status').innerText = 'Datos de ' + (currentCustomId || 'Sesión Temporal') + ' cargados con éxito.';
                } else {
                    // Si no hay datos, inicializar con los ejemplos y guardar bajo este nuevo UID.
                    console.log("No se encontraron datos para este UID. Usando valores iniciales y guardando.");
                    document.getElementById('auth-status').innerText = (currentCustomId ? `ID '${currentCustomId}' establecido.` : 'Sesión iniciada.') + ' Usando valores iniciales.';
                    
                    // Aplicar valores por defecto a los inputs y luego guardar
                    applyLoadedData({ 
                        baseCostsInputs: gatherDataForSaving().baseCostsInputs, // Obtiene valores iniciales del HTML
                        models: initialModels 
                    });
                    await saveDataToFirestore();
                }
            } catch (e) {
                console.error("Error al cargar de Firestore:", e);
                customAlert(`Error al cargar los datos: ${e.message}`, 'Error de Carga');
                document.getElementById('auth-status').innerText = `Error al cargar datos. Trabajando en modo solo lectura.`;
            } finally {
                if (isFirstLoad) {
                    // Ocultar overlay y mostrar app solo en la carga inicial
                    hideOverlayAndShowApp('Sesión iniciada. Listo para calcular.');
                }
            }
        }

        /**
         * Lógica principal para establecer o cargar un Custom ID.
         */
        async function loadDataByCustomKey() {
            if (!sessionUid || !db) {
                customAlert("Por favor, espera a que se conecte la DB y se autentique la sesión inicial.", "Aún Cargando");
                return;
            }

            const input = document.getElementById('custom-id-input');
            const requestedKey = input.value.trim();

            if (!requestedKey) {
                customAlert("Debes ingresar un nombre corto (ej: 'MiTienda') para poder guardar o cargar tus datos.", "ID Requerido");
                return;
            }
            
            // Limpiar el ID para el path de Firestore
            const cleanKey = requestedKey.toLowerCase().replace(/[^a-z0-9]/g, '');

            if (cleanKey.length < 3) {
                customAlert("El ID de Datos Personalizado debe contener al menos 3 caracteres alfanuméricos.", "ID Invalido");
                return;
            }

            const mappingRef = getMappingDocRef(cleanKey);
            document.getElementById('auth-status').innerText = `Buscando datos de '${requestedKey}'...`;
            document.getElementById('load-id-button').disabled = true;
            
            try {
                const mappingSnap = await getDoc(mappingRef);

                if (mappingSnap.exists()) {
                    // 1. El ID Personalizado YA existe. Usar el UID asociado.
                    const mappingData = mappingSnap.data();
                    dataUid = mappingData.uid; // Usar el UID asociado
                    currentCustomId = requestedKey;
                    
                    document.getElementById('auth-status').innerText = `ID encontrado. Cargando datos de '${requestedKey}'.`;
                    
                    // 2. Cargar los datos del UID encontrado
                    await loadDataFromUid(dataUid, false); 

                } else {
                    // 1. El ID Personalizado NO existe. Asignar el ID de sesión actual.
                    dataUid = sessionUid; // Usar el UID actual para guardar los nuevos datos
                    currentCustomId = requestedKey;
                    
                    // 2. Crear el mapeo público (Custom ID -> sessionUid)
                    await setDoc(mappingRef, { uid: sessionUid, created_at: new Date() });

                    document.getElementById('auth-status').innerText = `ID '${requestedKey}' establecido y guardado. Inicia sesión con este ID la próxima vez.`;
                    
                    // 3. Cargar los datos (que serán los iniciales, o los que estaban en la sesión actual)
                    await loadDataFromUid(dataUid, false);
                }
                
                document.getElementById('current-custom-id').innerText = `ID Activo: ${currentCustomId}`;
                document.getElementById('current-custom-id').classList.remove('hidden');

            } catch (e) {
                console.error("Error en loadDataByCustomKey:", e);
                customAlert(`Error al cargar/establecer el ID: ${e.message}`, 'Error');
            } finally {
                document.getElementById('load-id-button').disabled = false;
            }
        }

        /**
         * Aplica los datos cargados a los inputs HTML y a la variable 'models'.
         * @param {Object} data - Datos cargados de Firestore.
         */
        function applyLoadedData(data) {
            // 1. Aplicar Costos Base e Inputs
            if (data.baseCostsInputs) {
                for (const key in data.baseCostsInputs) {
                    const input = document.querySelector(`[data-key="${key}"]`);
                    if (input) {
                        input.value = data.baseCostsInputs[key];
                    }
                }
            }
            
            // 2. Aplicar Modelos
            if (Array.isArray(data.models)) {
                models = data.models;
            }

            // 3. Re-renderizar todo
            updateBaseCostsDisplay(); // Actualiza cálculos de costos unitarios
            renderModelInputs();      // Vuelve a dibujar la lista de modelos
            displayResults();         // Muestra el resultado
        }

        // --- FIREBASE INICIALIZACIÓN Y AUTENTICACIÓN (REFORZADA 2.0) ---

        async function initFirebase() {
            try {
                // 1. Validación de Configuración (previene fallos en initializeApp)
                if (!firebaseConfig || Object.keys(firebaseConfig).length === 0 || !firebaseConfig.apiKey) {
                    throw new Error("La configuración de Firebase es incompleta o no está disponible. Guardado/Carga deshabilitado.");
                }

                // 2. Inicializar Firebase
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                // 3. Autenticar y esperar el estado
                const authToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                
                // Intento de autenticación (si falla, onAuthStateChanged manejará el resultado)
                if (authToken) {
                    await signInWithCustomToken(auth, authToken).catch(e => {
                        console.warn("Fallo en signInWithCustomToken. Intentando anónimo.", e);
                        return signInAnonymously(auth);
                    });
                } else {
                    await signInAnonymously(auth);
                }
                
                // 4. Esperar el cambio de estado de autenticación
                await new Promise((resolve, reject) => {
                    const unsubscribe = onAuthStateChanged(auth, async (user) => {
                        unsubscribe(); 
                        if (user) {
                            sessionUid = user.uid; 
                            dataUid = user.uid;  
                            // loadDataFromUid will show the UI and status
                            await loadDataFromUid(sessionUid, true); 
                            resolve();
                        } else {
                            // Authentication failed state (no user)
                            sessionUid = null;
                            dataUid = null;
                            reject(new Error("La autenticación de Firebase falló."));
                        }
                    }, (error) => {
                         // Error listener for onAuthStateChanged
                        reject(error);
                    });
                });

            } catch (e) {
                // --- FALLO CRÍTICO DE CONEXIÓN O CONFIGURACIÓN ---
                console.error("Error crítico al inicializar o conectar con DB:", e);
                
                // 1. Inicializar UI con datos de ejemplo
                models = initialModels;
                applyLoadedData({ 
                    baseCostsInputs: gatherDataForSaving().baseCostsInputs,
                    models: models 
                });
                
                // 2. Deshabilitar la funcionalidad de guardar/cargar
                db = null; // Marcar la DB como no inicializada
                dataUid = null; 
                document.getElementById('save-button').disabled = true;
                document.getElementById('load-id-button').disabled = true;
                
                // 3. Mostrar la aplicación inmediatamente con un mensaje de error.
                hideOverlayAndShowApp(`ERROR CRÍTICO: ${e.message}. Trabajando sin conexión a DB (solo lectura).`);
            }
        }

        // --- CORE LOGIC (VINCULACIÓN HOJAS) ---
        
        /** Formatea un número como moneda ARS. */
        const formatCurrency = (value) => {
            return `$ ${Math.max(0, value).toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",")}`;
        };

        /** Muestra el modal personalizado con un mensaje de alerta. */
        function customAlert(message, title = "Atención") {
            document.getElementById('modal-title').innerText = title;
            document.getElementById('modal-message').innerText = message;
            document.getElementById('custom-modal').classList.remove('hidden');
            document.getElementById('custom-modal').classList.add('flex');
        }

        /** Cierra el modal personalizado. */
        function closeModal() {
            document.getElementById('custom-modal').classList.add('hidden');
            document.getElementById('custom-modal').classList.remove('flex');
        }


        /**
         * Calcula y actualiza los costos unitarios en la pestaña 1 (Costos Base).
         * @returns {Object} Los costos base unitarios calculados.
         */
        function calculateBaseUnitCosts() {
            // Función auxiliar para parsear y manejar errores (evita NaN)
            const getVal = (id) => parseFloat(document.getElementById(id)?.value) || 0;
            const safeDiv = (numerator, denominator) => denominator > 0 ? numerator / denominator : 0;

            // A. Materiales Unitarios
            const plaTotal = getVal('input-pla-total');
            const plaQty = getVal('input-pla-qty');
            const ledTotal = getVal('input-led-total');
            const ledQty = getVal('input-led-qty');
            const ledRgbTotal = getVal('input-led-rgb-total');
            const ledRgbQty = getVal('input-led-rgb-qty');
            const psTotal = getVal('input-ps-total');
            const psQty = getVal('input-ps-qty');
            const pkgTotal = getVal('input-pkg-total');
            const pkgQty = getVal('input-pkg-qty');

            const costPla = safeDiv(plaTotal, plaQty);
            const costPs = safeDiv(psTotal, psQty);
            const costPkg = safeDiv(pkgTotal, pkgQty);
            
            // CÁLCULO DE COSTO DE LED POR UNIDAD
            const totalWhiteLeds = ledQty * WHITE_LED_DENSITY; 
            const totalRgbLeds = ledRgbQty * RGB_LED_DENSITY;

            const costLedUnitWhite = safeDiv(ledTotal, totalWhiteLeds);
            const costLedUnitRgb = safeDiv(ledRgbTotal, totalRgbLeds);
            
            // B. Costos Fijos y de Producción
            const moSalary = getVal('input-mo-salary');
            const moHours = getVal('input-mo-hours');
            const energyKwh = getVal('input-energy-kwh');
            const energyKw = getVal('input-energy-kw');

            // ** CIF - Cálculo del total a partir del desglose **
            const cifRenta = getVal('input-cif-renta');
            const cifServicios = getVal('input-cif-servicios');
            const cifImpuestos = getVal('input-cif-impuestos');
            
            const cifTotalCalculated = cifRenta + cifServicios + cifImpuestos;
            document.getElementById('input-cif-total').value = cifTotalCalculated.toFixed(2); // Mostrar el total calculado

            const cifProduction = getVal('input-cif-production');

            // Costo por minuto de Mano de Obra
            const costMoMin = safeDiv(moSalary, (moHours * 60));
            // Costo de Energía por minuto: (Costo/kWh * Consumo/kW) / 60 minutos
            const costEnergyMin = safeDiv((energyKwh * energyKw), 60); 
            // Costo Fijo Indirecto por Unidad
            const costCifUnit = safeDiv(cifTotalCalculated, cifProduction);

            // Actualizar la interfaz de costos unitarios
            document.getElementById('calc-pla-gram').innerText = `Costo/g: ${formatCurrency(costPla)}`;
            document.getElementById('calc-led-m').innerText = `Costo/LED (120/m): ${formatCurrency(costLedUnitWhite)}`;
            document.getElementById('calc-led-rgb-m').innerText = `Costo/LED (60/m): ${formatCurrency(costLedUnitRgb)}`;
            document.getElementById('calc-ps-unit').innerText = `Costo/u: ${formatCurrency(costPs)}`;
            document.getElementById('calc-pkg-unit').innerText = `Costo/u: ${formatCurrency(costPkg)}`;
            document.getElementById('calc-mo-min').innerText = `Costo/min: ${formatCurrency(costMoMin)}`;
            document.getElementById('calc-energy-min').innerText = `Costo/min: ${formatCurrency(costEnergyMin)}`;
            document.getElementById('calc-cif-unit').innerText = `CIF/Unidad: ${formatCurrency(costCifUnit)}`;

            return {
                pla_gram: costPla,
                led_unit_white: costLedUnitWhite, 
                led_unit_rgb: costLedUnitRgb,   
                power_supply: costPs,
                packaging: costPkg,
                mo_min: costMoMin,
                energy_min: costEnergyMin,
                cif_unit: costCifUnit,
            };
        }

        /**
         * Obtiene todos los costos unitarios calculados y los márgenes.
         * @returns {Object} Costos unitarios y márgenes.
         */
        function getBaseCosts() {
            const calculatedCosts = calculateBaseUnitCosts();
            
            return {
                ...calculatedCosts,
                margin: (parseFloat(document.getElementById('input-margin')?.value) || 0) / 100,
                commissions: (parseFloat(document.getElementById('input-commissions')?.value) || 0) / 100,
            };
        }

        /**
         * Calcula el costo unitario total para un modelo específico.
         * @param {Object} model - Objeto del modelo con su consumo.
         * @param {Object} costs - Objeto con los costos base unitarios.
         * @returns {Object} Desglose y CUF (Costo Unitario de Fabricación).
         */
        function calculateModelCost(model, costs) {
            
            // 1. Determinar el costo por UNIDAD de LED
            const isRGB = model.ledType === 'rgb';
            const costPerLedUnit = isRGB ? costs.led_unit_rgb : costs.led_unit_white;

            // 2. Obtener la cantidad total de LEDs consumidos
            const totalLedsConsumed = model.consumption.led_quantity * LEDS_PER_SEGMENT;

            // 3. Obtener el tiempo total de impresión en minutos
            const totalPrintMinutes = (model.consumption.print_hours * 60) + model.consumption.print_minutes;

            // A. Costo Materiales Directos
            const costPla = model.consumption.pla_grams * costs.pla_gram;
            const costLeds = totalLedsConsumed * costPerLedUnit;
            const costMaterials = (
                costPla +
                costLeds + 
                costs.power_supply + 
                costs.packaging 
            );

            // B. Costos Variables de Producción (MOD + Energía)
            const costMOD = model.consumption.mo_min * costs.mo_min;
            const costEnergy = totalPrintMinutes * costs.energy_min;
            const costProduction = costMOD + costEnergy;

            // C. Costos Fijos Asignados (CIF)
            const costCIF = costs.cif_unit;

            // CUF: Costo Unitario de Fabricación
            const CUF = costMaterials + costProduction + costCIF;

            // D. Precio Final
            const suggestedPrice = CUF * (1 + costs.margin);
            // Fórmula: Precio Final = Precio Sugerido / (1 - Comisiones)
            const finalPrice = costs.commissions < 1 ? suggestedPrice / (1 - costs.commissions) : suggestedPrice;

            return {
                CUF,
                costPla,
                costLeds,
                costMaterials,
                costProduction,
                costMOD,
                costEnergy,
                costCIF,
                suggestedPrice,
                finalPrice,
            };
        }

        // --- RENDER FUNCTIONS ---

        /**
         * Renderiza la interfaz para ingresar el consumo de cada modelo.
         */
        function renderModelInputs() {
            const container = document.getElementById('model-list');
            container.innerHTML = ''; // Limpiar

            models.forEach(model => {
                const currentLedType = model.ledType;
                const modelCard = document.createElement('div');
                modelCard.className = 'card bg-white p-6 rounded-lg border-l-4 border-blue-500 space-y-3';
                
                // Usamos una función lambda para asegurar que saveDataToFirestore se llama en la actualización
                const getUpdateFunction = (key, isConsumption = true) => (
                    isConsumption 
                        ? `updateModelConsumption(${model.id}, '${key}', this.value); saveDataToFirestore();`
                        : `updateModelName(${model.id}, this.value); renderModelSelector(); saveDataToFirestore();`
                );

                modelCard.innerHTML = `
                    <div class="flex justify-between items-center mb-4">
                        <input type="text" id="name-${model.id}" value="${model.name}" oninput="${getUpdateFunction('name', false)}" class="text-xl font-semibold text-gray-800 p-1 border-b border-gray-300 w-3/4">
                        <button onclick="removeModel(${model.id})" class="remove-btn" title="Eliminar Modelo">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-red-500 hover:text-red-700" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.86 10.36A2 2 0 0116.14 19H7.86a2 2 0 01-1.99-1.64L5 7m5-3h4a2 2 0 012 2v1m-6 0h6m-3 0V4"></path>
                            </svg>
                        </button>
                    </div>

                    <!-- Selector de Tipo de LED -->
                    <div class="space-y-2">
                        <label class="block text-sm font-medium text-gray-700">Tipo de LED (Material):</label>
                        <select id="led-type-${model.id}" onchange="updateModelLedType(${model.id}, this.value); saveDataToFirestore();" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500">
                            <option value="white" ${currentLedType === 'white' ? 'selected' : ''}>Tira LED Blanca (120/m)</option>
                            <option value="rgb" ${currentLedType === 'rgb' ? 'selected' : ''}>Tira LED RGB (60/m)</option>
                        </select>
                    </div>

                    <label class="block pt-3">Filamento PLA (gramos):</label>
                    <input type="number" id="pla-${model.id}" value="${model.consumption.pla_grams}" oninput="${getUpdateFunction('pla_grams')}" class="w-full p-2 border border-gray-300 rounded-lg">
                    
                    <label class="block">Consumo de LED (cantidad de SECTORES de 3 LEDs):</label>
                    <input type="number" id="led-${model.id}" value="${model.consumption.led_quantity}" oninput="${getUpdateFunction('led_quantity')}" class="w-full p-2 border border-gray-300 rounded-lg">
                    <p class="text-xs text-gray-500 mt-1">Cada sector representa ${LEDS_PER_SEGMENT} LEDs. Ingresa el número de sectores de corte que lleva el modelo.</p>
                    
                    <label class="block pt-3 text-sm font-medium text-gray-700">Tiempos de Producción:</label>
                    
                    <label class="block text-sm text-gray-600">Mano de Obra Ensamblaje (minutos):</label>
                    <input type="number" id="mo-${model.id}" value="${model.consumption.mo_minutes}" oninput="${getUpdateFunction('mo_minutes')}" class="w-full p-2 border border-gray-300 rounded-lg">
                    
                    <!-- TIEMPO DE IMPRESIÓN (HORAS Y MINUTOS) -->
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="block text-sm text-gray-600">Horas de Impresión:</label>
                            <input type="number" id="print-h-${model.id}" value="${model.consumption.print_hours}" oninput="${getUpdateFunction('print_hours')}" class="w-full p-2 border border-gray-300 rounded-lg">
                        </div>
                        <div>
                            <label class="block text-sm text-gray-600">Minutos de Impresión:</label>
                            <input type="number" id="print-m-${model.id}" value="${model.consumption.print_minutes}" oninput="${getUpdateFunction('print_minutes')}" class="w-full p-2 border border-gray-300 rounded-lg">
                        </div>
                    </div>
                `;
                container.appendChild(modelCard);
            });
            renderModelSelector();
            displayResults();
        }

        /**
         * Renderiza las opciones en el selector de modelos.
         */
        function renderModelSelector() {
            const selector = document.getElementById('select-model');
            // Guardar el ID seleccionado previamente
            const currentSelectedId = parseFloat(selector?.value);
            
            selector.innerHTML = models.map(m => `<option value="${m.id}">${m.name}</option>`).join('');

            // Intentar re-seleccionar el modelo que estaba activo
            if (models.some(m => m.id === currentSelectedId)) {
                selector.value = currentSelectedId;
            } else if (models.length > 0) {
                selector.value = models[0].id;
            }
        }

        /**
         * Muestra el resultado del cálculo para el modelo seleccionado.
         */
        function displayResults() {
            const selector = document.getElementById('select-model');
            const selectedId = parseFloat(selector?.value);
            const model = models.find(m => m.id === selectedId);
            const costs = getBaseCosts();
            const resultDiv = document.getElementById('result-display');

            if (!model) {
                resultDiv.innerHTML = '<p class="text-center text-gray-500">Selecciona o agrega un modelo para calcular.</p>';
                return;
            }

            const calculation = calculateModelCost(model, costs);
            const format = formatCurrency;
            
            // Texto para mostrar el tipo de LED (vuelve a la versión sin nombre genérico)
            const ledTypeDesc = model.ledType === 'rgb' ? 'Tira RGB' : 'Tira Blanca';
            const totalLeds = model.consumption.led_quantity * LEDS_PER_SEGMENT;

            resultDiv.innerHTML = `
                <h3 class="text-2xl font-bold mb-6 text-blue-600 border-b pb-2">${model.name}</h3>
                
                <div class="space-y-4 mb-6">
                    <h4 class="text-xl font-semibold text-gray-700">Desglose de Costos</h4>
                    
                    <!-- Materiales Directos -->
                    <div class="p-3 bg-gray-50 rounded-lg">
                        <p class="font-bold mb-2 text-gray-700">1. Materiales Directos (${format(calculation.costMaterials)})</p>
                        <ul class="text-sm text-gray-600 ml-4 list-disc space-y-1">
                            <li>Filamento PLA (${model.consumption.pla_grams}g): ${format(calculation.costPla)}</li>
                            <li>Tira LED (${totalLeds} LEDs de ${ledTypeDesc}): ${format(calculation.costLeds)}</li>
                            <li>Fuente 12V (1 unidad): ${format(costs.power_supply)}</li>
                            <li>Embalaje/Caja (1 unidad): ${format(costs.packaging)}</li>
                        </ul>
                    </div>

                    <!-- Producción Variable -->
                    <div class="p-3 bg-gray-50 rounded-lg">
                        <p class="font-bold mb-2 text-gray-700">2. Producción Variable (${format(calculation.costProduction)})</p>
                        <ul class="text-sm text-gray-600 ml-4 list-disc space-y-1">
                            <li>Mano de Obra (${model.consumption.mo_minutes} min): ${format(calculation.costMOD)}</li>
                            <li>Energía de Impresión (${(model.consumption.print_hours * 60) + model.consumption.print_minutes} min): ${format(calculation.costEnergy)}</li>
                        </ul>
                    </div>

                    <!-- Costos Fijos -->
                    <div class="p-3 bg-gray-50 rounded-lg">
                        <p class="font-bold mb-2 text-gray-700">3. Costos Fijos Asignados (CIF): ${format(calculation.costCIF)}</p>
                        <p class="text-xs text-gray-500 ml-4">Costo fijo mensual prorrateado entre ${document.getElementById('input-cif-production').value} unidades.</p>
                    </div>
                </div>
                
                <div class="border-t border-gray-300 pt-4 space-y-3">
                    <p class="text-2xl font-bold flex justify-between">
                        <span>COSTO UNITARIO DE FABRICACIÓN (CUF):</span>
                        <span class="text-green-600">${format(calculation.CUF)}</span>
                    </p>
                    <p class="text-md flex justify-between">
                        <span>Precio de Venta Sugerido (Con ${costs.margin * 100}% Margen Neto):</span>
                        <span class="font-medium text-lg">${format(calculation.suggestedPrice)}</span>
                    </p>
                    <div class="p-3 bg-red-50 rounded-lg">
                        <p class="text-3xl font-extrabold flex justify-between">
                            <span>PRECIO FINAL RECOMENDADO:</span>
                            <span class="text-red-600">${format(calculation.finalPrice)}</span>
                        </p>
                        <p class="text-xs text-gray-600 text-right mt-1">Este precio incluye ${costs.commissions * 100}% de comisiones/impuestos (ej. Mercado Libre, IVA, etc.).</p>
                    </div>
                </div>
            `;
        }

        // --- MODEL MANIPULATION ---

        async function addModel() {
            const newModel = {
                id: Date.now(),
                name: "Nuevo Lightbox",
                ledType: 'white', // Default a blanca
                consumption: {
                    pla_grams: 50,
                    led_quantity: 12, // Ejemplo (12 sectores * 3 LEDs = 36 LEDs)
                    mo_minutes: 10,
                    print_hours: 2,   
                    print_minutes: 0,
                }
            };
            models.push(newModel);
            renderModelInputs();
            // Seleccionar el nuevo modelo para que se muestre inmediatamente
            document.getElementById('select-model').value = newModel.id;
            displayResults();
            if (db) await saveDataToFirestore();
        }

        async function removeModel(id) {
            if (models.length <= 1) {
                customAlert('Debe haber al menos un modelo para poder calcular los costos.', 'Alerta');
                return;
            }
            models = models.filter(m => m.id !== id);
            renderModelInputs();
            displayResults();
            if (db) await saveDataToFirestore();
        }

        async function updateModelConsumption(id, key, value) {
            const model = models.find(m => m.id === id);
            if (model) {
                // Validación básica para asegurar números positivos y limitar minutos
                const numValue = parseFloat(value) || 0;
                
                if (key === 'print_minutes') {
                    // Limitar minutos de impresión a 0-59
                    model.consumption[key] = Math.min(59, Math.max(0, numValue));
                } else {
                    model.consumption[key] = Math.max(0, numValue);
                }
            }
            displayResults();
            // Guardar en Firestore después de cualquier cambio de consumo
            if (db) await saveDataToFirestore(); 
        }

        function updateModelName(id, name) {
            const model = models.find(m => m.id === id);
            if (model) {
                model.name = name;
            }
            // El guardado se delega al oninput del renderModelInputs
        }

        /**
         * Actualiza el tipo de LED seleccionado para un modelo.
         * @param {number} id - ID del modelo.
         * @param {string} value - 'white' o 'rgb'.
         */
        async function updateModelLedType(id, value) {
            const model = models.find(m => m.id === id);
            if (model) {
                model.ledType = value;
            }
            displayResults();
            if (db) await saveDataToFirestore();
        }
        
        // --- UI UTILITIES ---

        /** * Llama al cálculo de costos base y al resultado final, y guarda en Firestore. 
         * Se usa en los eventos oninput de la pestaña 1 y 3.
         */
        async function updateBaseCostsDisplay() {
            calculateBaseUnitCosts();
            displayResults();
            if (db) await saveDataToFirestore();
        }

        function showSection(targetId) {
            document.querySelectorAll('.tab-content').forEach(div => {
                div.classList.add('hidden');
            });
            document.getElementById(targetId).classList.remove('hidden');

            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.classList.remove('border-blue-500', 'text-blue-600');
                btn.classList.add('border-transparent', 'text-gray-600');
            });

            document.querySelector(`.tab-btn[data-target="${targetId}"]`).classList.add('border-blue-500', 'text-blue-600');
            document.querySelector(`.tab-btn[data-target="${targetId}"]`).classList.remove('border-transparent', 'text-gray-600');
        }

        // Exponer funciones globales para el HTML
        window.saveDataToFirestore = saveDataToFirestore;
        window.updateBaseCostsDisplay = updateBaseCostsDisplay;
        window.addModel = addModel;
        window.removeModel = removeModel;
        window.updateModelConsumption = updateModelConsumption;
        window.updateModelName = updateModelName;
        window.updateModelLedType = updateModelLedType;
        window.displayResults = displayResults;
        window.showSection = showSection;
        window.closeModal = closeModal;
        window.loadDataByCustomKey = loadDataByCustomKey;
        
        // --- INICIALIZACIÓN ---
        window.onload = function() {
            initFirebase();
            showSection('results'); // Mostrar la pestaña de resultados por defecto al cargar
        };

    </script>
</body>
</html>
