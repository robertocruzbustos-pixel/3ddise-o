# 3ddise-o
Calculadora de costos Lightbox
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora Lightbox y 3D General (Doble Modo)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Librería para PDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f7fafc; }
        .card { box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); transition: all 0.3s ease; }
        .step-indicator.active { background-color: #3b82f6; color: white; border-color: #3b82f6; }
        .step-content { animation: fadeIn 0.4s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .bg-result { background: linear-gradient(135deg, #ecfdf5 0%, #d1fae5 100%); }
        .derived-value { font-weight: 700; color: #10b981; }
        /* Estilos específicos para el PDF */
        .pdf-content { background: white; padding: 20px; }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="app" class="min-h-screen">
        <header class="text-center mb-8">
            <h1 class="text-4xl font-extrabold text-blue-800">Calculadora de Costos (Doble Modo)</h1>
            <p class="text-lg text-gray-600">Lightbox y Modelos 3D Generales</p>
            <p id="user-info" class="text-xs text-gray-400 mt-1">Usuario ID: Cargando...</p>
        </header>

        <div class="max-w-5xl mx-auto card bg-white rounded-xl border border-gray-200 p-6 md:p-10">

            <!-- Navegación de Pasos -->
            <div class="flex justify-between items-center mb-8 border-b-2 pb-4 overflow-x-auto">
                <div id="step-1-btn" class="step-indicator active flex-1 text-center py-2 px-1 rounded-lg border-2 border-transparent cursor-pointer font-semibold text-sm transition mx-1" onclick="showStep(1)">
                    1. Costos y Materiales
                </div>
                <div id="step-2-btn" class="step-indicator flex-1 text-center py-2 px-1 rounded-lg border-2 border-gray-300 cursor-pointer font-semibold text-sm transition text-gray-500 mx-1" onclick="showStep(2)">
                    2. Modelos y Consumo
                </div>
                <div id="step-3-btn" class="step-1-btn flex-1 text-center py-2 px-1 rounded-lg border-2 border-gray-300 cursor-pointer font-semibold text-sm transition text-gray-500 mx-1" onclick="showStep(3)">
                    3. Resumen y PDF
                </div>
            </div>

            <!-- PASO 1: COSTOS BASE Y MATERIALES DINÁMICOS -->
            <div id="step-1" class="step-content">
                <h2 class="text-2xl font-bold mb-6 text-gray-800">1. Definición de Costos Base</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    
                    <!-- Materiales Estándar (PLA, LED) -->
                    <div class="space-y-4 p-4 bg-blue-50 rounded-lg border border-blue-200">
                        <h3 class="font-bold text-lg text-blue-800">Materiales Estándar (Derivación)</h3>
                        
                        <!-- PLA -->
                        <div class="input-group bg-white p-3 rounded shadow-sm">
                            <label class="block text-xs font-bold text-gray-500 uppercase">Filamento PLA</label>
                            <div class="grid grid-cols-2 gap-2 mt-1">
                                <div><label class="text-xs">Costo Carrete ($)</label><input type="number" id="costo_pla_carrete" value="30000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <div><label class="text-xs">Peso (g)</label><input type="number" id="peso_pla_carrete" value="1000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                            </div>
                            <p class="text-xs mt-1 text-right">Costo/g: <span id="derived_costo_pla" class="derived-value">0.00</span></p>
                        </div>

                        <!-- LED -->
                        <div class="input-group bg-white p-3 rounded shadow-sm">
                            <label class="block text-xs font-bold text-gray-500 uppercase">Paquete LEDs (Base)</label>
                            <div class="grid grid-cols-2 gap-2 mt-1">
                                <div><label class="text-xs">Costo Paquete ($)</label><input type="number" id="costo_paquete_led" value="2000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <div><label class="text-xs">Unidades</label><input type="number" id="unidades_paquete_led" value="5000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                            </div>
                            <p class="text-xs mt-1 text-right">Costo Unitario Base: <span id="derived_costo_led" class="derived-value">0.00</span></p>
                        </div>
                    </div>

                    <!-- Materiales Personalizados -->
                    <div class="space-y-4 p-4 bg-orange-50 rounded-lg border border-orange-200">
                        <div class="flex justify-between items-center">
                            <h3 class="font-bold text-lg text-orange-800">Otros Materiales Directos</h3>
                            <button onclick="addCustomMaterial()" class="text-xs bg-orange-500 text-white px-2 py-1 rounded hover:bg-orange-600 transition">+ Agregar</button>
                        </div>
                        <p class="text-xs text-gray-600">Define insumos que se consumen por unidad (tornillos, resina, etc.).</p>
                        
                        <!-- Contenedor de inputs dinámicos -->
                        <div id="custom-materials-list" class="space-y-3">
                            <!-- Aquí se inyectan los materiales extra -->
                        </div>
                    </div>

                    <!-- Fijos y Producción -->
                    <div class="space-y-4 p-4 bg-gray-50 rounded-lg border border-gray-200 md:col-span-2">
                        <h3 class="font-bold text-lg text-gray-700">Costos Fijos, Producción y Márgenes</h3>
                        <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                            <div><label class="block text-xs">Fuente 12V ($/u)</label><input type="number" id="costo_fuente" value="1200" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Embalaje ($/u)</label><input type="number" id="costo_embalaje" value="500" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Mano Obra ($/min)</label><input type="number" id="costo_mod_minuto" value="41.67" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Energía ($/min)</label><input type="number" id="costo_energia_minuto" value="0.005" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">CIF ($/u)</label><input type="number" id="cif_por_unidad" value="150" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Margen (%)</label><input type="number" id="margen_ganancia" value="35" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Comisiones (%)</label><input type="number" id="comisiones" value="15" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                        </div>
                    </div>
                </div>
                <button onclick="showStep(2)" class="w-full mt-6 py-3 bg-blue-600 text-white font-bold rounded-lg hover:bg-blue-700 transition">Siguiente: Modelos</button>
            </div>

            <!-- PASO 2: MODELOS Y CONSUMOS -->
            <div id="step-2" class="step-content hidden">
                <h2 class="text-2xl font-bold mb-6 text-gray-800">2. Consumo por Modelo</h2>
                
                <div class="bg-white border border-gray-300 p-4 rounded-lg mb-6 shadow-sm">
                    <label class="block text-sm font-bold text-gray-700 mb-1">Seleccionar o Crear Modelo:</label>
                    <div class="flex gap-2">
                        <select id="modelo_selector" class="flex-1 p-2 border border-gray-300 rounded bg-gray-50" onchange="loadModelData()"></select>
                        <button onclick="deleteCurrentModel()" class="bg-red-100 text-red-600 px-3 rounded border border-red-200 hover:bg-red-200 transition">Eliminar</button>
                    </div>
                    <div class="mt-2 flex gap-2">
                        <input type="text" id="model_name_input" class="flex-1 p-2 border border-gray-300 rounded" placeholder="Nombre del modelo (ej: Corazon-RGB)">
                        <button onclick="saveCurrentModel()" class="bg-green-600 text-white px-4 rounded hover:bg-green-700 font-medium transition">Guardar Modelo</button>
                    </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Selector de Modo de Cálculo -->
                    <div class="space-y-3 p-4 bg-blue-100 rounded-lg border border-blue-300 md:col-span-2">
                        <h3 class="font-bold text-blue-800">Definición del Producto</h3>
                        <label class="block text-sm font-medium text-gray-700">Selecciona el tipo de cálculo:</label>
                        <select id="calculation_mode" class="w-full p-2 border rounded bg-white font-semibold" onchange="toggleModelInputs(); calculateCost();">
                            <option value="lightbox">Lightbox (Con LEDs y Fuente)</option>
                            <option value="general3d">Impresión 3D General (Mate, Llavero, Figura, etc.)</option>
                        </select>
                    </div>

                    <!-- Consumo Básico (Siempre visible) -->
                    <div class="space-y-3 p-4 bg-gray-50 rounded-lg border">
                        <h3 class="font-bold text-gray-700">Consumo y Material Básico</h3>
                        <div><label class="text-sm">PLA (gramos)</label><input type="number" id="consumo_pla" value="80" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                        
                        <!-- Inputs Específicos de Lightbox (Se ocultan para General 3D) -->
                        <div id="lightbox-specific-inputs">
                            <h4 class="font-semibold text-xs text-blue-600 pt-2 border-t border-blue-200">Componentes Lightbox</h4>
                            <div><label class="text-sm">Sectores LED</label><input type="number" id="sectores_led" value="20" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="text-sm">Tipo LED</label>
                                <select id="tipo_led" class="w-full p-2 border rounded" onchange="calculateCost()">
                                    <option value="Blanca">Blanca</option>
                                    <option value="RGB">RGB</option>
                                </select>
                            </div>
                        </div>
                    </div>

                    <!-- Consumo Materiales Extra (Siempre visible) -->
                    <div class="space-y-3 p-4 bg-orange-50 rounded-lg border border-orange-200">
                        <h3 class="font-bold text-orange-800">Consumo Materiales Extra</h3>
                        <p class="text-xs text-gray-500">Define cuánto usa este modelo de tus materiales personalizados (tornillos, pegamento, etc.).</p>
                        <!-- Contenedor dinámico para consumo de extras -->
                        <div id="custom-consumption-list" class="space-y-2"></div>
                    </div>

                    <!-- Tiempos de Producción (Siempre visible) -->
                    <div class="space-y-3 p-4 bg-gray-50 rounded-lg border md:col-span-2">
                        <h3 class="font-bold text-gray-700">Tiempos de Producción</h3>
                        <div class="grid grid-cols-3 gap-4">
                            <div><label class="text-xs">MOD Ensamblaje (Min)</label><input type="number" id="mod_ensamblaje_min" value="15" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="text-xs">Impresión (Horas)</label><input type="number" id="horas_impresion" value="3" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="text-xs">Impresión (Min)</label><input type="number" id="minutos_impresion" value="0" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                        </div>
                    </div>
                </div>

                <div class="flex justify-between mt-6">
                    <button onclick="showStep(1)" class="px-6 py-2 bg-gray-400 text-white rounded hover:bg-gray-500 transition">Atrás</button>
                    <button onclick="showStep(3)" class="px-6 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 font-bold transition">Ver Resultados</button>
                </div>
            </div>

            <!-- PASO 3: RESUMEN Y PDF -->
            <div id="step-3" class="step-content hidden">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-2xl font-bold text-gray-800">3. Resumen de Costos</h2>
                    <button onclick="generatePDF()" class="bg-red-600 text-white px-4 py-2 rounded shadow hover:bg-red-700 flex items-center gap-2 transition">
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
                        Descargar PDF
                    </button>
                </div>

                <!-- Elemento a imprimir en PDF -->
                <div id="pdf-printable-area" class="pdf-content border rounded-xl shadow-sm">
                    
                    <div class="border-b pb-4 mb-4">
                        <h1 class="text-3xl font-extrabold text-gray-900">Presupuesto de Costos</h1>
                        <p class="text-gray-500">
                            Modelo: <span id="summary_model_name" class="text-blue-600 font-bold">---</span> | 
                            Tipo: <span id="summary_model_type" class="font-semibold text-purple-600">---</span>
                        </p>
                        <p class="text-xs text-gray-400 mt-1">Fecha: <span id="current-date"></span></p>
                    </div>

                    <div class="grid grid-cols-2 gap-6 mb-6">
                        <div class="p-4 bg-green-50 rounded-lg border border-green-200">
                            <p class="text-sm text-gray-600">Costo Unitario (CUF)</p>
                            <p id="cuf" class="text-2xl font-bold text-green-800">$0.00</p>
                        </div>
                        <div class="p-4 bg-blue-50 rounded-lg border border-blue-200">
                            <p class="text-sm text-gray-600">Precio de Venta Sugerido</p>
                            <p id="precio_final" class="text-2xl font-bold text-blue-800">$0.00</p>
                        </div>
                    </div>

                    <h3 class="font-bold text-gray-800 border-b mb-2 pb-1">Desglose de Materiales Directos</h3>
                    <table class="w-full text-sm text-left mb-6">
                        <tbody class="divide-y divide-gray-100">
                            <tr><td class="py-1">PLA</td><td class="py-1 text-right" id="costo_pla_calc">$0.00</td></tr>
                            <!-- Fila de LEDs y Fuente se muestran condicionalmente -->
                            <tr id="row-leds"><td class="py-1">LEDs</td><td class="py-1 text-right" id="costo_led_calc">$0.00</td></tr>
                            <tr id="row-fuente"><td class="py-1">Fuente 12V</td><td class="py-1 text-right" id="costo_fuente_calc">$0.00</td></tr>
                            <tr id="row-embalaje"><td class="py-1">Embalaje</td><td class="py-1 text-right" id="costo_embalaje_calc">$0.00</td></tr>
                            <!-- Materiales extra se insertan aquí por JS -->
                            <tr id="extra-materials-summary-start" class="hidden"></tr> 
                        </tbody>
                        <tfoot class="font-bold border-t">
                            <tr><td class="py-2">Subtotal Materiales</td><td class="py-2 text-right" id="subtotal_materiales">$0.00</td></tr>
                        </tfoot>
                    </table>

                    <h3 class="font-bold text-gray-800 border-b mb-2 pb-1">Producción y Fijos</h3>
                    <table class="w-full text-sm text-left mb-6">
                        <tbody class="divide-y divide-gray-100">
                            <tr><td class="py-1">Mano de Obra</td><td class="py-1 text-right" id="costo_mod_calc">$0.00</td></tr>
                            <tr><td class="py-1">Energía (Impresión)</td><td class="py-1 text-right" id="costo_energia_calc">$0.00</td></tr>
                            <tr><td class="py-1">CIF Asignado</td><td class="py-1 text-right" id="subtotal_cif">$0.00</td></tr>
                        </tbody>
                        <tfoot class="font-bold border-t">
                            <tr><td class="py-2">Subtotal Producción y Fijos</td><td class="py-2 text-right" id="subtotal_produccion">$0.00</td></tr>
                        </tfoot>
                    </table>
                    
                    <div class="text-xs text-gray-400 text-center mt-8">Generado por Lightbox Cost Calculator</div>
                </div>

                <button onclick="showStep(2)" class="w-full mt-6 py-3 bg-gray-400 text-white rounded font-bold hover:bg-gray-500 transition">Volver a Editar</button>
            </div>
            
            <div id="status-message" class="mt-4 text-center text-sm hidden"></div>
        </div>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Configuración de Firebase (Se asume en entorno Canvas o producción)
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let db, auth, userId = null;
        
        // Estructura de Datos Global
        window.appData = {
            baseCosts: {},      // Inputs simples del paso 1
            customMaterials: [], // Array de objetos {id, name, purchaseCost, purchaseQty}
            models: {}          // Modelos guardados { 'modelo-1': { ...consumos..., calculation_mode: 'lightbox' } }
        };

        const STATUS_MSG = document.getElementById('status-message');

        function displayStatus(msg, isError=false) {
            STATUS_MSG.textContent = msg;
            STATUS_MSG.className = `mt-4 p-2 text-center text-sm rounded ${isError ? 'bg-red-100 text-red-700' : 'bg-green-100 text-green-700'}`;
            STATUS_MSG.style.display = 'block';
            setTimeout(() => STATUS_MSG.style.display = 'none', 3000);
        }

        // --- Init Firebase ---
        async function initFirebase() {
            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                if (initialAuthToken) await signInWithCustomToken(auth, initialAuthToken);
                else await signInAnonymously(auth);

                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        document.getElementById('user-info').textContent = `ID: ${userId.substring(0,6)}...`;
                        loadData();
                    }
                });
            } catch (e) {
                console.error("Firebase Error:", e);
                // Si falla Firebase, inicializamos con defaults para que la app funcione offline/demo
                window.renderCustomMaterials();
                window.renderModelSelector();
            }
        }

        const DATA_PATH = (uid) => `artifacts/${appId}/users/${uid}/calculator_v4/data`;

        async function loadData() {
            if (!userId) return;
            try {
                const snap = await getDoc(doc(db, DATA_PATH(userId)));
                if (snap.exists()) {
                    const data = snap.data();
                    window.appData = data;
                    
                    // Restaurar Inputs Simples (Paso 1)
                    Object.keys(data.baseCosts || {}).forEach(id => {
                        const el = document.getElementById(id);
                        if(el) el.value = data.baseCosts[id];
                    });
                } 
                // Renderizar interfaces dinámicas
                window.renderCustomMaterials();
                window.renderModelSelector();
                window.toggleModelInputs();
                window.calculateCost(); // Calcular inicial
            } catch (e) { console.error(e); }
        }

        window.saveData = async function() {
            if (!userId) return;
            
            // 1. Capturar inputs base
            document.querySelectorAll('#step-1 input[type="number"]:not([data-custom])').forEach(el => {
                window.appData.baseCosts[el.id] = parseFloat(el.value) || 0;
            });

            try {
                await setDoc(doc(db, DATA_PATH(userId)), window.appData, { merge: true });
                displayStatus("Guardado en la nube");
            } catch (e) { displayStatus("Error al guardar", true); }
        }

        initFirebase();
    </script>

    <script>
        // --- Lógica de la Aplicación ---
        
        const FORMAT = { style: 'currency', currency: 'ARS' };
        
        // 1. Gestión de Materiales Personalizados
        window.addCustomMaterial = function() {
            const id = 'mat_' + Date.now();
            window.appData.customMaterials.push({ id: id, name: 'Nuevo Material', cost: 1000, qty: 10 });
            window.renderCustomMaterials();
            window.saveData();
        }

        window.removeCustomMaterial = function(id) {
            window.appData.customMaterials = window.appData.customMaterials.filter(m => m.id !== id);
            window.renderCustomMaterials();
            window.saveData();
        }

        window.updateCustomMaterial = function(id, field, value) {
            const mat = window.appData.customMaterials.find(m => m.id === id);
            if (mat) {
                mat[field] = field === 'name' ? value : (parseFloat(value) || 0);
                window.renderCustomMaterials(); // Re-render para actualizar unitario
                window.saveData();
                window.calculateCost(); // Recalcular costos
            }
        }

        window.renderCustomMaterials = function() {
            const container = document.getElementById('custom-materials-list');
            container.innerHTML = '';
            
            window.appData.customMaterials.forEach(mat => {
                const unitCost = mat.qty > 0 ? mat.cost / mat.qty : 0;
                
                const div = document.createElement('div');
                div.className = "bg-white p-2 rounded border border-orange-100 text-xs";
                div.innerHTML = `
                    <div class="flex gap-2 mb-1">
                        <input type="text" value="${mat.name}" class="flex-1 border-b border-orange-200 focus:outline-none" onchange="updateCustomMaterial('${mat.id}', 'name', this.value)">
                        <button onclick="removeCustomMaterial('${mat.id}')" class="text-red-400 font-bold">×</button>
                    </div>
                    <div class="grid grid-cols-3 gap-2 items-center">
                        <div><span class="text-gray-400 block" style="font-size:10px">Compra $</span><input type="number" value="${mat.cost}" class="w-full border rounded p-1" oninput="updateCustomMaterial('${mat.id}', 'cost', this.value)"></div>
                        <div><span class="text-gray-400 block" style="font-size:10px">Cant.</span><input type="number" value="${mat.qty}" class="w-full border rounded p-1" oninput="updateCustomMaterial('${mat.id}', 'qty', this.value)"></div>
                        <div class="text-right"><span class="text-gray-400 block" style="font-size:10px">Unitario</span><span class="font-bold text-orange-600">$${unitCost.toFixed(2)}</span></div>
                    </div>
                `;
                container.appendChild(div);
            });
            
            // También actualizar los inputs de consumo en el Paso 2
            window.renderCustomConsumptionInputs();
        }

        // 2. Control de visibilidad de inputs específicos (NUEVO)
        window.toggleModelInputs = function() {
            const mode = document.getElementById('calculation_mode').value;
            const lightboxInputs = document.getElementById('lightbox-specific-inputs');
            
            if (mode === 'lightbox') {
                lightboxInputs.classList.remove('hidden');
            } else {
                lightboxInputs.classList.add('hidden');
            }
        }

        // 3. Gestión de Consumos Dinámicos (Paso 2)
        window.renderCustomConsumptionInputs = function() {
            const container = document.getElementById('custom-consumption-list');
            const currentModelKey = document.getElementById('modelo_selector').value;
            const currentModel = window.appData.models[currentModelKey] || {};
            const customConsumptions = currentModel.customConsumptions || {};

            container.innerHTML = '';
            
            if (window.appData.customMaterials.length === 0) {
                container.innerHTML = '<p class="text-gray-400 italic text-center">No hay materiales extra definidos en el Paso 1.</p>';
                return;
            }

            window.appData.customMaterials.forEach(mat => {
                const val = customConsumptions[mat.id] || 0;
                // Usamos el ID de input específico para poder recogerlo al guardar/cargar
                const div = document.createElement('div');
                div.innerHTML = `
                    <label class="text-sm flex justify-between">
                        <span>${mat.name}</span>
                        <span class="text-xs text-gray-400">(Unit: $${(mat.qty > 0 ? mat.cost/mat.qty : 0).toFixed(2)})</span>
                    </label>
                    <input type="number" id="cons_${mat.id}" value="${val}" class="w-full p-2 border rounded custom-cons-input" data-mat-id="${mat.id}" oninput="window.calculateCost()">
                `;
                container.appendChild(div);
            });
        }

        // 4. Gestión de Modelos
        window.renderModelSelector = function() {
            const selector = document.getElementById('modelo_selector');
            const currentVal = selector.value;
            selector.innerHTML = '';
            
            const keys = Object.keys(window.appData.models);
            if (keys.length === 0) {
                selector.innerHTML = '<option value="">Crear nuevo modelo...</option>';
            } else {
                keys.forEach(key => {
                    const opt = document.createElement('option');
                    opt.value = key;
                    opt.textContent = key.replace(/-/g, ' ').toUpperCase();
                    selector.appendChild(opt);
                });
                if (keys.includes(currentVal)) selector.value = currentVal;
            }
        }

        window.loadModelData = function() {
            const key = document.getElementById('modelo_selector').value;
            const model = window.appData.models[key];
            if (!model) { 
                // Si se selecciona "Crear nuevo modelo...", limpiamos los campos y ponemos defaults
                document.getElementById('model_name_input').value = '';
                document.getElementById('consumo_pla').value = 0;
                document.getElementById('sectores_led').value = 0;
                document.getElementById('mod_ensamblaje_min').value = 0;
                document.getElementById('horas_impresion').value = 0;
                document.getElementById('minutos_impresion').value = 0;
                document.getElementById('calculation_mode').value = 'lightbox';
                
                window.renderCustomConsumptionInputs(); // Limpia los campos de consumo custom
                window.toggleModelInputs();
                window.calculateCost();
                return;
            }

            // Cargar datos estándar
            ['consumo_pla', 'sectores_led', 'tipo_led', 'mod_ensamblaje_min', 'horas_impresion', 'minutos_impresion', 'calculation_mode'].forEach(k => {
                const el = document.getElementById(k);
                if(el) el.value = model[k];
            });

            // Cargar nombre
            document.getElementById('model_name_input').value = key;

            // Recargar inputs dinámicos con los valores de este modelo
            window.renderCustomConsumptionInputs();
            window.toggleModelInputs();
            window.calculateCost();
        }

        window.saveCurrentModel = function() {
            let name = document.getElementById('model_name_input').value.trim();
            if (!name) name = document.getElementById('modelo_selector').value;
            if (!name) { alert("Por favor, ponle un nombre al modelo"); return; }

            const key = name.toLowerCase().replace(/\s+/g, '-');
            
            // Recoger consumos custom
            const customCons = {};
            document.querySelectorAll('.custom-cons-input').forEach(input => {
                customCons[input.dataset.matId] = parseFloat(input.value) || 0;
            });

            const newModel = {
                consumo_pla: parseFloat(document.getElementById('consumo_pla').value) || 0,
                sectores_led: parseFloat(document.getElementById('sectores_led').value) || 0,
                tipo_led: document.getElementById('tipo_led').value,
                mod_ensamblaje_min: parseFloat(document.getElementById('mod_ensamblaje_min').value) || 0,
                horas_impresion: parseFloat(document.getElementById('horas_impresion').value) || 0,
                minutos_impresion: parseFloat(document.getElementById('minutos_impresion').value) || 0,
                calculation_mode: document.getElementById('calculation_mode').value, // NUEVO CAMPO
                customConsumptions: customCons
            };

            window.appData.models[key] = newModel;
            window.renderModelSelector();
            document.getElementById('modelo_selector').value = key;
            window.saveData();
            alert("Modelo Guardado");
        }
        
        window.deleteCurrentModel = function() {
            const key = document.getElementById('modelo_selector').value;
            if(confirm("¿Eliminar " + key + "?")) {
                delete window.appData.models[key];
                window.renderModelSelector();
                window.saveData();
            }
        }

        // 5. Cálculo Principal
        window.showStep = function(step) {
            [1,2,3].forEach(s => {
                document.getElementById(`step-${s}`).classList.add('hidden');
                document.getElementById(`step-${s}-btn`).classList.remove('active', 'border-blue-500', 'text-blue-600');
            });
            document.getElementById(`step-${step}`).classList.remove('hidden');
            document.getElementById(`step-${step}-btn`).classList.add('active');
            
            if (step === 2) window.toggleModelInputs();
            window.calculateCost();
        }

        window.calculateCost = function() {
            // A. Derivaciones Paso 1
            const getVal = (id) => parseFloat(document.getElementById(id).value) || 0;
            
            const costPlaReel = getVal('costo_pla_carrete');
            const weightPla = getVal('peso_pla_carrete');
            const unitPla = weightPla > 0 ? costPlaReel / weightPla : 0;
            document.getElementById('derived_costo_pla').textContent = unitPla.toFixed(2);

            const costLedPack = getVal('costo_paquete_led');
            const unitLedPack = getVal('unidades_paquete_led');
            const unitLedBase = unitLedPack > 0 ? costLedPack / unitLedPack : 0;
            document.getElementById('derived_costo_led').textContent = unitLedBase.toFixed(4);

            const unitLedWhite = unitLedBase;
            const unitLedRGB = unitLedBase * 1.5; // RGB más caro (factor ejemplo)

            // B. Cálculo de Modelo
            const mode = document.getElementById('calculation_mode').value;
            const isLightbox = mode === 'lightbox';
            
            const consPla = getVal('consumo_pla');
            const costTotalPla = consPla * unitPla;

            let costTotalLed = 0;
            let cFuente = 0;
            let cEmb = getVal('costo_embalaje');

            if (isLightbox) {
                // Costos específicos de Lightbox
                const sectorsLed = getVal('sectores_led');
                const typeLed = document.getElementById('tipo_led').value;
                costTotalLed = sectorsLed * 3 * (typeLed === 'RGB' ? unitLedRGB : unitLedWhite); // 3 LEDs por sector
                cFuente = getVal('costo_fuente');
            }
            // NOTA: cEmb y CIF_por_unidad se aplican a ambos, ya que son Costo/Unidad

            // C. Materiales Extra (Iterar sobre array dinámico)
            let costTotalExtras = 0;
            let extrasHTML = '';
            
            // Recoger valores actuales de los inputs de consumo custom
            document.querySelectorAll('.custom-cons-input').forEach(input => {
                const matId = input.dataset.matId;
                const qtyUsed = parseFloat(input.value) || 0;
                
                // Buscar datos base del material
                const matBase = window.appData.customMaterials.find(m => m.id === matId);
                if (matBase && qtyUsed > 0) {
                    const unitCost = matBase.qty > 0 ? matBase.cost / matBase.qty : 0;
                    const totalThisMat = qtyUsed * unitCost;
                    costTotalExtras += totalThisMat;
                    
                    // Añadir fila al PDF
                    extrasHTML += `<tr><td class="py-1">${matBase.name} (${qtyUsed} u)</td><td class="py-1 text-right">$${totalThisMat.toFixed(2)}</td></tr>`;
                }
            });

            // Inyectar extras en la tabla del PDF/Resumen
            const extrasRow = document.getElementById('extra-materials-summary-start');
            if (extrasRow) {
                document.querySelectorAll('.extra-row-pdf').forEach(e => e.remove());
                if (extrasHTML) {
                    extrasRow.insertAdjacentHTML('afterend', extrasHTML.replace(/<tr>/g, '<tr class="extra-row-pdf">'));
                }
            }

            const subMateriales = costTotalPla + costTotalLed + cFuente + cEmb + costTotalExtras;

            // D. Producción
            const modMin = getVal('mod_ensamblaje_min');
            const costMod = modMin * getVal('costo_mod_minuto');
            
            const hImp = getVal('horas_impresion');
            const mImp = getVal('minutos_impresion');
            const totalMinImp = (hImp * 60) + mImp;
            const costEnergia = totalMinImp * getVal('costo_energia_minuto');
            
            const subProduccion = costMod + costEnergia;
            const subCif = getVal('cif_por_unidad');

            // Totales
            const CUF = subMateriales + subProduccion + subCif;
            const margen = getVal('margen_ganancia') / 100;
            const comision = getVal('comisiones') / 100;
            
            const precioSugerido = CUF * (1 + margen);
            const precioFinal = (1 - comision) !== 0 ? precioSugerido / (1 - comision) : 0;

            // Render DOM Resumen
            document.getElementById('cuf').textContent = `$${CUF.toLocaleString('es-AR', {minimumFractionDigits: 2})}`;
            document.getElementById('precio_final').textContent = `$${precioFinal.toLocaleString('es-AR', {minimumFractionDigits: 2})}`;
            
            // Renderización condicional en el resumen
            document.getElementById('row-leds').style.display = isLightbox ? '' : 'none';
            document.getElementById('row-fuente').style.display = isLightbox ? '' : 'none';

            document.getElementById('costo_pla_calc').textContent = `$${costTotalPla.toFixed(2)}`;
            document.getElementById('costo_led_calc').textContent = `$${costTotalLed.toFixed(2)}`;
            document.getElementById('costo_fuente_calc').textContent = `$${cFuente.toFixed(2)}`;
            document.getElementById('costo_embalaje_calc').textContent = `$${cEmb.toFixed(2)}`;
            document.getElementById('subtotal_materiales').textContent = `$${subMateriales.toFixed(2)}`;
            
            document.getElementById('costo_mod_calc').textContent = `$${costMod.toFixed(2)}`;
            document.getElementById('costo_energia_calc').textContent = `$${costEnergia.toFixed(2)}`;
            document.getElementById('subtotal_produccion').textContent = `$${(subProduccion + subCif).toFixed(2)}`; // Subtotal incluye CIF
            document.getElementById('subtotal_cif').textContent = `$${subCif.toFixed(2)}`;

            // Actualizar header del resumen
            const selector = document.getElementById('modelo_selector');
            const modelName = document.getElementById('model_name_input').value || selector.options[selector.selectedIndex]?.text || '---';
            document.getElementById('summary_model_name').textContent = modelName;
            document.getElementById('summary_model_type').textContent = isLightbox ? 'Lightbox' : 'Impresión 3D General';
            document.getElementById('current-date').textContent = new Date().toLocaleDateString();
            
            // Guardar Costos Base automáticamente
            window.saveData(); 
        }

        window.generatePDF = function() {
            const element = document.getElementById('pdf-printable-area');
            const modelName = document.getElementById('summary_model_name').textContent;
            
            const opt = {
                margin:       0.5,
                filename:     `Presupuesto_${modelName.replace(/\s/g, '_')}.pdf`,
                image:        { type: 'jpeg', quality: 0.98 },
                html2canvas:  { scale: 2 },
                jsPDF:        { unit: 'in', format: 'letter', orientation: 'portrait' }
            };

            // Usar html2pdf
            html2pdf().set(opt).from(element).save();
        }

        // Event Listeners globales para recálculo
        document.addEventListener('input', (e) => {
            if (e.target.type === 'number' && e.target.id.startsWith('cons_')) window.calculateCost(); // Solo custom inputs
        });
        
        // Ejecutar inicialización al cargar
        document.addEventListener('DOMContentLoaded', () => {
            window.toggleModelInputs(); // Asegura el estado inicial
        });

    </script>
</body>
</html>
