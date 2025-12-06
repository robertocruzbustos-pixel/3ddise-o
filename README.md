# 3ddise-o
Calculadora de costos Lightbox
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
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="app" class="min-h-screen">
        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-extrabold text-blue-800">Calculadora de Costos de Producción</h1>
            <p class="text-lg text-gray-600">Costos Fijos Derivados (CF/U) y Tasas Variables</p>
            <div class="mt-2 flex justify-center items-center gap-4">
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
                <div id="step-1-btn" class="step-indicator active flex-1 cursor-pointer transition mx-1" onclick="showStep(1)">
                    <div class="step-indicator-dot"></div>
                    <span class="text-xs md:text-sm">1. Costos Base y Tasas</span>
                </div>
                <div id="step-2-btn" class="step-indicator flex-1 cursor-pointer transition text-gray-500 mx-1" onclick="showStep(2)">
                    <div class="step-indicator-dot"></div>
                    <span class="text-xs md:text-sm">2. Modelos y Consumo</span>
                </div>
                <div id="step-3-btn" class="step-indicator flex-1 cursor-pointer transition text-gray-500 mx-1" onclick="showStep(3)">
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
                                <div><label class="text-xs">Costo Carrete ($)</label><input type="number" id="costo_pla_carrete" value="30000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <div><label class="text-xs">Peso (g)</label><input type="number" id="peso_pla_carrete" value="1000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <p class="text-xs mt-1 text-right">Costo/g: <span id="derived_costo_pla" class="derived-value">0.00</span></p>
                            </div>
    
                            <!-- LED -->
                            <div class="input-group bg-white p-2 rounded shadow-sm col-span-2 md:col-span-1">
                                <label class="block text-xs font-bold text-gray-500 uppercase">Paquete LEDs (Unit)</label>
                                <div><label class="text-xs">Costo Paquete ($)</label><input type="number" id="costo_paquete_led" value="2000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <div><label class="text-xs">Unidades</label><input type="number" id="unidades_paquete_led" value="5000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <p class="text-xs mt-1 text-right">Costo Unitario Base: <span id="derived_costo_led" class="derived-value">0.00</span></p>
                            </div>
                            
                            <!-- Otros Fijos por Unidad -->
                            <div><label class="block text-xs">Fuente 12V ($/u)</label><input type="number" id="costo_fuente" value="1200" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Embalaje ($/u)</label><input type="number" id="costo_embalaje" value="500" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Plug Hembra ($/u)</label><input type="number" id="costo_plug_hembra" value="80" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
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
                                        <div><label class="text-xs">Costo Laboral Mensual ($)</label><input type="number" id="salario_bruto_mensual" value="450000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                        <div><label class="text-xs">Minutos Productivos Mensual</label><input type="number" id="minutos_productivos_mensual" value="9600" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                        <p class="text-xs mt-1 font-bold text-right">MOD/min: <span id="derived_costo_mod_minuto" class="derived-value text-yellow-600">0.00</span></p>
                                        <input type="hidden" id="costo_mod_minuto">
                                    </div>
                                    <!-- CÁLCULO ENERGÍA -->
                                    <div>
                                        <div><label class="text-xs">Costo Total Luz Mes ($)</label><input type="number" id="costo_luz_mensual" value="15000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                        <div class="grid grid-cols-2 gap-2">
                                            <div><label class="text-xs">Consumo kWh Mes</label><input type="number" id="consumo_kwh_mensual" value="300" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                            <div><label class="text-xs">Potencia Impresora (W)</label><input type="number" id="potencia_impresora_w" value="150" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                        </div>
                                        <p class="text-xs mt-1 font-bold text-right">Energía/min: <span id="derived_costo_energia_minuto" class="derived-value text-yellow-600">0.00</span></p>
                                        <input type="hidden" id="costo_energia_minuto">
                                    </div>
                                </div>
                            </div>

                            <!-- Box 2: COSTO FIJO POR UNIDAD (CF/U) -->
                            <div class="p-3 bg-white rounded shadow-sm md:col-span-1">
                                <h4 class="font-bold text-sm text-gray-700 mb-2">Costo Fijo por Unidad (CF/U)</h4>
                                <div><label class="text-xs">Renta Mensual ($)</label><input type="number" id="renta_mensual" value="100000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <div><label class="text-xs">Salarios Fijos ($)</label><input type="number" id="salarios_fijos_mensual" value="300000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <div><label class="text-xs">Servicios Fijos ($)</label><input type="number" id="servicios_fijos_mensual" value="50000" class="w-full p-1 border rounded" oninput="calculateCost()"></div>
                                <div><label class="text-xs font-bold text-blue-600">Volumen Mensual (u)</label><input type="number" id="unidades_a_producir_mensual" value="500" class="w-full p-1 border rounded font-semibold" oninput="calculateCost()"></div>

                                <p class="text-xs mt-2 font-bold text-right border-t pt-2">CF/U Derivado: <span id="derived_costo_fijo_unidad" class="derived-value text-blue-600">0.00</span></p>
                                <input type="hidden" id="costo_fijo_unidad"> <!-- Campo oculto para el cálculo principal -->
                            </div>
                        </div>
                    </div>
                    
                    <!-- Materiales Personalizados -->
                    <div class="space-y-4 p-4 bg-orange-50 rounded-lg border border-orange-200">
                        <div class="flex justify-between items-center">
                            <h3 class="font-bold text-lg text-orange-800">Otros Materiales Directos</h3>
                            <button onclick="addCustomMaterial()" class="text-xs bg-orange-500 text-white px-2 py-1 rounded hover:bg-orange-600 transition">+ Agregar</button>
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
                            <div><label class="block text-xs">Margen (%)</label><input type="number" id="margen_ganancia" value="35" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Comisiones (%)</label><input type="number" id="comisiones" value="15" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="block text-xs">Fijo por Unidad (Ej: Publicidad)</label><input type="number" id="fijo_por_unidad" value="150" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
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
                    <!-- Selector de Modo de Cálculo (El "Modo Doble") -->
                    <div class="space-y-3 p-4 bg-blue-100 rounded-lg border border-blue-300 md:col-span-2">
                        <h3 class="font-bold text-blue-800">Definición del Producto (Modo Doble)</h3>
                        <label class="block text-sm font-medium text-gray-700">Selecciona el tipo de cálculo:</label>
                        <select id="calculation_mode" class="w-full p-2 border rounded bg-white font-semibold" onchange="toggleModelInputs(); calculateCost();">
                            <option value="lightbox">Lightbox (Con LEDs, Fuente, Plug y Embalaje)</option>
                            <option value="general3d">Impresión 3D General (Solo filamento y MOD/Energía)</option>
                        </select>
                    </div>

                    <!-- Consumo Básico (Siempre visible) -->
                    <div class="space-y-3 p-4 bg-gray-50 rounded-lg border">
                        <h3 class="font-bold text-gray-700">Consumo de Material</h3>
                        <div><label class="text-sm">PLA (gramos)</label><input type="number" id="consumo_pla" value="80" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                        
                        <!-- Inputs Específicos de Lightbox (Ocultos inicialmente) -->
                        <div id="lightbox-specific-inputs" class="hidden"> 
                            <h4 class="font-semibold text-xs text-blue-600 pt-2 border-t border-blue-200">Componentes Lightbox</h4>
                            <div><label class="text-sm">Sectores LED</label><input type="number" id="sectores_led" value="20" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="text-sm">Tipo LED</label>
                                <select id="tipo_led" class="w-full p-2 border rounded" onchange="calculateCost()">
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
                            <div><label class="text-xs">MOD Ensamblaje (Min)</label><input type="number" id="mod_ensamblaje_min" value="15" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="text-xs">Impresión (Horas)</label><input type="number" id="horas_impresion" value="3" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                            <div><label class="text-xs">Impresión (Min)</label><input type="number" id="minutos_impresion" value="0" class="w-full p-2 border rounded" oninput="calculateCost()"></div>
                        </div>
                        <p class="text-xs text-gray-500 mt-2">NOTA: Estos tiempos se usan para asignar los costos variables de MOD y Energía definidos en el Paso 1.</p>
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
                            <p class="text-sm text-gray-600">Costo Unitario de Fabricación (CUF)</p>
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

                <button onclick="showStep(2)" class="w-full mt-6 py-3 bg-gray-400 text-white rounded font-bold hover:bg-gray-500 transition">Volver a Editar</button>
            </div>
            
            <div id="status-message" class="mt-4 text-center text-sm hidden"></div>
        </div>
    </div>

    <script>
        // Definición de función showStep global (para onclick en HTML)
        window.showStep = function(step) {
            [1,2,3].forEach(s => {
                document.getElementById(`step-${s}`).classList.add('hidden');
                document.getElementById(`step-${s}-btn`).classList.remove('active');
            });
            document.getElementById(`step-${step}`).classList.remove('hidden');
            document.getElementById(`step-${step}-btn`).classList.add('active');
            
            if (step === 2) window.toggleModelInputs();
            if (step === 3) {
                 // Actualizar la fecha actual
                document.getElementById('current-date').textContent = new Date().toLocaleDateString('es-ES', { year: 'numeric', month: 'long', day: 'numeric' });
            }
            window.calculateCost(); 
        }

        // Definición de función generatePDF global
        window.generatePDF = function() {
            const element = document.getElementById('pdf-printable-area');
            html2pdf(element, {
                margin: 10,
                filename: `Presupuesto_${document.getElementById('summary_model_name').textContent.replace(/\s/g, '_')}_${new Date().toLocaleDateString('en-CA')}.pdf`,
                image: { type: 'jpeg', quality: 0.98 },
                html2canvas: { scale: 2, logging: true, dpi: 192, letterRendering: true },
                jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
            });
        }
    </script>

    <!-- Firebase SDKs -->
    <script type="module">
        // Importaciones de Firebase
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        // VOLVEMOS a signInWithPopup para una mejor UX
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, GoogleAuthProvider, signInWithPopup, signOut } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        setLogLevel('Debug');

        // --- Configuración de Credenciales de Firebase ---
        
        // 1. CONFIGURACIÓN DE RESPALDO (FALLBACK) PARA TU SITIO WEB (IMPORTANTE)
        // ** Reemplaza los valores de 'YOUR_...' con tus credenciales reales de Firebase **
        const YOUR_FIREBASE_CONFIG = {
            apiKey: "AIzaSyA1Yy13XMF2oB7KkFJVPOKvp8z01YiuNF0", 
            authDomain: "calculador-de-costos-c4f71.firebaseapp.com",
            projectId: "calculador-de-costos-c4f71",
            storageBucket: "calculador-de-costos-c4f71.firebasestorage.app",
            messagingSenderId: "880952513286",
            appId: "1:880952513286:web:3a6798f1ead2527977c487"
        };

        // 2. Lógica para obtener las credenciales (Prioriza Canvas, luego Fallback)
        let firebaseConfig = YOUR_FIREBASE_CONFIG;
        let appId = 'github-page-calculator'; // ID de app genérico para fuera de Canvas
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        // Comprobar si las variables de Canvas están presentes
        if (typeof __firebase_config !== 'undefined' && typeof __app_id !== 'undefined') {
            try {
                firebaseConfig = JSON.parse(__firebase_config);
                appId = __app_id;
                console.log("INFO: Usando configuración de Firebase de Canvas.");
            } catch (e) {
                console.warn("ADVERTENCIA: Fallo al parsear __firebase_config, usando configuración de respaldo.");
            }
        } else {
            console.log("INFO: Usando configuración de Firebase de respaldo (YOUR_FIREBASE_CONFIG).");
        }
        
        // --- Variables y Servicios Globales ---

        let db, auth, userId = null;
        const googleProvider = new GoogleAuthProvider(); // Instanciamos el proveedor una vez
        
        // Elementos UI para autenticación
        const authButton = document.getElementById('auth-button');
        const authStatusText = document.getElementById('auth-status-text');
        const userInfo = document.getElementById('user-info');

        // Estructura de Datos Global para persistencia
        window.appData = {
            baseCosts: {},      // Inputs simples y derivados del paso 1
            customMaterials: [], // Array de objetos {id, name, cost, qty}
            models: {}          // Modelos guardados { 'modelo-1': { ...consumos..., calculation_mode: 'lightbox' } }
        };

        const STATUS_MSG = document.getElementById('status-message');

        function displayStatus(msg, isError=false) {
            console.log(isError ? `ERROR: ${msg}` : `INFO: ${msg}`);
            STATUS_MSG.textContent = msg;
            STATUS_MSG.className = `mt-4 p-2 text-center text-sm rounded ${isError ? 'bg-red-100 text-red-700' : 'bg-green-100 text-green-700'}`;
            STATUS_MSG.style.display = 'block';
            setTimeout(() => STATUS_MSG.style.display = 'none', 5000); // Mostramos el mensaje un poco más de tiempo
        }

        // --- Manejadores de Autenticación de Google ---

        // Función con signInWithPopup (ventana emergente)
        async function signInGoogle() {
            if (!auth) {
                displayStatus("Error: El servicio de autenticación no está listo.", true);
                return;
            }
            try {
                // Iniciar la sesión con POPUP (requiere dominio autorizado)
                const result = await signInWithPopup(auth, googleProvider); 
                displayStatus(`¡Iniciado sesión como: ${result.user.email}!`);
            } catch (error) {
                console.error("Error al iniciar sesión con Google:", error);
                
                let friendlyMessage = "Error de conexión. Inténtalo de nuevo.";
                if (error.code === 'auth/unauthorized-domain') {
                    friendlyMessage = "ERROR: Dominio NO autorizado (auth/unauthorized-domain). Asegúrate de haber añadido 'robertocruzbustos-pixel.github.io' a Firebase Auth.";
                } else if (error.code === 'auth/popup-closed-by-user') {
                     friendlyMessage = "Ventana de inicio de sesión cerrada por el usuario.";
                } else if (error.code === 'auth/network-request-failed') {
                    friendlyMessage = "ERROR: Fallo de red/conexión. Revisa la consola y tu configuración de API Key.";
                } else {
                    friendlyMessage = `Error de Google Auth: ${error.code}. Revisa la consola.`;
                }

                displayStatus(friendlyMessage, true);
            }
        }

        async function signOutGoogle() {
            try {
                await signOut(auth);
                displayStatus("Sesión cerrada. Volviendo a modo anónimo.");
                // onAuthStateChanged se dispara al cerrar sesión
            } catch (error) {
                console.error("Error al cerrar sesión:", error);
                displayStatus("Error al cerrar sesión.", true);
            }
        }

        function updateAuthUI(user) {
            if (user && user.isAnonymous === false) {
                // Usuario autenticado con Google
                authButton.onclick = signOutGoogle;
                authStatusText.textContent = "Cerrar Sesión";
                authButton.classList.remove('bg-green-600', 'hover:bg-green-700');
                authButton.classList.add('bg-red-600', 'hover:bg-red-700');
                userInfo.textContent = `Proveedor: ${user.email}`;
            } else {
                // Usuario anónimo o deslogueado
                authButton.onclick = signInGoogle;
                authStatusText.textContent = "Iniciar Sesión con Google (Popup)";
                authButton.classList.remove('bg-red-600', 'hover:bg-red-700');
                authButton.classList.add('bg-green-600', 'hover:bg-green-700');
                userInfo.textContent = user && user.isAnonymous ? `ID Anónimo: ${user.uid.substring(0,6)}...` : 'ID: Desconectado';
            }
        }


        // --- Inicialización de Firebase ---
        async function initFirebase() {
            // Verifica que la configuración NO esté vacía
            if (Object.keys(firebaseConfig).length === 0 || !firebaseConfig.apiKey || firebaseConfig.apiKey.includes('YOUR_')) {
                displayStatus("ERROR CRÍTICO: Debes reemplazar 'YOUR_...' con tus credenciales reales de Firebase en el código fuente.", true);
                userInfo.textContent = 'ERROR de Configuración.';
                return;
            }

            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                // 1. Autenticación inicial (Anónima o Custom Token)
                let authAttempt;
                if (initialAuthToken) {
                    authAttempt = signInWithCustomToken(auth, initialAuthToken).catch(e => {
                        console.warn("Fallo en signInWithCustomToken, intentando signInAnonymously:", e);
                        return signInAnonymously(auth);
                    });
                } else {
                    authAttempt = signInAnonymously(auth);
                }
                
                await authAttempt; // Esperamos a que se complete el intento de autenticación inicial

                // 2. Listener de estado de autenticación (clave para cargar datos)
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        updateAuthUI(user);
                        loadData(); // Cargar datos del usuario
                    } else {
                        // Desconectado o error
                        userId = null;
                        updateAuthUI(null);
                        // Si no hay usuario, cargamos los defaults
                        if (!window.appData.models || !Object.keys(window.appData.models).length) {
                             window.renderCustomMaterials();
                             window.renderModelSelector();
                             window.calculateCost(); 
                        }
                    }
                });

            } catch (e) {
                console.error("Error de Firebase (Inicialización crítica):", e);
                displayStatus(`Error CRÍTICO al conectar con Firebase: ${e.message}`, true);
                userInfo.textContent = 'Error de conexión a Firebase.';
            }
        }

        // Ruta de los datos del usuario en Firestore (Datos Privados)
        const DATA_PATH = (uid) => `artifacts/${appId}/users/${uid}/calculator_v5/data`;

        async function loadData() {
            if (!userId) return;
            try {
                const snap = await getDoc(doc(db, DATA_PATH(userId)));
                if (snap.exists()) {
                    const data = snap.data();
                    window.appData = data;
                    
                    // Restaurar Inputs Simples (Paso 1)
                    document.querySelectorAll('#step-1 input[type="number"], #step-1 input[type="hidden"], #step-1 select').forEach(el => {
                        const savedValue = data.baseCosts[el.id];
                        if(el && savedValue !== undefined) el.value = savedValue;
                    });

                    if (!window.appData.customMaterials) window.appData.customMaterials = [];
                    if (!window.appData.models) window.appData.models = {};
                } else {
                    // displayStatus("No hay datos guardados. Usando valores predeterminados.");
                }
                
                // Aseguramos que el DOM se renderice con los datos cargados
                window.renderCustomMaterials();
                window.renderModelSelector();
                window.toggleModelInputs(); 
                window.calculateCost(); 
            } catch (e) { 
                console.error("Error al cargar datos:", e);
                displayStatus("Error al cargar datos del usuario.", true);
            }
        }

        window.saveData = async function() {
            // Guardamos incluso si es anónimo para persistencia temporal en la sesión (depende del token)
            if (!userId || !db) return; 
            
            // 1. Capturar inputs base 
            window.appData.baseCosts = {};
            document.querySelectorAll('#step-1 input[type="number"], #step-1 input[type="hidden"], #step-1 select').forEach(el => {
                window.appData.baseCosts[el.id] = parseFloat(el.value) || el.value; // Guardar valor de select
            });

            try {
                // Usamos el ID del usuario actual (Google o Anónimo)
                await setDoc(doc(db, DATA_PATH(userId)), window.appData, { merge: true });
                // displayStatus("Guardado en la nube"); // Comentado para evitar spam en cada input
            } catch (e) { 
                // displayStatus("Error al guardar, revisa la consola para ver si es un problema de permisos.", true); 
                console.error("Error al guardar datos en Firestore:", e); 
            }
        }

        initFirebase();
    </script>

    <script>
        // --- Lógica de la Aplicación (No necesita módulos) ---
        
        // Función auxiliar para obtener valores numéricos
        const getVal = (id) => parseFloat(document.getElementById(id).value) || 0;


        // 1. Gestión de Materiales Personalizados
        window.addCustomMaterial = function() {
            const id = 'mat_' + Date.now();
            // Valores predeterminados para el nuevo material
            window.appData.customMaterials.push({ id: id, name: 'Tornillos M3', cost: 100, qty: 100 });
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

        // 2. Control de visibilidad de inputs específicos (MODO DOBLE)
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
                container.innerHTML = '<p class="text-gray-400 italic text-center text-xs">No hay materiales extra definidos en el Paso 1.</p>';
                return;
            }

            window.appData.customMaterials.forEach(mat => {
                // Si el modelo no tiene el consumo guardado, usa 0
                const val = customConsumptions[mat.id] !== undefined ? customConsumptions[mat.id] : 0; 
                
                const unitCost = mat.qty > 0 ? mat.cost / mat.qty : 0;
                const div = document.createElement('div');
                div.innerHTML = `
                    <label class="text-sm flex justify-between">
                        <span>${mat.name}</span>
                        <span class="text-xs text-gray-400">(Unit: $${unitCost.toFixed(2)})</span>
                    </label>
                    <input type="number" id="cons_${mat.id}" value="${val}" class="w-full p-2 border rounded custom-cons-input" data-mat-id="${mat.id}" oninput="window.calculateCost()">
                `;
                container.appendChild(div);
            });
        }

        // 4. Gestión de Modelos (Carga, Guardado y Eliminación)
        window.renderModelSelector = function() {
            const selector = document.getElementById('modelo_selector');
            const currentVal = selector.value;
            selector.innerHTML = '';
            
            const keys = Object.keys(window.appData.models);
            if (keys.length === 0) {
                const opt = document.createElement('option');
                opt.value = '';
                opt.textContent = 'Crear nuevo modelo...';
                selector.appendChild(opt);
            } else {
                keys.forEach(key => {
                    const opt = document.createElement('option');
                    opt.value = key;
                    opt.textContent = key.replace(/-/g, ' ').toUpperCase();
                    selector.appendChild(opt);
                });
                // Seleccionar el modelo actual o el primero
                if (keys.includes(currentVal)) selector.value = currentVal;
                else selector.value = keys[0] || ''; 
            }
            window.loadModelData(); 
        }

        window.loadModelData = function() {
            const key = document.getElementById('modelo_selector').value;
            const model = window.appData.models[key];
            
            const defaultMode = 'lightbox';
            
            // Cargar defaults si no hay modelo o es nuevo
            if (!model) { 
                document.getElementById('model_name_input').value = '';
                document.getElementById('consumo_pla').value = 80;
                document.getElementById('sectores_led').value = 20;
                document.getElementById('mod_ensamblaje_min').value = 15;
                document.getElementById('horas_impresion').value = 3;
                document.getElementById('minutos_impresion').value = 0;
                document.getElementById('calculation_mode').value = defaultMode;
                document.getElementById('tipo_led').value = 'Blanca';
            } else {
                // Cargar datos del modelo
                ['consumo_pla', 'sectores_led', 'mod_ensamblaje_min', 'horas_impresion', 'minutos_impresion', 'calculation_mode'].forEach(k => {
                    const el = document.getElementById(k);
                    if(el) el.value = model[k] !== undefined ? model[k] : el.value;
                });
                
                // Cargar selectores
                document.getElementById('tipo_led').value = model.tipo_led || 'Blanca';
                document.getElementById('model_name_input').value = key;
            }

            window.renderCustomConsumptionInputs();
            window.toggleModelInputs();
            window.calculateCost();
        }

        window.saveCurrentModel = function() {
            let name = document.getElementById('model_name_input').value.trim();
            const currentKey = document.getElementById('modelo_selector').value;

            if (!name && currentKey) name = currentKey; 
            if (!name) { 
                displayStatus("Por favor, ponle un nombre al modelo.", true);
                return; 
            }

            const key = name.toLowerCase().replace(/\s+/g, '-');
            
            // Recoger consumos custom
            const customCons = {};
            document.querySelectorAll('.custom-cons-input').forEach(input => {
                customCons[input.dataset.matId] = parseFloat(input.value) || 0;
            });

            // Recoger datos del modelo
            const newModel = {
                consumo_pla: parseFloat(document.getElementById('consumo_pla').value) || 0,
                sectores_led: parseFloat(document.getElementById('sectores_led').value) || 0,
                tipo_led: document.getElementById('tipo_led').value,
                mod_ensamblaje_min: parseFloat(document.getElementById('mod_ensamblaje_min').value) || 0,
                horas_impresion: parseFloat(document.getElementById('horas_impresion').value) || 0,
                minutos_impresion: parseFloat(document.getElementById('minutos_impresion').value) || 0,
                calculation_mode: document.getElementById('calculation_mode').value, 
                customConsumptions: customCons
            };

            // Si el nombre cambió, eliminamos el modelo anterior
            if (currentKey && currentKey !== key) {
                delete window.appData.models[currentKey];
            }

            window.appData.models[key] = newModel;
            window.renderModelSelector();
            document.getElementById('modelo_selector').value = key;
            window.saveData();
            displayStatus(`Modelo '${name}' guardado exitosamente.`);
        }
        
        window.deleteCurrentModel = function() {
            const key = document.getElementById('modelo_selector').value;
            if (!key) {
                displayStatus("No hay un modelo seleccionado para eliminar.", true);
                return;
            }
            
            const name = key.replace(/-/g, ' ').toUpperCase();
            // Utilizamos un simple window.confirm como fallback
            if (window.confirm(`¿Estás seguro de eliminar el modelo: ${name}?`)) {
                delete window.appData.models[key];
                // Forzamos la carga del siguiente modelo o defaults
                window.renderModelSelector();
                window.saveData();
                displayStatus("Modelo eliminado.");
            }
        }
        


        // 5. Cálculo Principal: Derivación de Tasas y Asignación de Costos
        window.calculateCost = function() {
            
            // --- A. CÁLCULO DE TASAS UNITARIAS (PASO 1) ---

            // 1. Tasa PLA / LED
            const unitPla = getVal('peso_pla_carrete') > 0 ? getVal('costo_pla_carrete') / getVal('peso_pla_carrete') : 0;
            document.getElementById('derived_costo_pla').textContent = unitPla.toFixed(2);
            
            const unitLedBase = getVal('unidades_paquete_led') > 0 ? getVal('costo_paquete_led') / getVal('unidades_paquete_led') : 0;
            document.getElementById('derived_costo_led').textContent = unitLedBase.toFixed(4);

            const unitLedWhite = unitLedBase;
            const unitLedRGB = unitLedBase * 1.5; // Factor de costo RGB (asumido)

            // 2. Tasa Mano de Obra por Minuto (MOM)
            const salarioBrutoMensual = getVal('salario_bruto_mensual');
            const minutosProductivosMensual = getVal('minutos_productivos_mensual');
            const costoModMinuto = minutosProductivosMensual > 0 ? salarioBrutoMensual / minutosProductivosMensual : 0;
            document.getElementById('derived_costo_mod_minuto').textContent = costoModMinuto.toFixed(2);
            document.getElementById('costo_mod_minuto').value = costoModMinuto; 

            // 3. Tasa Costo de Energía por Minuto (CEM)
            const costoLuzMensual = getVal('costo_luz_mensual');
            const consumoKwhMensual = getVal('consumo_kwh_mensual');
            const potenciaImpresoraW = getVal('potencia_impresora_w');
            
            const costoKwh = consumoKwhMensual > 0 ? costoLuzMensual / consumoKwhMensual : 0;
            const kwhPorMinutoImpresora = potenciaImpresoraW / 1000 / 60; 
            const costoEnergiaMinuto = costoKwh * kwhPorMinutoImpresora;

            document.getElementById('derived_costo_energia_minuto').textContent = costoEnergiaMinuto.toFixed(4);
            document.getElementById('costo_energia_minuto').value = costoEnergiaMinuto; 

            // 4. Costo Fijo por Unidad (CF/U)
            const rentaMensual = getVal('renta_mensual');
            const salariosFijosMensual = getVal('salarios_fijos_mensual');
            const serviciosFijosMensual = getVal('servicios_fijos_mensual');
            const unidadesAProducirMensual = getVal('unidades_a_producir_mensual');
            
            const costoFijoTotal = rentaMensual + salariosFijosMensual + serviciosFijosMensual;
            const costoFijoUnidadDerivado = unidadesAProducirMensual > 0 ? costoFijoTotal / unidadesAProducirMensual : 0;

            document.getElementById('derived_costo_fijo_unidad').textContent = costoFijoUnidadDerivado.toFixed(2);
            document.getElementById('costo_fijo_unidad').value = costoFijoUnidadDerivado;


            // --- B. CÁLCULO DE COSTOS POR PRODUCTO (ASIGNACIÓN) ---
            
            const mode = document.getElementById('calculation_mode').value;
            const isLightbox = mode === 'lightbox';
            
            // Costo de Material Directo (MD) - PLA
            const consPla = getVal('consumo_pla');
            const costTotalPla = consPla * unitPla;

            let costTotalLed = 0;
            let cFuente = 0;
            let cPlugHembra = 0; 
            let cEmb = 0;

            if (isLightbox) {
                // Costos MD específicos de Lightbox
                const sectorsLed = getVal('sectores_led');
                const typeLed = document.getElementById('tipo_led').value;
                costTotalLed = sectorsLed * 3 * (typeLed === 'RGB' ? unitLedRGB : unitLedWhite); // Asumiendo 3 LEDs por sector
                cFuente = getVal('costo_fuente');
                cPlugHembra = getVal('costo_plug_hembra'); 
                cEmb = getVal('costo_embalaje');
            }


            // --- VISIBILIDAD DE FILAS EN RESUMEN (PASO 3) ---
            
            const lightboxRows = ['row-leds', 'row-fuente', 'row-plug', 'row-embalaje'];

            lightboxRows.forEach(rowId => {
                const row = document.getElementById(rowId);
                if (row) {
                    if (isLightbox) {
                        row.classList.remove('hidden');
                    } else {
                        row.classList.add('hidden');
                    }
                }
            });


            // Materiales Extra (Iterar sobre consumos dinámicos)
            let costTotalExtras = 0;
            let extrasHTML = '';
            
            document.querySelectorAll('.custom-cons-input').forEach(input => {
                const matId = input.dataset.matId;
                const consQty = parseFloat(input.value) || 0;
                
                const matData = window.appData.customMaterials.find(m => m.id === matId);
                const unitCost = matData && matData.qty > 0 ? matData.cost / matData.qty : 0;
                
                const cost = consQty * unitCost;
                costTotalExtras += cost;
                
                extrasHTML += `<tr><td class="py-1">${matData.name} (${consQty} u)</td><td class="py-1 text-right">$${cost.toFixed(2)}</td></tr>`;
            });
            
            // Insertar el resumen de extras
            const extraStart = document.getElementById('extra-materials-summary-start');
            if (extraStart.parentElement) {
                // Limpiar filas anteriores de extras
                let nextSibling = extraStart.nextElementSibling;
                // Asumimos que la siguiente fila es el subtotal materiales, si no es una fila extra
                while (nextSibling && !nextSibling.classList.contains('font-bold')) {
                    const temp = nextSibling.nextElementSibling;
                    nextSibling.remove();
                    nextSibling = temp;
                }

                // Insertar nuevas filas de extras
                extraStart.insertAdjacentHTML('afterend', extrasHTML);
            }


            // CÁLCULO DE TIEMPOS
            const minutosImpresionTotal = (getVal('horas_impresion') * 60) + getVal('minutos_impresion');
            const modEnsamblajeMin = getVal('mod_ensamblaje_min');
            
            // COSTO TOTAL DE CONVERSIÓN (asignación de tasas variables)
            const cMOD = modEnsamblajeMin * costoModMinuto; // MOD solo aplica a Ensamblaje
            const cEnergia = minutosImpresionTotal * costoEnergiaMinuto; // Energía solo aplica a Impresión
            
            // CF/U Derivado se toma directamente
            const cFijoDerivado = costoFijoUnidadDerivado;


            // RESUMEN DE COSTOS
            const subtotalMateriales = costTotalPla + costTotalLed + cFuente + cPlugHembra + cEmb + costTotalExtras;
            const costoFijoUnidadAdicional = getVal('fijo_por_unidad');

            // Subtotal Producción/Conversión/Fijos = MOD + Energía + CF/U Derivado + Fijo Adicional
            const subtotalProduccion = cMOD + cEnergia + cFijoDerivado + costoFijoUnidadAdicional;
            
            // CUF = Costo Unitario de Fabricación
            const CUF = subtotalMateriales + subtotalProduccion;

            // PRECIO FINAL
            const margen = getVal('margen_ganancia') / 100;
            const comisiones = getVal('comisiones') / 100;

            // Precio de Venta (PV) = CUF / (1 - Margen - Comisiones)
            const divisor = 1 - margen - comisiones;
            const precioFinal = divisor > 0 ? CUF / divisor : CUF * 2; 

            
            // --- C. ACTUALIZACIÓN DEL RESUMEN (PASO 3) ---

            document.getElementById('summary_model_name').textContent = document.getElementById('model_name_input').value || 'Modelo sin nombre';
            document.getElementById('summary_model_type').textContent = isLightbox ? 'Lightbox (Conectado)' : 'Impresión 3D General';
            
            // Valores principales
            document.getElementById('cuf').textContent = `$${CUF.toFixed(2)}`; 
            document.getElementById('precio_final').textContent = `$${precioFinal.toFixed(2)}`;

            // Desglose de Materiales
            document.getElementById('costo_pla_calc').textContent = `$${costTotalPla.toFixed(2)}`;
            document.getElementById('costo_led_calc').textContent = `$${costTotalLed.toFixed(2)}`;
            document.getElementById('costo_fuente_calc').textContent = `$${cFuente.toFixed(2)}`;
            document.getElementById('costo_plug_calc').textContent = `$${cPlugHembra.toFixed(2)}`;
            document.getElementById('costo_embalaje_calc').textContent = `$${cEmb.toFixed(2)}`;
            document.getElementById('subtotal_materiales').textContent = `$${subtotalMateriales.toFixed(2)}`;
            
            // Desglose de Conversión y Fijos
            document.getElementById('costo_mod_calc').textContent = `$${cMOD.toFixed(2)}`;
            document.getElementById('costo_energia_calc').textContent = `$${cEnergia.toFixed(2)}`;
            document.getElementById('costo_fijo_unidad_calc').textContent = `$${cFijoDerivado.toFixed(2)}`;
            document.getElementById('costo_fijo_calc').textContent = `$${costoFijoUnidadAdicional.toFixed(2)}`;
            document.getElementById('subtotal_produccion').textContent = `$${subtotalProduccion.toFixed(2)}`;
            
            // Guardar datos base
            window.saveData(); 
        }

        // Ejecutar cálculo y renderizado inicial al cargar la página
        document.addEventListener('DOMContentLoaded', () => {
            // El initFirebase llama a todo el renderizado y cálculo después de la autenticación.
            window.showStep(1); 
        });

    </script>
</body>
</html>
