<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora de Costos con Google Auth</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Librería para PDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f7fafc; }
        .card { box-shadow: 0 0 20px rgba(0, 0, 0, 0.05); transition: all 0.3s ease; }
        .step-indicator { 
            display: flex; 
            flex-direction: column; 
            align-items: center; 
            text-align: center;
            padding: 8px 4px;
        }
        .step-indicator-dot {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background-color: #d1d5db; /* gray-300 */
            transition: background-color 0.3s;
            margin-bottom: 4px;
        }
        .step-indicator.active .step-indicator-dot { 
            background-color: #3b82f6; /* blue-600 */
        }
        .step-indicator.active { color: #3b82f6; font-weight: 700; }
        
        .step-content { animation: fadeIn 0.4s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        
        .derived-value { font-weight: 700; color: #10b981; }
        
        /* Estilos para inputs de tipo number que elimina las flechas */
        input[type="number"]::-webkit-outer-spin-button,
        input[type="number"]::-webkit-inner-spin-button {
            -webkit-appearance: none;
            margin: 0;
        }
        input[type="number"] {
            -moz-appearance: textfield;
        }
        
        /* Tooltip styles */
        .tooltip-container {
            position: relative;
            display: inline-block;
        }
        
        .tooltip-container .tooltip-text {
            visibility: hidden;
            width: 250px;
            background-color: #1f2937;
            color: #fff;
            text-align: center;
            border-radius: 6px;
            padding: 8px;
            position: absolute;
            z-index: 1000;
            bottom: 125%;
            left: 50%;
            transform: translateX(-50%);
            opacity: 0;
            transition: opacity 0.3s;
            font-size: 12px;
            font-weight: normal;
            pointer-events: none;
        }
        
        .tooltip-container:hover .tooltip-text {
            visibility: visible;
            opacity: 1;
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="app" class="min-h-screen">
        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-extrabold text-blue-800">Calculadora de Costos de Producción</h1>
            <p class="text-lg text-gray-600">Costos Fijos Derivados (CF/U) y Tasas Variables</p>
            <div class="mt-2 flex justify-center items-center gap-4 flex-wrap">
                <!-- Estado de conexión -->
                <div class="flex items-center gap-2">
                    <div id="online-status" class="w-3 h-3 rounded-full bg-green-500 animate-pulse"></div>
                    <span id="online-text" class="text-xs text-gray-500">Conectando...</span>
                </div>
                
                <!-- Botón de autenticación -->
                <button id="auth-button" class="px-4 py-2 text-sm font-semibold rounded-lg shadow transition duration-150 flex items-center gap-2">
                    <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24"><path d="M12.24 10.29v-2.18c0-.66.07-1.15.2-1.55l-.01-.01c-.13-.39-.37-.87-.72-1.39l-2.09 1.43c.12.36.19.78.19 1.15v2.54c0 .41-.05.81-.13 1.15l2.09 1.43c.35-.52.59-1.0.72-1.39.13-.4.2-1.04.2-1.66zm6.39 3.02c.03-.2.05-.4.05-.61 0-1.12-.22-2.19-.65-3.19l-1.92 1.32c.15.34.23.72.23 1.13 0 .21-.02.42-.06.63h-3.83c0-1.08.35-2.04.93-2.84l-1.87-1.28c-.8.64-1.31 1.54-1.52 2.53h-2.3v-2.09h-.02l-2.07 1.41c-.44 1-.68 2.07-.68 3.19s.24 2.19.68 3.19l2.07 1.41h2.3c.21.99.72 1.89 1.52 2.53l1.87-1.28c-.58-.8-.93-1.76-.93-2.84h3.83c.04.21.06.42.06.63 0 .41-.08.8-.23 1.13l1.92 1.32c.43-1 .65-2.07.65-3.19z"></path><path d="M12 24c6.63 0 12-5.37 12-12S18.63 0 12 0 0 5.37 0 12s5.37 12 12 12zm0-22c5.52 0 10 4.48 10 10s-4.48 10-10 10S2 17.52 2 12 6.48 2 12 2z"></path></svg>
                    <span id="auth-status-text">Iniciar Sesión con Google</span>
                </button>
                <p id="user-info" class="text-xs text-gray-500">ID: Cargando...</p>
            </div>
        </header>

        <div class="max-w-5xl mx-auto card bg-white rounded-xl border border-gray-200 p-6 md:p-10">

            <!-- Navegación de Pasos -->
            <div class="flex justify-between items-start mb-8 border-b-2 pb-4 overflow-x-auto">
                <div id="step-1-btn" class="step-indicator active flex-1 cursor-pointer transition mx-1" onclick="window.app.showStep(1)">
                    <div class="step-indicator-dot"></div>
                    <span class="text-xs md:text-sm">1. Costos Base y Tasas</span>
                </div>
                <div id="step-2-btn" class="step-indicator flex-1 cursor-pointer transition text-gray-500 mx-1" onclick="window.app.showStep(2)">
                    <div class="step-indicator-dot"></div>
                    <span class="text-xs md:text-sm">2. Modelos y Consumo</span>
                </div>
                <div id="step-3-btn" class="step-indicator flex-1 cursor-pointer transition text-gray-500 mx-1" onclick="window.app.showStep(3)">
                    <div class="step-indicator-dot"></div>
                    <span class="text-xs md:text-sm">3. Resumen y PDF</span>
                </div>
            </div>

            <!-- PASO 1: COSTOS BASE, MATERIALES Y CÁLCULO DE TASAS -->
            <div id="step-1" class="step-content">
                <h2 class="text-2xl font-bold mb-6 text-gray-800">1. Definición de Costos Base</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    
                    <!-- Materiales Estándar (PLA, LED, Varios) -->
                    <div class="space-y-4 p-4 bg-blue-50 rounded-lg border border-blue-200 md:col-span-2">
                        <h3 class="font-bold text-lg text-blue-800">Materiales Directos Base y Fijos por Unidad</h3>
                        
                        <div class="grid grid-cols-2 md:grid-cols-5 gap-4">
                            <!-- PLA -->
                            <div class="input-group bg-white p-2 rounded shadow-sm col-span-2 md:col-span-1">
                                <label class="block text-xs font-bold text-gray-500 uppercase">Filamento PLA (Unit)</label>
                                <div><label class="text-xs">Costo Carrete ($)</label><input type="number" id="costo_pla_carrete" value="30000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                <div><label class="text-xs">Peso (g)</label><input type="number" id="peso_pla_carrete" value="1000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                <p class="text-xs mt-1 text-right">Costo/g: <span id="derived_costo_pla" class="derived-value">0.00</span></p>
                            </div>
    
                            <!-- LED -->
                            <div class="input-group bg-white p-2 rounded shadow-sm col-span-2 md:col-span-1">
                                <label class="block text-xs font-bold text-gray-500 uppercase">Paquete LEDs (Unit)</label>
                                <div><label class="text-xs">Costo Paquete ($)</label><input type="number" id="costo_paquete_led" value="2000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                <div><label class="text-xs">Unidades</label><input type="number" id="unidades_paquete_led" value="5000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                <p class="text-xs mt-1 text-right">Costo Unitario Base: <span id="derived_costo_led" class="derived-value">0.00</span></p>
                            </div>
                            
                            <!-- Otros Fijos por Unidad -->
                            <div><label class="block text-xs">Fuente 12V ($/u)</label><input type="number" id="costo_fuente" value="1200" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                            <div><label class="block text-xs">Embalaje ($/u)</label><input type="number" id="costo_embalaje" value="500" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                            <div><label class="block text-xs">Plug Hembra ($/u)</label><input type="number" id="costo_plug_hembra" value="80" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                        </div>
                    </div>

                    <!-- CÁLCULO DE COSTOS FIJOS Y DE PRODUCCIÓN (DERIVADOS) -->
                    <div class="space-y-4 p-4 bg-yellow-50 rounded-lg border border-yellow-200 md:col-span-2">
                        <h3 class="font-bold text-lg text-yellow-800">Costos Fijos y de Producción (Derivados)</h3>
                        
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            
                            <!-- Box 1: TASAS VARIABLES POR MINUTO -->
                            <div class="p-3 bg-white rounded shadow-sm md:col-span-2 grid grid-cols-2 gap-4">
                                <div><h4 class="font-bold text-sm text-gray-700 mb-2">Tasas Variables por Minuto</h4>
                                    <!-- CÁLCULO MOD -->
                                    <div class="mb-3">
                                        <div class="tooltip-container">
                                            <label class="text-xs cursor-help border-b border-dotted border-blue-300">Costo Laboral Mensual ($)</label>
                                            <span class="tooltip-text">Salario bruto total mensual de los operarios</span>
                                        </div>
                                        <input type="number" id="salario_bruto_mensual" value="450000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()">
                                        <div class="tooltip-container">
                                            <label class="text-xs cursor-help border-b border-dotted border-blue-300">Minutos Productivos Mensual</label>
                                            <span class="tooltip-text">Total de minutos trabajados realmente productivos por mes</span>
                                        </div>
                                        <input type="number" id="minutos_productivos_mensual" value="9600" class="w-full p-1 border rounded" oninput="window.app.calculateCost()">
                                        <p class="text-xs mt-1 font-bold text-right">MOD/min: <span id="derived_costo_mod_minuto" class="derived-value text-yellow-600">0.00</span></p>
                                        <input type="hidden" id="costo_mod_minuto">
                                    </div>
                                    <!-- CÁLCULO ENERGÍA -->
                                    <div>
                                        <div><label class="text-xs">Costo Total Luz Mes ($)</label><input type="number" id="costo_luz_mensual" value="15000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                        <div class="grid grid-cols-2 gap-2">
                                            <div><label class="text-xs">Consumo kWh Mes</label><input type="number" id="consumo_kwh_mensual" value="300" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                            <div><label class="text-xs">Potencia Impresora (W)</label><input type="number" id="potencia_impresora_w" value="150" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                        </div>
                                        <p class="text-xs mt-1 font-bold text-right">Energía/min: <span id="derived_costo_energia_minuto" class="derived-value text-yellow-600">0.00</span></p>
                                        <input type="hidden" id="costo_energia_minuto">
                                    </div>
                                </div>
                            </div>

                            <!-- Box 2: COSTO FIJO POR UNIDAD (CF/U) -->
                            <div class="p-3 bg-white rounded shadow-sm md:col-span-1">
                                <h4 class="font-bold text-sm text-gray-700 mb-2">Costo Fijo por Unidad (CF/U)</h4>
                                <div><label class="text-xs">Renta Mensual ($)</label><input type="number" id="renta_mensual" value="100000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                <div><label class="text-xs">Salarios Fijos ($)</label><input type="number" id="salarios_fijos_mensual" value="300000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                <div><label class="text-xs">Servicios Fijos ($)</label><input type="number" id="servicios_fijos_mensual" value="50000" class="w-full p-1 border rounded" oninput="window.app.calculateCost()"></div>
                                <div class="tooltip-container">
                                    <label class="text-xs font-bold text-blue-600 cursor-help border-b border-dotted border-blue-400">Volumen Mensual (u)</label>
                                    <span class="tooltip-text">CF/U = (Renta + Salarios Fijos + Servicios) / Volumen Mensual</span>
                                </div>
                                <input type="number" id="unidades_a_producir_mensual" value="500" class="w-full p-1 border rounded font-semibold" oninput="window.app.calculateCost()">

                                <p class="text-xs mt-2 font-bold text-right border-t pt-2">CF/U Derivado: <span id="derived_costo_fijo_unidad" class="derived-value text-blue-600">0.00</span></p>
                                <input type="hidden" id="costo_fijo_unidad">
                            </div>
                        </div>
                    </div>
                    
                    <!-- Materiales Personalizados -->
                    <div class="space-y-4 p-4 bg-orange-50 rounded-lg border border-orange-200">
                        <div class="flex justify-between items-center">
                            <h3 class="font-bold text-lg text-orange-800">Otros Materiales Directos</h3>
                            <button onclick="window.app.addCustomMaterial()" class="text-xs bg-orange-500 text-white px-2 py-1 rounded hover:bg-orange-600 transition">+ Agregar</button>
                        </div>
                        <p class="text-xs text-gray-600">Define insumos que se consumen por unidad (tornillos, resina, etc.).</p>
                        <div id="custom-materials-list" class="space-y-3">
                            <!-- Aquí se inyectan los materiales extra -->
                        </div>
                    </div>

                    <!-- Márgenes -->
                    <div class="space-y-4 p-4 bg-gray-50 rounded-lg border border-gray-200">
                        <h3 class="font-bold text-lg text-gray-700">Márgenes y Fijos (Adicionales)</h3>
                        <div class="grid grid-cols-2 gap-4">
                            <div><label class="block text-xs">Margen (%)</label><input type="number" id="margen_ganancia" value="35" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                            <div><label class="block text-xs">Comisiones (%)</label><input type="number" id="comisiones" value="15" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                            <div><label class="block text-xs">Fijo por Unidad (Ej: Publicidad)</label><input type="number" id="fijo_por_unidad" value="150" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                        </div>
                    </div>
                </div>
                <button onclick="window.app.showStep(2)" class="w-full mt-6 py-3 bg-blue-600 text-white font-bold rounded-lg hover:bg-blue-700 transition">Siguiente: Modelos</button>
            </div>

            <!-- PASO 2: MODELOS Y CONSUMOS -->
            <div id="step-2" class="step-content hidden">
                <h2 class="text-2xl font-bold mb-6 text-gray-800">2. Consumo por Modelo</h2>
                
                <div class="bg-white border border-gray-300 p-4 rounded-lg mb-6 shadow-sm">
                    <label class="block text-sm font-bold text-gray-700 mb-1">Seleccionar o Crear Modelo:</label>
                    <div class="flex gap-2">
                        <select id="modelo_selector" class="flex-1 p-2 border border-gray-300 rounded bg-gray-50" onchange="window.app.loadModelData()"></select>
                        <button onclick="window.app.deleteCurrentModel()" class="bg-red-100 text-red-600 px-3 rounded border border-red-200 hover:bg-red-200 transition">Eliminar</button>
                    </div>
                    <div class="mt-2 flex gap-2">
                        <input type="text" id="model_name_input" class="flex-1 p-2 border border-gray-300 rounded" placeholder="Nombre del modelo (ej: Corazon-RGB)">
                        <button onclick="window.app.saveCurrentModel()" class="bg-green-600 text-white px-4 rounded hover:bg-green-700 font-medium transition">Guardar Modelo</button>
                    </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Selector de Modo de Cálculo (El "Modo Doble") -->
                    <div class="space-y-3 p-4 bg-blue-100 rounded-lg border border-blue-300 md:col-span-2">
                        <h3 class="font-bold text-blue-800">Definición del Producto (Modo Doble)</h3>
                        <label class="block text-sm font-medium text-gray-700">Selecciona el tipo de cálculo:</label>
                        <select id="calculation_mode" class="w-full p-2 border rounded bg-white font-semibold" onchange="window.app.toggleModelInputs(); window.app.calculateCost();">
                            <option value="lightbox">Lightbox (Con LEDs, Fuente, Plug y Embalaje)</option>
                            <option value="general3d">Impresión 3D General (Solo filamento y MOD/Energía)</option>
                        </select>
                    </div>

                    <!-- Consumo Básico (Siempre visible) -->
                    <div class="space-y-3 p-4 bg-gray-50 rounded-lg border">
                        <h3 class="font-bold text-gray-700">Consumo de Material</h3>
                        <div><label class="text-sm">PLA (gramos)</label><input type="number" id="consumo_pla" value="80" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                        
                        <!-- Inputs Específicos de Lightbox (Ocultos inicialmente) -->
                        <div id="lightbox-specific-inputs" class="hidden"> 
                            <h4 class="font-semibold text-xs text-blue-600 pt-2 border-t border-blue-200">Componentes Lightbox</h4>
                            <div><label class="text-sm">Sectores LED</label><input type="number" id="sectores_led" value="20" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                            <div><label class="text-sm">Tipo LED</label>
                                <select id="tipo_led" class="w-full p-2 border rounded" onchange="window.app.calculateCost()">
                                    <option value="Blanca">Blanca</option>
                                    <option value="RGB">RGB</option>
                                </select>
                            </div>
                            <p class="text-xs text-gray-500 mt-2">NOTA: Los costos de Fuente, Plug Hembra y Embalaje (definidos en Paso 1) se incluyen automáticamente para este modo.</p>
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
                        <h3 class="font-bold text-gray-700">Tiempos de Producción (En minutos)</h3>
                        <div class="grid grid-cols-3 gap-4">
                            <div><label class="text-xs">MOD Ensamblaje (Min)</label><input type="number" id="mod_ensamblaje_min" value="15" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                            <div><label class="text-xs">Impresión (Horas)</label><input type="number" id="horas_impresion" value="3" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                            <div><label class="text-xs">Impresión (Min)</label><input type="number" id="minutos_impresion" value="0" class="w-full p-2 border rounded" oninput="window.app.calculateCost()"></div>
                        </div>
                        <p class="text-xs text-gray-500 mt-2">NOTA: Estos tiempos se usan para asignar los costos variables de MOD y Energía definidos en el Paso 1.</p>
                    </div>
                </div>

                <div class="flex justify-between mt-6">
                    <button onclick="window.app.showStep(1)" class="px-6 py-2 bg-gray-400 text-white rounded hover:bg-gray-500 transition">Atrás</button>
                    <button onclick="window.app.showStep(3)" class="px-6 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 font-bold transition">Ver Resultados</button>
                </div>
            </div>

            <!-- PASO 3: RESUMEN Y PDF -->
            <div id="step-3" class="step-content hidden">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-2xl font-bold text-gray-800">3. Resumen de Costos</h2>
                    <button onclick="window.app.generatePDF()" class="bg-red-600 text-white px-4 py-2 rounded shadow hover:bg-red-700 flex items-center gap-2 transition">
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
                            <p class="text-sm text-gray-600">Costo Unitario de Fabricación (CUF)</p>
                            <p id="cuf" class="text-2xl font-bold text-green-800">$0.00</p>
                        </div>
                        <div class="p-4 bg-blue-50 rounded-lg border border-blue-200 tooltip-container">
                            <p class="text-sm text-gray-600">Precio de Venta Sugerido</p>
                            <p id="precio_final" class="text-2xl font-bold text-blue-800">$0.00</p>
                            <span class="tooltip-text">Precio = CUF / (1 - %Margen - %Comisiones)</span>
                        </div>
                    </div>

                    <h3 class="font-bold text-gray-800 border-b mb-2 pb-1">Desglose de Materiales Directos</h3>
                    <table class="w-full text-sm text-left mb-6">
                        <tbody class="divide-y divide-gray-100">
                            <tr><td class="py-1">PLA (Gramos)</td><td class="py-1 text-right" id="costo_pla_calc">$0.00</td></tr>
                            <tr id="row-leds"><td class="py-1">LEDs</td><td class="py-1 text-right" id="costo_led_calc">$0.00</td></tr>
                            <tr id="row-fuente"><td class="py-1">Fuente 12V</td><td class="py-1 text-right" id="costo_fuente_calc">$0.00</td></tr>
                            <tr id="row-plug"><td class="py-1">Plug Hembra</td><td class="py-1 text-right" id="costo_plug_calc">$0.00</td></tr>
                            <tr id="row-embalaje"><td class="py-1">Embalaje</td><td class="py-1 text-right" id="costo_embalaje_calc">$0.00</td></tr>
                            <tr id="extra-materials-summary-start" class="hidden"></tr> 
                        </tbody>
                        <tfoot class="font-bold border-t">
                            <tr><td class="py-2">Subtotal Materiales</td><td class="py-2 text-right" id="subtotal_materiales">$0.00</td></tr>
                        </tfoot>
                    </table>

                    <h3 class="font-bold text-gray-800 border-b mb-2 pb-1">Costos de Conversión y Fijos</h3>
                    <table class="w-full text-sm text-left mb-6">
                        <tbody class="divide-y divide-gray-100">
                            <tr><td class="py-1">Mano de Obra (MOD - Ensamblaje)</td><td class="py-1 text-right" id="costo_mod_calc">$0.00</td></tr>
                            <tr><td class="py-1">Energía (Impresión)</td><td class="py-1 text-right" id="costo_energia_calc">$0.00</td></tr>
                            <tr class="bg-blue-50 font-semibold text-blue-800">
                                <td class="py-1">Costo Fijo por Unidad (CF/U)</td>
                                <td class="py-1 text-right" id="costo_fijo_unidad_calc">$0.00</td>
                            </tr>
                            <tr><td class="py-1">Fijo por Unidad (Ej: Publicidad)</td><td class="py-1 text-right" id="costo_fijo_calc">$0.00</td></tr>
                        </tbody>
                        <tfoot class="font-bold border-t">
                            <tr><td class="py-2">Subtotal Conversión y Fijos</td><td class="py-2 text-right" id="subtotal_produccion">$0.00</td></tr>
                        </tfoot>
                    </table>
                    
                    <div class="text-xs text-gray-400 text-center mt-8">Generado por la Calculadora de Costos</div>
                </div>

                <button onclick="window.app.showStep(2)" class="w-full mt-6 py-3 bg-gray-400 text-white rounded font-bold hover:bg-gray-500 transition">Volver a Editar</button>
            </div>
            
            <div id="status-message" class="mt-4 text-center text-sm hidden"></div>
        </div>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        // Importaciones de Firebase
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, GoogleAuthProvider, signInWithPopup, signOut } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        setLogLevel('debug');

        // --- Configuración de Credenciales de Firebase ---
        // Tu configuración original (NO MODIFICADA)
        const YOUR_FIREBASE_CONFIG = {
            apiKey: "AIzaSyA1Yy13XMF2oB7KkFJVPOKvp8z01YiuNF0", 
            authDomain: "calculador-de-costos-c4f71.firebaseapp.com",
            projectId: "calculador-de-costos-c4f71",
            storageBucket: "calculador-de-costos-c4f71.firebasestorage.app",
            messagingSenderId: "880952513286",
            appId: "1:880952513286:web:3a6798f1ead2527977c487"
        };

        // Lógica para obtener credenciales
        let firebaseConfig = YOUR_FIREBASE_CONFIG;
        let appId = 'github-page-calculator';
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        if (typeof __firebase_config !== 'undefined' && typeof __app_id !== 'undefined') {
            try {
                firebaseConfig = JSON.parse(__firebase_config);
                appId = __app_id;
                console.log("INFO: Usando configuración de Firebase de Canvas.");
            } catch (e) {
                console.warn("ADVERTENCIA: Fallo al parsear __firebase_config, usando configuración de respaldo.");
            }
        } else {
            console.log("INFO: Usando configuración de Firebase de respaldo.");
        }
        
        // --- Constantes y Variables Globales ---
        const STORAGE_KEY = 'cost_calculator_cache_v2';
        const DATA_PATH = (uid) => `artifacts/${appId}/users/${uid}/calculator_v5/data`;
        
        let db, auth, userId = null;
        let isOnline = navigator.onLine;
        let saveTimeout = null;
        
        const googleProvider = new GoogleAuthProvider();
        
        // --- Objeto Principal de la Aplicación ---
        window.app = {
            data: {
                baseCosts: {},
                customMaterials: [],
                models: {}
            },
            
            // Estado de la aplicación
            status: {
                isOnline: isOnline,
                isLoading: false,
                lastSave: null
            },
