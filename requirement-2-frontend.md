## CONTEXTO DEL PROYECTO

Crear una aplicación web profesional médica en Next.js para evaluación clínica EN VIVO de Trastornos del Movimiento Funcional usando la escala S-FMDRS (Simplified Functional Movement Disorders Rating Scale) validada científicamente por Nielsen et al. 2017.

## ESPECIFICACIONES TÉCNICAS

### Stack Tecnológico
- **Framework**: Next.js 14+ (App Router)
- **Styling**: Tailwind CSS
- **Componentes UI**: shadcn/ui
- **Iconos**: Lucide React
- **Tipografía**: Inter (sistema de fuentes médicas profesionales)
- **Estado**: React Hooks (useState, useEffect)
- **Datos**: Mock data local (sin backend)

### Paleta de Colores Médica Profesional
```css
/* Primarios */
--medical-blue: #0066CC;
--medical-green: #059669;
--medical-red: #DC2626;
--medical-amber: #D97706;

/* Neutrales */
--gray-50: #F9FAFB;
--gray-100: #F3F4F6;
--gray-200: #E5E7EB;
--gray-700: #374151;
--gray-900: #111827;

/* Severidad */
--severity-0: #10B981; /* Verde - Ninguno */
--severity-1: #FCD34D; /* Amarillo - Leve */
--severity-2: #FB923C; /* Naranja - Moderado */
--severity-3: #DC2626; /* Rojo - Severo */
```

---

## ESTRUCTURA DE LA APLICACIÓN

### Página Principal: `/evaluacion`

La aplicación consiste en **UNA SOLA PÁGINA** con tres secciones principales:

1. **Header**: Información del paciente y progreso
2. **Área Principal**: Protocolo de 10 pasos con cronómetro
3. **Formulario de Puntuación**: Evaluación de 9 regiones

---

## COMPONENTE 1: HEADER CON INFO DEL PACIENTE
```typescript
// Header debe mostrar:
interface PatientInfo {
  id: string;
  nombre: string;
  edad: number;
  sexo: 'M' | 'F';
  fechaEvaluacion: Date;
  evaluador: string;
}

// Ejemplo de datos mock:
const pacienteMock: PatientInfo = {
  id: "P-2024-001",
  nombre: "Juan Pérez Rodríguez",
  edad: 45,
  sexo: "M",
  fechaEvaluacion: new Date(),
  evaluador: "Dr. Carlos Mendoza"
};
```

### Diseño del Header:
```
┌────────────────────────────────────────────────────────┐
│  🏥 Sistema S-FMDRS - Evaluación en Vivo               │
│                                                        │
│  Paciente: Juan Pérez Rodríguez  |  ID: P-2024-001    │
│  Edad: 45 años  |  Sexo: M                            │
│  Fecha: 15 Ene 2024 10:30 AM                          │
│  Evaluador: Dr. Carlos Mendoza                        │
│                                                        │
│  Progreso: Paso 3 de 10  ████████░░░░░░░░ 30%         │
└────────────────────────────────────────────────────────┘
```

---

## COMPONENTE 2: PROTOCOLO DE 10 PASOS

### Estructura de datos del protocolo:
```typescript
interface ProtocolStep {
  paso: number;
  titulo: string;
  duracion: number; // segundos
  instruccionPaciente: string;
  instruccionMedico: string;
  objetivosObservacion: string[];
  ejemplosAnormalidades: string[];
  regionesEvaluar: string[]; // qué regiones S-FMDRS se relacionan
}

const protocoloCompleto: ProtocolStep[] = [
  {
    paso: 1,
    titulo: "Observación en Reposo",
    duracion: 15,
    instruccionPaciente: "Permanezca sentado cómodamente con las manos apoyadas en los apoyabrazos de la silla. Intente relajarse.",
    instruccionMedico: "Observe al paciente desde una vista frontal completa. Busque cualquier movimiento involuntario en estado de reposo.",
    objetivosObservacion: [
      "Temblores en reposo",
      "Sacudidas mioclónicas",
      "Movimientos coreiformes",
      "Posturas distónicas",
      "Tics"
    ],
    ejemplosAnormalidades: [
      "Temblor de manos o piernas en reposo",
      "Sacudidas breves e involuntarias de brazos/piernas",
      "Movimientos serpenteantes de dedos",
      "Posturas anormales sostenidas de cuello/tronco"
    ],
    regionesEvaluar: ["Todas las regiones corporales", "Observación general"]
  },
  {
    paso: 2,
    titulo: "Cara, Cuello y Habla",
    duracion: 15,
    instruccionPaciente: "Por favor, recite los meses del año en voz alta y clara: Enero, Febrero, Marzo, Abril...",
    instruccionMedico: "Acérquese para observar detalles faciales. Evalúe movimientos de cara, cuello y calidad del habla.",
    objetivosObservacion: [
      "Muecas faciales",
      "Blefaroespasmo (cierre involuntario de ojos)",
      "Movimientos de lengua/boca",
      "Temblor o sacudidas de cabeza",
      "Movimientos de cuello (tortícolis)",
      "Disartria o afonía"
    ],
    ejemplosAnormalidades: [
      "Parpadeo excesivo o contracciones faciales",
      "Protrusión involuntaria de la lengua",
      "Cabeza girada o inclinada de forma sostenida",
      "Temblor cefálico (movimiento sí-sí o no-no)",
      "Habla susurrante o entrecortada sin causa orgánica"
    ],
    regionesEvaluar: ["Cara y lengua", "Cabeza y cuello", "Habla"]
  },
  {
    paso: 3,
    titulo: "Manos en Reposo sobre Muslos",
    duracion: 15,
    instruccionPaciente: "Coloque ambas manos sobre sus muslos con las palmas hacia arriba. Mantenga esta posición relajada.",
    instruccionMedico: "Observe específicamente las manos, dedos y antebrazos. Busque temblor, sacudidas o posturas anormales.",
    objetivosObservacion: [
      "Temblor de reposo en manos",
      "Temblor postural fino",
      "Mioclonías de dedos",
      "Distonía de dedos/mano",
      "Movimientos coreiformes"
    ],
    ejemplosAnormalidades: [
      "Temblor rítmico de dedos o manos",
      "Sacudidas breves de dedos individuales",
      "Dedos en posiciones anormales (flexión/extensión sostenida)",
      "Movimientos de píldora (pill-rolling) sin Parkinson"
    ],
    regionesEvaluar: ["Extremidad superior izquierda", "Extremidad superior derecha"]
  },
  {
    paso: 4,
    titulo: "Brazos Extendidos (Postura)",
    duracion: 10,
    instruccionPaciente: "Extienda ambos brazos hacia adelante a la altura de los hombros, con las palmas hacia abajo. Mantenga esta posición.",
    instruccionMedico: "Evalúe temblor postural, deriva de los brazos, y posturas distónicas. Compare ambos lados.",
    objetivosObservacion: [
      "Temblor postural (aparece al sostener postura)",
      "Deriva hacia abajo (debilidad funcional)",
      "Pronación/supinación anormal",
      "Distonía de brazos/manos",
      "Asimetría entre lados"
    ],
    ejemplosAnormalidades: [
      "Temblor irregular de alta amplitud al sostener brazos",
      "Un brazo cae gradualmente (debilidad funcional)",
      "Dedos en postura de pianista",
      "Temblor que aumenta con atención al síntoma"
    ],
    regionesEvaluar: ["Extremidad superior izquierda", "Extremidad superior derecha", "Tronco (estabilidad)"]
  },
  {
    paso: 5,
    titulo: "Prueba Dedo-Nariz",
    duracion: 30,
    instruccionPaciente: "Con su dedo índice, toque su nariz y luego toque mi dedo. Repita esto 5 veces con cada mano.",
    instruccionMedico: "Sostenga su dedo a ~50cm del paciente. Observe coordinación, temblor de acción y exactitud. Evalúe cada brazo por separado.",
    objetivosObservacion: [
      "Temblor de acción (aparece durante movimiento)",
      "Dismetría (falla en llegar al objetivo)",
      "Incoordinación",
      "Bradicinesia (lentitud)",
      "Temblor que empeora al acercarse al objetivo"
    ],
    ejemplosAnormalidades: [
      "Temblor marcado que aparece solo durante el movimiento",
      "Trayectoria errática o en zigzag",
      "Mejora paradójica cuando paciente está distraído",
      "Patrón inconsistente entre repeticiones"
    ],
    regionesEvaluar: ["Extremidad superior izquierda", "Extremidad superior derecha"]
  },
  {
    paso: 6,
    titulo: "Golpeteo Rápido de Dedos",
    duracion: 15,
    instruccionPaciente: "Golpee su dedo pulgar con su dedo índice lo más rápido que pueda. Haga esto con cada mano durante 15 segundos.",
    instruccionMedico: "Evalúe velocidad, amplitud, ritmo y fatiga del movimiento. Compare ambas manos.",
    objetivosObservacion: [
      "Bradicinesia (lentitud anormal)",
      "Decrementación (amplitud disminuye)",
      "Arritmia (irregularidad)",
      "Sacudidas superpuestas",
      "Fatiga excesiva"
    ],
    ejemplosAnormalidades: [
      "Movimientos extremadamente lentos sin causa orgánica",
      "Paradas súbitas en medio de la secuencia",
      "Amplitud errática (muy grande, luego muy pequeña)",
      "Diferencia marcada y variable entre manos"
    ],
    regionesEvaluar: ["Extremidad superior izquierda", "Extremidad superior derecha"]
  },
  {
    paso: 7,
    titulo: "Golpeteo de Talones",
    duracion: 15,
    instruccionPaciente: "Golpee sus talones alternativamente contra el piso lo más rápido que pueda.",
    instruccionMedico: "Observe velocidad, ritmo, amplitud y simetría. Busque sacudidas o irregularidades.",
    objetivosObservacion: [
      "Velocidad del movimiento",
      "Regularidad del ritmo",
      "Simetría entre piernas",
      "Sacudidas mioclónicas",
      "Movimientos coreiformes"
    ],
    ejemplosAnormalidades: [
      "Sacudidas breves superpuestas al movimiento voluntario",
      "Lentitud exagerada sin rigidez orgánica",
      "Patrón completamente irregular",
      "Movimientos bruscos y erráticos"
    ],
    regionesEvaluar: ["Extremidad inferior izquierda", "Extremidad inferior derecha"]
  },
  {
    paso: 8,
    titulo: "Transición Sentado a Parado",
    duracion: 10,
    instruccionPaciente: "Por favor, póngase de pie desde la silla sin usar sus manos si es posible.",
    instruccionMedico: "Observe la secuencia del movimiento, tiempo necesario, uso de manos, y cualquier anormalidad durante la transición.",
    objetivosObservacion: [
      "Tiempo para completar la tarea",
      "Necesidad de usar manos",
      "Patrón de movimiento",
      "Movimientos involuntarios durante transición",
      "Expresión de esfuerzo exagerado"
    ],
    ejemplosAnormalidades: [
      "Múltiples intentos fallidos dramáticos",
      "Sacudidas o espasmos durante el levantamiento",
      "Patrón bizarro o no fisiológico",
      "Debilidad súbita que aparece/desaparece"
    ],
    regionesEvaluar: ["Extremidades inferiores", "Tronco", "Marcha (preparación)"]
  },
  {
    paso: 9,
    titulo: "Bipedestación y Balance",
    duracion: 20,
    instruccionPaciente: "Permanezca de pie con los pies separados al ancho de hombros (10 seg). Luego junte sus pies completamente (10 seg).",
    instruccionMedico: "Evalúe postura, balance, oscilaciones y movimientos involuntarios. Prueba de Romberg modificada.",
    objetivosObservacion: [
      "Postura global (camptocormia, escoliosis)",
      "Oscilaciones excesivas",
      "Balance con ojos abiertos",
      "Movimientos de tronco",
      "Temblor ortostático"
    ],
    ejemplosAnormalidades: [
      "Oscilaciones dramáticas sin caída",
      "Flexión anterior exagerada del tronco",
      "Temblor de piernas al estar de pie",
      "Inestabilidad excesiva que mejora con distracción"
    ],
    regionesEvaluar: ["Tronco y abdomen", "Extremidades inferiores", "Marcha (evaluación previa)"]
  },
  {
    paso: 10,
    titulo: "Marcha (Caminata)",
    duracion: 30,
    instruccionPaciente: "Camine naturalmente hasta el otro lado de la habitación (~5 metros), gire, y regrese al punto de inicio. Use ayudas para caminar si normalmente las usa.",
    instruccionMedico: "Observe patrón de marcha, velocidad, balance, braceo, longitud de paso y consistencia. Esta es la evaluación MÁS IMPORTANTE.",
    objetivosObservacion: [
      "Patrón general de marcha",
      "Velocidad y fluidez",
      "Longitud y simetría de pasos",
      "Balance y oscilaciones",
      "Braceo de brazos",
      "Giro y cambio de dirección",
      "Uso de ayudas técnicas"
    ],
    ejemplosAnormalidades: [
      "Marcha bizarra (no corresponde a patrón neurológico conocido)",
      "Excesiva lentitud sin rigidez",
      "Patrón inconsistente (cambia durante la caminata)",
      "Arrastra pies pero puede puntillas/talones",
      "Mejora dramática con distracción",
      "Patrón de marcha astasia-abasia",
      "Excesivas oscilaciones sin caídas"
    ],
    regionesEvaluar: ["Marcha (puntuación específica)", "Extremidades inferiores", "Tronco"]
  }
];
```

---

## COMPONENTE 3: INTERFAZ DEL PROTOCOLO (PASO A PASO)

### Diseño de la tarjeta de cada paso:
```
┌──────────────────────────────────────────────────────────────┐
│  PASO 3 DE 10                            ████████░░░░ 30%    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🎯 MANOS EN REPOSO SOBRE MUSLOS                             │
│                                                              │
│  ⏱️ Cronómetro: 15 segundos                                  │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  │              🟢 00:08 restantes                        │ │
│  │         ████████████░░░░░░░░░░░░░                     │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌─── 📋 INSTRUCCIONES ───────────────────────────────────┐ │
│  │                                                        │ │
│  │  Para el paciente:                                     │ │
│  │  "Coloque ambas manos sobre sus muslos con las        │ │
│  │   palmas hacia arriba. Mantenga esta posición         │ │
│  │   relajada."                                           │ │
│  │                                                        │ │
│  │  Para el médico:                                       │ │
│  │  Observe específicamente las manos, dedos y           │ │
│  │  antebrazos. Busque temblor, sacudidas o posturas     │ │
│  │  anormales.                                            │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌─── 🔍 QUÉ OBSERVAR ────────────────────────────────────┐ │
│  │                                                        │ │
│  │  ✓ Temblor de reposo en manos                         │ │
│  │  ✓ Temblor postural fino                              │ │
│  │  ✓ Mioclonías de dedos                                │ │
│  │  ✓ Distonía de dedos/mano                             │ │
│  │  ✓ Movimientos coreiformes                            │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌─── ⚠️ EJEMPLOS DE ANORMALIDADES ──────────────────────┐ │
│  │                                                        │ │
│  │  • Temblor rítmico de dedos o manos                   │ │
│  │  • Sacudidas breves de dedos individuales             │ │
│  │  • Dedos en posiciones anormales sostenidas           │ │
│  │  • Movimientos de píldora (pill-rolling)              │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌─── 🎯 REGIONES A EVALUAR ──────────────────────────────┐ │
│  │                                                        │ │
│  │  • Extremidad superior izquierda                      │ │
│  │  • Extremidad superior derecha                        │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  [⏸️ Pausar]  [🔄 Reiniciar]  [⏭️ Siguiente Paso]          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Funcionalidades del cronómetro:
- Cuenta regresiva visual
- Cambio de color: Verde → Amarillo (últimos 5 seg) → Rojo (últimos 3 seg)
- Sonido/beep al finalizar (opcional, con botón mute)
- Botón pausar/reanudar
- Botón reiniciar paso
- Auto-avanza al siguiente paso (con confirmación)

---

## COMPONENTE 4: FORMULARIO DE PUNTUACIÓN (9 REGIONES)

### Después de completar los 10 pasos, mostrar:
```typescript
interface RegionEvaluacion {
  id: string;
  nombre: string;
  descripcion: string;
  ejemplos: string[];
  severidad: 0 | 1 | 2 | 3;
  duracion: 0 | 1 | 2 | 3;
}

const regiones: RegionEvaluacion[] = [
  {
    id: "cara-lengua",
    nombre: "Cara y Lengua",
    descripcion: "Movimientos anormales de músculos faciales, boca, mandíbula o lengua",
    ejemplos: [
      "Muecas repetitivas",
      "Blefaroespasmo",
      "Protrusión de lengua",
      "Contracciones faciales"
    ],
    severidad: 0,
    duracion: 0
  },
  {
    id: "cabeza-cuello",
    nombre: "Cabeza y Cuello",
    descripcion: "Movimientos anormales de cabeza o cuello",
    ejemplos: [
      "Temblor cefálico (sí-sí, no-no)",
      "Tortícolis",
      "Sacudidas mioclónicas",
      "Movimientos rotacionales"
    ],
    severidad: 0,
    duracion: 0
  },
  {
    id: "ext-sup-izq",
    nombre: "Extremidad Superior Izquierda y Hombro",
    descripcion: "Incluye hombro, brazo, antebrazo, mano y dedos izquierdos",
    ejemplos: [
      "Temblor (reposo/postural/acción)",
      "Distonía de mano/dedos",
      "Mioclonías",
      "Debilidad funcional"
    ],
    severidad: 0,
    duracion: 0
  },
  {
    id: "ext-sup-der",
    nombre: "Extremidad Superior Derecha y Hombro",
    descripcion: "Incluye hombro, brazo, antebrazo, mano y dedos derechos",
    ejemplos: [
      "Temblor (reposo/postural/acción)",
      "Distonía de mano/dedos",
      "Mioclonías",
      "Debilidad funcional"
    ],
    severidad: 0,
    duracion: 0
  },
  {
    id: "tronco-abdomen",
    nombre: "Tronco y Abdomen",
    descripcion: "Movimientos anormales de columna, tórax o abdomen",
    ejemplos: [
      "Camptocormia (flexión anterior)",
      "Escoliosis funcional",
      "Movimientos torsionales",
      "Mioclonías abdominales"
    ],
    severidad: 0,
    duracion: 0
  },
  {
    id: "ext-inf-izq",
    nombre: "Extremidad Inferior Izquierda",
    descripcion: "Incluye cadera, muslo, pierna, pie y dedos izquierdos",
    ejemplos: [
      "Temblor de pierna/pie",
      "Distonía de pie",
      "Debilidad funcional",
      "Sacudidas mioclónicas"
    ],
    severidad: 0,
    duracion: 0
  },
  {
    id: "ext-inf-der",
    nombre: "Extremidad Inferior Derecha",
    descripcion: "Incluye cadera, muslo, pierna, pie y dedos derechos",
    ejemplos: [
      "Temblor de pierna/pie",
      "Distonía de pie",
      "Debilidad funcional",
      "Sacudidas mioclónicas"
    ],
    severidad: 0,
    duracion: 0
  },
  {
    id: "marcha",
    nombre: "Marcha",
    descripcion: "Evaluación global del patrón de caminata",
    ejemplos: [
      "Patrón bizarro",
      "Inconsistencia",
      "Lentitud excesiva",
      "Astasia-abasia",
      "Oscilaciones sin caídas"
    ],
    severidad: 0,
    duracion: 0
  },
  {
    id: "habla",
    nombre: "Habla",
    descripcion: "Alteraciones en la producción del lenguaje oral",
    ejemplos: [
      "Disartria funcional",
      "Afonía/susurro",
      "Tartamudeo funcional",
      "Prosodia anormal"
    ],
    severidad: 0,
    duracion: 0
  }
];
```

### Diseño del formulario de puntuación:
```
┌──────────────────────────────────────────────────────────────┐
│  ✅ PROTOCOLO COMPLETADO                                     │
│  Ahora evalúe cada región corporal observada                │
└──────────────────────────────────────────────────────────────┘

┌─── REGIÓN 1/9: CARA Y LENGUA ────────────────────────────────┐
│                                                              │
│  Movimientos anormales de músculos faciales, boca,          │
│  mandíbula o lengua                                          │
│                                                              │
│  Ejemplos:                                                   │
│  • Muecas repetitivas                                        │
│  • Blefaroespasmo                                            │
│  • Protrusión de lengua                                      │
│  • Contracciones faciales                                    │
│                                                              │
│  ┌─── SEVERIDAD ─────────────────────────────────────────┐  │
│  │  ¿Qué tan anormales fueron los movimientos?          │  │
│  │                                                       │  │
│  │  ⦿ 0 - Ninguno (movimientos normales)                │  │
│  │  ○ 1 - Leve (sutil pero presente)                    │  │
│  │  ○ 2 - Moderado (claramente anormal pero funcional)  │  │
│  │  ○ 3 - Severo (muy anormal, interfiere gravemente)   │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌─── DURACIÓN ──────────────────────────────────────────┐  │
│  │  ¿Con qué frecuencia observó los movimientos?        │  │
│  │  (Durante los ~15 segundos observados)               │  │
│  │                                                       │  │
│  │  ⦿ 0 - Ninguna (0% del tiempo)                       │  │
│  │  ○ 1 - Ocasional (visto 1-2 veces, <25%)             │  │
│  │  ○ 2 - Frecuente (intermitente ~50%, va y viene)     │  │
│  │  ○ 3 - Constante (>75% del tiempo, casi continuo)    │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  📝 Notas adicionales (opcional):                            │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Subtotal: 0 + 0 = 0 puntos                                  │
│                                                              │
│  [◀️ Región Anterior]  [Siguiente Región ▶️]                │
│                                                              │
└──────────────────────────────────────────────────────────────┘

[Barra de progreso: Región 1/9  ██░░░░░░░░ 11%]
```

---

## COMPONENTE 5: RESUMEN FINAL Y RESULTADOS

### Al completar las 9 regiones:
┌──────────────────────────────────────────────────────────────┐
│  📊 RESULTADOS DE LA EVALUACIÓN S-FMDRS                      │
└──────────────────────────────────────────────────────────────┘
┌─── PUNTAJE TOTAL ────────────────────────────────────────────┐
│                                                              │
│           S-FMDRS TOTAL: 18/54                               │
│           ████████░░░░░░░░░░░ 33%                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
┌─── DESGLOSE POR REGIONES ────────────────────────────────────┐
│                                                              │
│  Región                      Sev  Dur  Total  Estado        │
│  ─────────────────────────────────────────────────────────  │
│  Cara y lengua               0    0    0     ✅ Normal       │
│  Cabeza y cuello             1    1    2     ⚠️ Leve        │
│  Ext. superior izquierda     2    2    4     🟠 Moderado    │
│  Ext. superior derecha       1    1    2     ⚠️ Leve        │
│  Tronco y abdomen            0    1    1     ⚠️ Leve        │
│  Ext. inferior izquierda     2    3    5     🔴 Severo      │
│  Ext. inferior derecha       1    2    3     🟠 Moderado    │
│  Marcha                      2    2    4     🟠 Moderado    │
│  Habla                       0    0    0     ✅ Normal       │
│                                                              │
│  Regiones afectadas: 6/9 (67%)                               │
│  Mayor compromiso: Extremidad inferior izquierda (5 pts)     │
│                                                              │
└──────────────────────Continuar0:41────────────────────────────────────────┘
┌─── GRÁFICO VISUAL ───────────────────────────────────────────┐
│                                                              │
│  Distribución de severidad:                                  │
│                                                              │
│  🟢 Normal (0):     ███ 2 regiones (22%)                     │
│  ⚠️ Leve (1):      ████ 3 regiones (33%)                    │
│  🟠 Moderado (2):  ████ 3 regiones (33%)                     │
│  🔴 Severo (3):    █ 1 región (11%)                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
┌─── NOTAS CLÍNICAS ───────────────────────────────────────────┐
│                                                              │
│  📝 Observaciones generales (opcional):                      │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Paciente presenta predominancia de síntomas en          │ │
│  │ extremidad inferior izquierda con patrón de marcha      │ │
│  │ inconsistente. Mejoría paradójica con distracción.      │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
┌─── ACCIONES ─────────────────────────────────────────────────┐
│                                                              │
│  [💾 Guardar Evaluación]  [📄 Generar PDF]                   │
│  [📧 Enviar por Email]    [🔙 Volver a Inicio]              │
│                                                              │
└──────────────────────────────────────────────────────────────┘

---

## FUNCIONALIDADES TÉCNICAS CLAVE

### 1. Sistema de Cronómetro
```typescript
// Hook personalizado para cronómetro
function useTimer(initialSeconds: number) {
  const [seconds, setSeconds] = useState(initialSeconds);
  const [isRunning, setIsRunning] = useState(false);
  const [isCompleted, setIsCompleted] = useState(false);

  useEffect(() => {
    if (!isRunning || seconds === 0) return;

    const interval = setInterval(() => {
      setSeconds(prev => {
        if (prev === 1) {
          setIsCompleted(true);
          setIsRunning(false);
          // Reproducir sonido de finalización
          playCompletionSound();
        }
        return prev - 1;
      });
    }, 1000);

    return () => clearInterval(interval);
  }, [isRunning, seconds]);

  const start = () => setIsRunning(true);
  const pause = () => setIsRunning(false);
  const reset = () => {
    setSeconds(initialSeconds);
    setIsCompleted(false);
    setIsRunning(false);
  };

  return { seconds, isRunning, isCompleted, start, pause, reset };
}

// Color del cronómetro según tiempo restante
function getTimerColor(seconds: number, total: number) {
  const percentage = (seconds / total) * 100;
  if (percentage > 33) return 'text-green-600';
  if (percentage > 20) return 'text-yellow-600';
  return 'text-red-600';
}
```

### 2. Validación de Formulario
```typescript
// Validar que severidad y duración sean consistentes
function validateRegion(severidad: number, duracion: number): string | null {
  if (severidad > 0 && duracion === 0) {
    return "⚠️ Si observó movimientos anormales (severidad > 0), la duración no puede ser 0";
  }
  if (severidad === 0 && duracion > 0) {
    return "⚠️ Si no observó movimientos (severidad = 0), la duración debe ser 0";
  }
  if (severidad === 3 && duracion < 2) {
    return "⚠️ Un movimiento severo generalmente tiene duración frecuente o constante";
  }
  return null;
}
```

### 3. Cálculo de Puntaje
```typescript
function calcularPuntajeTotal(regiones: RegionEvaluacion[]): number {
  return regiones.reduce((total, region) => {
    return total + region.severidad + region.duracion;
  }, 0);
}

// Puntaje máximo: 54 (9 regiones × 6 puntos máximo por región)
const puntajeMaximo = 54;
```

### 4. Persistencia de Datos
```typescript
// Guardar progreso en localStorage
function guardarProgreso(data: any) {
  localStorage.setItem('sfmdrs-evaluacion-actual', JSON.stringify({
    ...data,
    timestamp: new Date().toISOString()
  }));
}

// Recuperar si hay evaluación incompleta
function recuperarProgreso() {
  const saved = localStorage.getItem('sfmdrs-evaluacion-actual');
  if (saved) {
    const data = JSON.parse(saved);
    // Verificar si es del mismo día
    const isToday = new Date(data.timestamp).toDateString() === new Date().toDateString();
    if (isToday) return data;
  }
  return null;
}
```

---

## NAVEGACIÓN Y ESTADOS
```typescript
type AppState = 
  | { phase: 'inicio'; step: null }
  | { phase: 'protocolo'; step: number } // 1-10
  | { phase: 'puntuacion'; region: number } // 1-9
  | { phase: 'resultados'; data: EvaluacionCompleta };

// Control de navegación
function NavigationController() {
  const [state, setState] = useState<AppState>({ phase: 'inicio', step: null });

  const iniciarProtocolo = () => setState({ phase: 'protocolo', step: 1 });
  
  const siguientePaso = () => {
    if (state.phase === 'protocolo' && state.step < 10) {
      setState({ phase: 'protocolo', step: state.step + 1 });
    } else if (state.phase === 'protocolo' && state.step === 10) {
      setState({ phase: 'puntuacion', region: 1 });
    } else if (state.phase === 'puntuacion' && state.region < 9) {
      setState({ phase: 'puntuacion', region: state.region + 1 });
    } else if (state.phase === 'puntuacion' && state.region === 9) {
      setState({ phase: 'resultados', data: calcularResultados() });
    }
  };

  return { state, iniciarProtocolo, siguientePaso };
}
```

---

## RESPONSIVE DESIGN

### Desktop (>1024px):
- Layout de dos columnas: Instrucciones (70%) + Info lateral (30%)
- Cronómetro grande y prominente
- Formulario con campos lado a lado

### Tablet (768px - 1024px):
- Layout de una columna
- Cronómetro mediano
- Formulario apilado verticalmente

### Mobile (NO PRIORIDAD - diseño básico funcional):
- Versión simplificada
- Cronómetro pequeño
- Botones grandes para facilitar touch

---

## CRITERIOS DE ÉXITO DEL MVP

### Funcionalidad:
- ✅ Cronómetro funcional con pausar/reiniciar
- ✅ Navegación fluida entre 10 pasos del protocolo
- ✅ Formulario de 9 regiones con validación
- ✅ Cálculo correcto del puntaje total (0-54)
- ✅ Resumen visual con gráficos
- ✅ Persistencia en localStorage

### UX/UI:
- ✅ Interfaz limpia y profesional (aspecto médico)
- ✅ Transiciones suaves entre pasos
- ✅ Indicadores visuales claros de progreso
- ✅ Feedback inmediato en interacciones
- ✅ Validación en tiempo real

### Código:
- ✅ TypeScript con tipos estrictos
- ✅ Componentes reutilizables (Button, Card, RadioGroup)
- ✅ Código limpio y documentado
- ✅ Sin warnings en consola

---

## ESTRUCTURA DE COMPONENTES SUGERIDA
app/
├── evaluacion/
│   ├── page.tsx (componente principal)
│   ├── components/
│   │   ├── Header.tsx
│   │   ├── ProtocolStep.tsx
│   │   ├── Timer.tsx
│   │   ├── RegionForm.tsx
│   │   ├── ResultsSummary.tsx
│   │   └── ProgressBar.tsx
│   ├── hooks/
│   │   ├── useTimer.ts
│   │   ├── useEvaluation.ts
│   │   └── useLocalStorage.ts
│   └── types/
│       ├── protocol.ts
│       ├── evaluation.ts
│       └── patient.ts
└── components/ui/ (shadcn/ui)
├── button.tsx
├── card.tsx
├── radio-group.tsx
├── textarea.tsx
└── progress.tsx

---

## DATOS MOCK INICIALES
```typescript
// Para testing rápido, incluir un paciente de ejemplo
const PACIENTE_EJEMPLO = {
  id: "P-2024-001",
  nombre: "Juan Pérez Rodríguez",
  edad: 45,
  sexo: "M",
  fechaEvaluacion: new Date(),
  evaluador: "Dr. Carlos Mendoza"
};

// Opción para "Llenar con datos de ejemplo" para testing
const EVALUACION_EJEMPLO: RegionEvaluacion[] = [
  { id: "cara-lengua", nombre: "Cara y Lengua", severidad: 0, duracion: 0 },
  { id: "cabeza-cuello", nombre: "Cabeza y Cuello", severidad: 1, duracion: 1 },
  { id: "ext-sup-izq", nombre: "Ext. Sup. Izq", severidad: 2, duracion: 2 },
  // ... etc
];
```

---

## NOTAS IMPORTANTES

1. **Cronómetro**: Debe ser GRANDE y VISIBLE, es crítico para el flujo
2. **Instrucciones**: Siempre separar "Para el paciente" vs "Para el médico"
3. **Validación**: Mostrar warnings pero NO bloquear (médico tiene criterio final)
4. **Autosave**: Guardar progreso cada vez que completa un paso/región
5. **Confirmaciones**: Pedir confirmación antes de acciones destructivas (salir, reiniciar)
6. **Accesibilidad**: Usar semantic HTML, ARIA labels, contraste adecuado

---

## PRIORIDAD DE DESARROLLO

1. **ALTA**: Header + Protocolo de 10 pasos + Cronómetro
2. **ALTA**: Formulario de 9 regiones + Validación básica
3. **MEDIA**: Resumen de resultados + Cálculos
4. **MEDIA**: LocalStorage + Autosave
5. **BAJA**: Exportar PDF, gráficos avanzados

---

**OBJETIVO FINAL**: Una aplicación profesional, médicamente precisa, fácil de usar, que permita a un neurólogo evaluar a un paciente con TNF en consultorio en ~15-20 minutos siguiendo el protocolo científico validado de S-FMDRS.