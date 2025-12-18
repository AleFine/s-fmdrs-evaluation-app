## CONTEXTO DEL PROYECTO

Desarrollar una aplicación móvil médica profesional en React Native para la evaluación clínica de Trastornos del Movimiento Funcional (TNF) usando la escala S-FMDRS (Simplified Functional Movement Disorders Rating Scale). La app soporta 3 roles de usuario: Administrador, Médico y Paciente, con sistema completo de autenticación y gestión de tratamientos.

---

## ESPECIFICACIONES TÉCNICAS

### Stack Tecnológico
- **Framework**: React Native (Expo recomendado)
- **Navegación**: React Navigation v6+
- **Estado Global**: React Context API + AsyncStorage
- **UI Components**: React Native Paper o NativeBase
- **Iconos**: React Native Vector Icons (Ionicons, MaterialCommunityIcons)
- **Gráficos**: Victory Native (para charts) o React Native Chart Kit
- **Formularios**: React Hook Form
- **Validación**: Yup o Zod
- **Datos Mock**: JSON locales en memoria (sin backend real)

### Paleta de Colores Médica
```javascript
const COLORS = {
  // Primarios
  primary: '#0066CC',      // Azul médico
  secondary: '#059669',    // Verde
  danger: '#DC2626',       // Rojo
  warning: '#D97706',      // Ámbar
  
  // Neutrales
  background: '#F9FAFB',
  surface: '#FFFFFF',
  border: '#E5E7EB',
  textPrimary: '#111827',
  textSecondary: '#6B7280',
  
  // Severidad (para visualizaciones)
  severity0: '#10B981',    // Verde - Ninguno
  severity1: '#FCD34D',    // Amarillo - Leve
  severity2: '#FB923C',    // Naranja - Moderado
  severity3: '#DC2626',    // Rojo - Severo
};
```

---

## ARQUITECTURA DE LA APLICACIÓN

### 1. SISTEMA DE AUTENTICACIÓN Y ROLES

#### Estructura de Usuarios Mock
```javascript
// mockData/usuarios.js
export const USUARIOS_MOCK = {
  administradores: [
    {
      id: 'admin-001',
      nombres: 'Carlos Alberto',
      apellidos: 'Mendoza Silva',
      dni: '12345678',
      email: 'admin@sfmdrs.com',
      password: 'Admin123',
      rol: 'ADMINISTRADOR',
      fechaCreacion: '2024-01-15T10:00:00Z'
    }
  ],
  medicos: [
    {
      id: 'med-001',
      nombres: 'María Elena',
      apellidos: 'García López',
      dni: '23456789',
      email: 'maria.garcia@hospital.com',
      password: 'xK9mP2vL',
      rol: 'MEDICO',
      especialidad: 'Neurología',
      fechaCreacion: '2024-01-16T09:00:00Z',
      pacientesAsignados: ['pac-001', 'pac-002', 'pac-003']
    },
    {
      id: 'med-002',
      nombres: 'Roberto Carlos',
      apellidos: 'Fernández Torres',
      dni: '34567890',
      email: 'roberto.fernandez@hospital.com',
      password: 'aB7nQ4sM',
      rol: 'MEDICO',
      especialidad: 'Neurología',
      fechaCreacion: '2024-01-17T11:00:00Z',
      pacientesAsignados: ['pac-004']
    }
  ],
  pacientes: [
    {
      id: 'pac-001',
      nombres: 'Juan José',
      apellidos: 'Pérez Rodríguez',
      dni: '45678901',
      email: 'juan.perez@email.com',
      password: 'pL3kR8xN',
      rol: 'PACIENTE',
      edad: 45,
      sexo: 'M',
      fechaCreacion: '2024-01-18T08:00:00Z',
      medicoAsignado: 'med-001',
      tratamientos: ['trat-001', 'trat-002']
    },
    {
      id: 'pac-002',
      nombres: 'Ana María',
      apellidos: 'González Sánchez',
      dni: '56789012',
      email: 'ana.gonzalez@email.com',
      password: 'qW2mT9bV',
      rol: 'PACIENTE',
      edad: 38,
      sexo: 'F',
      fechaCreacion: '2024-01-19T14:00:00Z',
      medicoAsignado: 'med-001',
      tratamientos: ['trat-003']
    },
    {
      id: 'pac-003',
      nombres: 'Luis Alberto',
      apellidos: 'Martínez Ruiz',
      dni: '67890123',
      email: 'luis.martinez@email.com',
      password: 'zX4vC7nM',
      rol: 'PACIENTE',
      edad: 52,
      sexo: 'M',
      fechaCreacion: '2024-01-20T10:00:00Z',
      medicoAsignado: 'med-001',
      tratamientos: ['trat-004']
    },
    {
      id: 'pac-004',
      nombres: 'Carmen Rosa',
      apellidos: 'López Díaz',
      dni: '78901234',
      email: 'carmen.lopez@email.com',
      password: 'hG6tR2kL',
      rol: 'PACIENTE',
      edad: 41,
      sexo: 'F',
      fechaCreacion: '2024-01-21T16:00:00Z',
      medicoAsignado: 'med-002',
      tratamientos: []
    }
  ]
};
```

#### Pantallas de Autenticación

**LoginScreen.js**
```javascript
// Campos:
- Email (input con validación)
- Password (input tipo password)
- Botón "Iniciar Sesión"
- Mostrar error si credenciales inválidas

// Funcionalidad:
- Buscar usuario en USUARIOS_MOCK por email
- Verificar password
- Guardar sesión en AsyncStorage
- Navegar según rol:
  * ADMINISTRADOR → AdminHomeScreen
  * MEDICO → MedicoHomeScreen  
  * PACIENTE → PacienteHomeScreen
```

---

### 2. ROL ADMINISTRADOR

#### Estructura de Navegación Admin
```
AdminNavigator
├── AdminHomeScreen (Tab principal)
├── CrearMedicoScreen
├── CrearPacienteScreen
├── ListaMedicosScreen
└── ListaPacientesScreen
```

#### AdminHomeScreen.js
```javascript
// Diseño:
<ScrollView>
  <Header>
    <Avatar>CA</Avatar>
    <Text>Dr. Carlos Mendoza</Text>
    <Text>Administrador</Text>
  </Header>
  
  <StatsCards>
    <Card>
      <Icon name="doctor" />
      <Text>12 Médicos</Text>
      <Text>Activos</Text>
    </Card>
    <Card>
      <Icon name="account-group" />
      <Text>48 Pacientes</Text>
      <Text>Registrados</Text>
    </Card>
    <Card>
      <Icon name="clipboard-check" />
      <Text>156 Evaluaciones</Text>
      <Text>Este mes</Text>
    </Card>
  </StatsCards>
  
  <QuickActions>
    <Button onPress={navigateToCrearMedico}>
      <Icon name="doctor-plus" />
      <Text>Crear Médico</Text>
    </Button>
    <Button onPress={navigateToCrearPaciente}>
      <Icon name="account-plus" />
      <Text>Crear Paciente</Text>
    </Button>
  </QuickActions>
  
  <RecentActivity>
    <Text>Actividad Reciente</Text>
    <FlatList data={actividadReciente} renderItem={ActivityItem} />
  </RecentActivity>
</ScrollView>
```

#### CrearMedicoScreen.js
```javascript
// Formulario con React Hook Form:
const formSchema = {
  nombres: string().required('Nombres requeridos'),
  apellidos: string().required('Apellidos requeridos'),
  dni: string().length(8, 'DNI debe tener 8 dígitos').required(),
  email: string().email('Email inválido').required(),
  especialidad: string().required()
};

// Campos del formulario:
- Nombres (TextInput)
- Apellidos (TextInput)
- DNI (TextInput numérico, 8 caracteres)
- Email (TextInput tipo email)
- Especialidad (Picker/Dropdown)

// Al guardar:
1. Validar formulario
2. Generar password aleatorio de 8 caracteres
3. Crear objeto médico con id único
4. Agregar a USUARIOS_MOCK.medicos (en memoria)
5. Mostrar modal: "Credenciales generadas:"
   - Email: {email}
   - Password: {password}
   - "Estas credenciales han sido enviadas al correo del médico"
6. Navegar de vuelta a AdminHome
```

#### CrearPacienteScreen.js
```javascript
// Similar a CrearMedicoScreen pero sin especialidad
// Campos adicionales:
- Edad (TextInput numérico)
- Sexo (Radio buttons: Masculino/Femenino)
- Médico asignado (Picker con lista de médicos disponibles)

// Al guardar:
1. Validar formulario
2. Generar password aleatorio de 8 caracteres
3. Crear objeto paciente
4. Actualizar pacientesAsignados del médico seleccionado
5. Mostrar modal con credenciales
6. Navegar de vuelta
```

---

### 3. ROL MÉDICO

#### Estructura de Navegación Médico
```
MedicoNavigator (Bottom Tabs)
├── PacientesTab
│   ├── ListaPacientesScreen
│   └── DetallePacienteScreen
│       └── CrearTratamientoScreen
│           └── EvaluacionS_FMDRSScreen
│               └── ResultadosEvaluacionScreen
├── TratamientosTab
│   ├── ListaTratamientosScreen
│   └── DetalleTratamientoScreen
│       ├── HistorialCuestionariosScreen
│       └── AnálisisIAScreen
└── PerfilTab
    └── PerfilMedicoScreen
```

#### ListaPacientesScreen.js
```javascript
// Muestra TODOS los pacientes del sistema
<SafeAreaView>
  <Header>
    <Text>Mis Pacientes</Text>
    <SearchBar placeholder="Buscar paciente..." />
  </Header>
  
  <FlatList
    data={todosLosPacientes}
    renderItem={({ item }) => (
      <PacienteCard
        nombre={item.nombres + ' ' + item.apellidos}
        edad={item.edad}
        sexo={item.sexo}
        tratamientosActivos={item.tratamientos.filter(t => t.estado === 'ACTIVO').length}
        ultimaEvaluacion={obtenerUltimaEvaluacion(item.id)}
        onPress={() => navigateToDetallePaciente(item.id)}
      />
    )}
  />
</SafeAreaView>

// Diseño de PacienteCard:
┌─────────────────────────────────────┐
│ 👤 Juan José Pérez Rodríguez        │
│ 45 años • Masculino                 │
│                                     │
│ 🏥 1 tratamiento activo             │
│ 📅 Última evaluación: 10 Ene 2024  │
│ 📊 Puntaje S-FMDRS: 18/54           │
│                                     │
│           [Ver Detalles →]          │
└─────────────────────────────────────┘
```

#### DetallePacienteScreen.js
```javascript
<ScrollView>
  <PatientHeader>
    <Avatar size="large">{iniciales}</Avatar>
    <Text variant="h4">{nombreCompleto}</Text>
    <Text>DNI: {dni} • {edad} años</Text>
    <StatusBadge>{estado}</StatusBadge>
  </PatientHeader>
  
  <InfoPersonal>
    <InfoRow label="Email" value={email} />
    <InfoRow label="Fecha de Registro" value={formatDate(fechaCreacion)} />
    <InfoRow label="Médico Tratante" value={nombreMedico} />
  </InfoPersonal>
  
  <TratamientosSection>
    <Text variant="h6">Tratamientos</Text>
    {tratamientos.length === 0 ? (
      <EmptyState>
        <Icon name="clipboard-outline" size={64} color="gray" />
        <Text>Sin tratamientos aún</Text>
        <Button onPress={handleCrearTratamiento}>
          Iniciar Nuevo Tratamiento
        </Button>
      </EmptyState>
    ) : (
      <FlatList
        data={tratamientos}
        renderItem={({ item }) => (
          <TratamientoCard
            numero={item.numero}
            estado={item.estado}
            fechaInicio={item.fechaInicio}
            evaluaciones={item.cuestionarios.length}
            puntajePromedio={calcularPromedio(item.cuestionarios)}
            onPress={() => navigateToDetalleTratamiento(item.id)}
          />
        )}
      />
    )}
  </TratamientosSection>
  
  <FAB
    icon="plus"
    label="Nuevo Tratamiento"
    onPress={handleCrearTratamiento}
  />
</ScrollView>
```

#### CrearTratamientoScreen.js
```javascript
// Modal o pantalla simple
<View>
  <Text variant="h5">Iniciar Nuevo Tratamiento</Text>
  <Text>Paciente: {nombrePaciente}</Text>
  
  <TextInput
    label="Nombre del Tratamiento (opcional)"
    placeholder="Ej: Tratamiento Rehabilitación 2024"
    value={nombreTratamiento}
    onChangeText={setNombreTratamiento}
  />
  
  <TextInput
    label="Objetivo del Tratamiento"
    placeholder="Ej: Reducir temblor en extremidades superiores"
    multiline
    numberOfLines={4}
    value={objetivo}
    onChangeText={setObjetivo}
  />
  
  <DatePicker
    label="Fecha de Inicio"
    value={fechaInicio}
    onChange={setFechaInicio}
  />
  
  <Button mode="contained" onPress={handleCrearYComenzar}>
    Crear Tratamiento y Comenzar Primera Evaluación
  </Button>
</View>

// Al crear:
1. Generar nuevo tratamiento con estado 'ACTIVO'
2. Asignar médico actual como responsable
3. Agregar tratamiento al array del paciente
4. Navegar directamente a EvaluacionS_FMDRSScreen
```

---

### 4. PROTOCOLO S-FMDRS (10 PASOS + 9 REGIONES)

#### EvaluacionS_FMDRSScreen.js
```javascript
// Estado de la evaluación
const [fase, setFase] = useState<'PROTOCOLO' | 'PUNTUACION' | 'ANALISIS'>('PROTOCOLO');
const [pasoActual, setPasoActual] = useState(1);
const [regionActual, setRegionActual] = useState(1);
const [respuestas, setRespuestas] = useState({});
const [tiemposPasos, setTiemposPasos] = useState([]);

// Estructura del protocolo (igual que el prompt de V0)
const PROTOCOLO_10_PASOS = [
  {
    paso: 1,
    titulo: "Observación en Reposo",
    duracion: 15,
    instruccionPaciente: "Permanezca sentado cómodamente...",
    instruccionMedico: "Observe al paciente desde una vista frontal...",
    objetivosObservacion: [...],
    ejemplosAnormalidades: [...],
    regionesEvaluar: [...]
  },
  // ... (todos los 10 pasos completos)
];

// Renderizado según fase
return (
  <SafeAreaView>
    {fase === 'PROTOCOLO' && (
      <ProtocoloStepScreen
        paso={PROTOCOLO_10_PASOS[pasoActual - 1]}
        onComplete={handlePasoCompletado}
        onNext={handleSiguientePaso}
      />
    )}
    
    {fase === 'PUNTUACION' && (
      <PuntuacionRegionScreen
        region={REGIONES_SFMDRS[regionActual - 1]}
        onComplete={handleRegionCompletada}
        onNext={handleSiguienteRegion}
      />
    )}
    
    {fase === 'ANALISIS' && (
      <ResultadosEvaluacionScreen
        respuestas={respuestas}
        puntajeTotal={calcularPuntajeTotal(respuestas)}
        onGuardar={handleGuardarEvaluacion}
      />
    )}
  </SafeAreaView>
);
```

#### ProtocoloStepScreen.js
```javascript
<ScrollView>
  {/* Header con progreso */}
  <ProgressHeader>
    <Text>PASO {paso.paso} DE 10</Text>
    <ProgressBar progress={paso.paso / 10} />
  </ProgressHeader>
  
  {/* Cronómetro principal */}
  <TimerCard>
    <Text variant="h3">{paso.titulo}</Text>
    <Spacer />
    <Timer
      duration={paso.duracion}
      onComplete={() => playSound('complete')}
      color={getTimerColor(segundosRestantes)}
    />
    <TimerControls>
      <IconButton icon="pause" onPress={pausar} />
      <IconButton icon="refresh" onPress={reiniciar} />
      <IconButton icon="skip-next" onPress={siguiente} />
    </TimerControls>
  </TimerCard>
  
  {/* Instrucciones */}
  <Accordion title="📋 Instrucciones" defaultExpanded>
    <InstructionBox type="patient">
      <Text variant="subtitle">Para el paciente:</Text>
      <Text>{paso.instruccionPaciente}</Text>
    </InstructionBox>
    <InstructionBox type="doctor">
      <Text variant="subtitle">Para el médico:</Text>
      <Text>{paso.instruccionMedico}</Text>
    </InstructionBox>
  </Accordion>
  
  {/* Qué observar */}
  <Accordion title="🔍 Qué Observar">
    {paso.objetivosObservacion.map(objetivo => (
      <CheckItem key={objetivo} text={objetivo} />
    ))}
  </Accordion>
  
  {/* Ejemplos de anormalidades */}
  <Accordion title="⚠️ Ejemplos de Anormalidades">
    {paso.ejemplosAnormalidades.map(ejemplo => (
      <ListItem key={ejemplo}>
        <Text>• {ejemplo}</Text>
      </ListItem>
    ))}
  </Accordion>
  
  {/* Regiones a evaluar */}
  <Card>
    <Text variant="subtitle">🎯 Regiones a Evaluar:</Text>
    {paso.regionesEvaluar.map(region => (
      <Chip key={region}>{region}</Chip>
    ))}
  </Card>
  
  <Spacer height={20} />
  
  <Button
    mode="contained"
    onPress={onNext}
    disabled={!cronometroCompletado}
  >
    {paso.paso < 10 ? 'Siguiente Paso' : 'Comenzar Puntuación'}
  </Button>
</ScrollView>
```

#### PuntuacionRegionScreen.js
```javascript
const REGIONES_SFMDRS = [
  {
    id: 'cara-lengua',
    nombre: 'Cara y Lengua',
    descripcion: 'Movimientos anormales de músculos faciales...',
    ejemplos: ['Muecas repetitivas', 'Blefaroespasmo', ...]
  },
  // ... 9 regiones completas
];

<ScrollView>
  <ProgressHeader>
    <Text>REGIÓN {regionActual} DE 9</Text>
    <ProgressBar progress={regionActual / 9} />
  </ProgressHeader>
  
  <RegionCard>
    <Text variant="h5">{region.nombre}</Text>
    <Text>{region.descripcion}</Text>
    
    <ExamplesSection>
      <Text variant="subtitle">Ejemplos:</Text>
      {region.ejemplos.map(ejemplo => (
        <Text key={ejemplo}>• {ejemplo}</Text>
      ))}
    </ExamplesSection>
  </RegionCard>
  
  {/* Severidad */}
  <FormSection>
    <Text variant="h6">SEVERIDAD</Text>
    <Text color="gray">¿Qué tan anormales fueron los movimientos?</Text>
    
    <RadioButton.Group
      onValueChange={value => setSeveridad(Number(value))}
      value={severidad.toString()}
    >
      <RadioOption
        value="0"
        label="0 - Ninguno"
        description="Movimientos normales"
        color={COLORS.severity0}
      />
      <RadioOption
        value="1"
        label="1 - Leve"
        description="Sutil pero presente"
        color={COLORS.severity1}
      />
      <RadioOption
        value="2"
        label="2 - Moderado"
        description="Claramente anormal pero funcional"
        color={COLORS.severity2}
      />
      <RadioOption
        value="3"
        label="3 - Severo"
        description="Muy anormal, interfiere gravemente"
        color={COLORS.severity3}
      />
    </RadioButton.Group>
  </FormSection>
  
  {/* Duración */}
  <FormSection>
    <Text variant="h6">DURACIÓN</Text>
    <Text color="gray">¿Con qué frecuencia observó los movimientos?</Text>
    
    <RadioButton.Group
      onValueChange={value => setDuracion(Number(value))}
      value={duracion.toString()}
    >
      <RadioOption
        value="0"
        label="0 - Ninguna"
        description="0% del tiempo"
      />
      <RadioOption
        value="1"
        label="1 - Ocasional"
        description="Visto 1-2 veces, <25%"
      />
      <RadioOption
        value="2"
        label="2 - Frecuente"
        description="Intermitente ~50%"
      />
      <RadioOption
        value="3"
        label="3 - Constante"
        description=">75% del tiempo"
      />
    </RadioButton.Group>
  </FormSection>
  
  {/* Validación en tiempo real */}
  {validationWarning && (
    <WarningCard>
      <Icon name="alert" color="warning" />
      <Text>{validationWarning}</Text>
    </WarningCard>
  )}
  
  {/* Notas opcionales */}
  <TextInput
    label="Notas adicionales (opcional)"
    multiline
    numberOfLines={3}
    value={notas}
    onChangeText={setNotas}
  />
  
  {/* Subtotal */}
  <SubtotalCard>
    <Text>Subtotal: {severidad} + {duracion} = {severidad + duracion} puntos</Text>
  </SubtotalCard>
  
  <ButtonRow>
    {regionActual > 1 && (
      <Button mode="outlined" onPress={handleAnterior}>
        ← Anterior
      </Button>
    )}
    <Button
      mode="contained"
      onPress={handleSiguiente}
      disabled={!isRegionCompleta()}
    >
      {regionActual < 9 ? 'Siguiente →' : 'Ver Resultados'}
    </Button>
  </ButtonRow>
</ScrollView>
```

---

### 5. RESULTADOS Y ANÁLISIS CON IA

#### ResultadosEvaluacionScreen.js
```javascript
<ScrollView>
  {/* Puntaje Total */}
  <ScoreCard>
    <Text variant="h3">S-FMDRS TOTAL</Text>
    <AnimatedNumber
      value={puntajeTotal}
      suffix="/54"
      fontSize={48}
      color={getScoreColor(puntajeTotal)}
    />
    <ProgressBar progress={puntajeTotal / 54} />
    <Text>Severidad: {getSeveridadCategoria(puntajeTotal)}</Text>
  </ScoreCard>
  
  {/* Desglose por regiones */}
  <Card>
    <Text variant="h6">Desglose por Regiones</Text>
    <DataTable>
      <DataTable.Header>
        <DataTable.Title>Región</DataTable.Title>
        <DataTable.Title numeric>Sev</DataTable.Title>
        <DataTable.Title numeric>Dur</DataTable.Title>
        <DataTable.Title numeric>Total</DataTable.Title>
      </DataTable.Header>
      
      {respuestas.map(region => (
        <DataTable.Row key={region.id}>
          <DataTable.Cell>{region.nombre}</DataTable.Cell>
          <DataTable.Cell numeric>{region.severidad}</DataTable.Cell>
          <DataTable.Cell numeric>{region.duracion}</DataTable.Cell>
          <DataTable.Cell numeric>
            <Badge color={getBadgeColor(region.total)}>
              {region.total}
            </Badge>
          </DataTable.Cell>
        </DataTable.Row>
      ))}
    </DataTable>
  </Card>
  
  {/* Gráfico de barras */}
  <Card>
    <Text variant="h6">Distribución de Afectación</Text>
    <VictoryChart theme={VictoryTheme.material}>
      <VictoryBar
        data={regionesData}
        x="nombre"
        y="total"
        style={{
          data: {
            fill: ({ datum }) => getSeverityColor(datum.total)
          }
        }}
      />
    </VictoryChart>
  </Card>
  
  {/* Análisis con IA */}
  <Card>
    <Text variant="h6">Análisis con Inteligencia Artificial</Text>
    
    <Text color="gray">
      Seleccione el modelo de IA para generar un análisis clínico:
    </Text>
    
    <RadioButton.Group
      onValueChange={setModeloIA}
      value={modeloIA}
    >
      <RadioOption value="claude" label="Claude (Anthropic)" />
      <RadioOption value="gemini" label="Gemini (Google)" />
    </RadioButton.Group>
    
    <Button
      mode="contained"
      onPress={handleGenerarAnalisisIA}
      loading={analizando}
      disabled={analizando}
    >
      {analizando ? 'Analizando...' : 'Generar Análisis'}
    </Button>
    
    {analisisIA && (
      <AnalysisCard>
        <Text variant="subtitle">Análisis Generado:</Text>
        <Markdown>{analisisIA}</Markdown>
        
        <Divider />
        
        <TextInput
          label="Comentarios del Médico"
          placeholder="Agregue sus observaciones clínicas..."
          multiline
          numberOfLines={5}
          value={comentarioMedico}
          onChangeText={setComentarioMedico}
        />
      </AnalysisCard>
    )}
  </Card>
  
  {/* Notas clínicas generales */}
  <TextInput
    label="Observaciones Generales de la Evaluación"
    multiline
    numberOfLines={4}
    value={observacionesGenerales}
    onChangeText={setObservacionesGenerales}
  />
  
  {/* Botones de acción */}
  <ButtonGroup>
    <Button
      mode="contained"
      icon="content-save"
      onPress={handleGuardarEvaluacion}
      loading={guardando}
    >
      Guardar Evaluación
    </Button>
    
    <Button
      mode="outlined"
      icon="file-pdf"
      onPress={handleExportarPDF}
    >
      Exportar PDF
    </Button>
  </ButtonGroup>
</ScrollView>

// Función mock para simular análisis IA
async function handleGenerarAnalisisIA() {
  setAnalizando(true);
  
  // Simular llamada a API (2-3 segundos)
  await new Promise(resolve => setTimeout(resolve, 2500));
  
  // Análisis mock basado en datos reales
  const analisisMock = generarAnalisisMock(respuestas, modeloIA);
  setAnalisisIA(analisisMock);
  
  setAnalizando(false);
}

function generarAnalisisMock(respuestas, modelo) {
  const puntaje = calcularPuntajeTotal(respuestas);
  const regionesMasAfectadas = obtenerTop3RegionesAfectadas(respuestas);
  
  const analisis = `
## Análisis Clínico S-FMDRS
**Generado con: ${modelo === 'claude' ? 'Claude (Anthropic)' : 'Gemini (Google)'}**

### Resumen Ejecutivo
El paciente presenta un puntaje S-FMDRS total de **${puntaje}/54 puntos**, correspondiente a una severidad **${getSeveridadCategoria(puntaje)}**.

### Hallazgos Principales
${regionesMasAfectadas.map((region, i) => `
${i + 1}. **${region.nombre}**: ${region.total} puntos
   - Severidad: ${region.severidad}/3
   - Duración: ${region.duracion}/3
   - Interpretación: ${interpretarRegion(region)}
`).join('\n')}

### Patrones Observados
- Distribución de síntomas: ${analizarDistribucion(respuestas)}
- Lateralidad: ${analizarLateralidad(respuestas)}
- Fenómeno de distracción: ${evaluarDistraccion(respuestas)}

### Recomendaciones
1. ${generarRecomendacion1(puntaje, respuestas)}
2. ${generarRecomendacion2(regionesMasAfectadas)}
3. ${generarRecomendacion3(respuestas)}

### Próximos Pasos
- Seguimiento recomendado en ${calcularDiasSeguimiento(puntaje)} días
- Considerar evaluación con fisioterapeuta especializado
- Documentar progreso fotográfico/videográfico si es posible

---
*Nota: Este análisis es generado por IA y debe ser revisado y validado por el médico tratante.*
  `;
  
  return analisis.trim();
}
```

---

### 6. ROL PACIENTE - DASHBOARD Y PROGRESO

#### Estructura de Navegación Paciente
```
PacienteNavigator (Bottom Tabs)
├── InicioTab
│   └── PacienteDashboardScreen
├── TratamientosTab
│   └── MisTratamientosScreen
│       └── DetalleTratamientoPacienteScreen
│           └── HistorialEvaluacionesScreen
└── PerfilTab
    └── PerfilPacienteScreen
```

#### PacienteDashboardScreen.js
```javascript
<ScrollView>
  {/* Header con info del paciente */}
  <PatientHeader>
    <Avatar size="large">{iniciales}</Avatar>
    <Text variant="h4">¡Hola, {nombre}!</Text>
    <Text color="gray">Bienvenido a tu panel de progreso</Text>
  </PatientHeader>
  
  {tratamientoActivo ? (
    <>
      {/* Indicadores principales */}
      <TwoColumnGrid>
        {/* Mejoría vs anterior */}
        <MetricCard>
          <Icon name="trending-up" size={32} color="green" />
          <AnimatedNumber
            value={porcentajeMejoriaAnterior}
            suffix="%"
            fontSize={36}
            color="green"
          />
          <Text>Mejoría vs Evaluación Anterior</Text>
          <Text color="gray" size="small">
            {ultimasEvaluaciones[0].puntaje} → {ultimasEvaluaciones[1].puntaje} pts
          </Text>
        </MetricCard>
        
        {/* Mejoría vs inicial */}
        <MetricCard>
          <Icon name="chart-line" size={32} color="blue" />
          <AnimatedNumber
            value={porcentajeMejoriaInicial}
            suffix="%"
            fontSize={36}
            color="blue"
          />
          <Text>Mejoría vs Evaluación Inicial</Text>
          <Text color="gray" size="small">
            {evaluacionInicial.puntaje} → {evaluacionActual.puntaje} pts
          </Text>
        </MetricCard>
      </TwoColumnGrid>
      
      {/* Gráfico de evolución temporal */}
      <Card>
        <Text variant="h6">📈 Evolución Temporal</Text>
        <VictoryChart
          theme={VictoryTheme.material}
          padding={{ left: 50, right: 20, top: 20, bottom: 40 }}
        >
          <VictoryLine
            data={historialEvaluaciones.map(ev => ({
              x: new Date(ev.fecha),
              y: ev.puntaje
            }))}
            style={{
              data: { stroke: COLORS.primary, strokeWidth: 3 }
            }}
          />
          <VictoryScatter
            data={historialEvaluaciones.map(ev => ({
              x: new Date(ev.fecha),
              y: ev.puntaje
            }))}
            size={5}
            style={{
              data: { fill: COLORS.primary }
            }}
          />
          <VictoryAxis
            dependentAxis
            label="Puntaje S-FMDRS"
            style={{
              axisLabel: { padding: 40 }
            }}
          />
          <VictoryAxis
            label="Fecha"
            tickFormat={(date) => format(date, 'dd/MM')}
          />
        </VictoryChart>
        <Text color="gray" size="small" style={{ textAlign: 'center' }}>
          Menor puntaje = Mayor mejoría
        </Text>
      </Card>
      
      {/* Mapa de calor corporal */}
      <Card>
        <Text variant="h6">🗺️ Mapa de Calor Corporal</Text>
        <Text color="gray">Distribución actual de síntomas</Text>
        
        <HeatmapBody>
          {/* SVG o imagen del cuerpo humano */}
          <BodySVG>
            {/* Cada región coloreada según severidad */}
            <BodyPart
              part="head"
              color={getHeatColor(regiones.cabezaCuello)}
              onPress={() => showRegionDetail('cabezaCuello')}
            />
            <BodyPart
              part="leftArm"
              color={getHeatColor(regiones.extSupIzq)}
              onPress={() => showRegionDetail('extSupIzq')}
            />
            {/* ... otras partes */}
          </BodySVG>
          
          <HeatmapLegend>
            <LegendItem color={COLORS.severity0} label="Sin síntomas" />
            <LegendItem color={COLORS.severity1} label="Leve" />
            <LegendItem color={COLORS.severity2} label="Moderado" />
            <LegendItem color={COLORS.severity3} label="Severo" />
          </HeatmapLegend>
        </HeatmapBody>
      </Card>
      
      {/* Top 3 áreas mejoradas */}
      <Card>
        <Text variant="h6">🏆 Top 3 Áreas Mejoradas</Text>
        <FlatList
          data={top3AreasMejoradas}
          renderItem={({ item, index }) => (
            <ProgressItem>
              <Badge size="large" color="green">{index + 1}</Badge>
              <View style={{ flex: 1 }}>
                <Text variant="subtitle">{item.nombre}</Text>
                <ProgressBar
                  progress={item.porcentajeMejora / 100}
                  color="green"
                />
                <Text color="gray" size="small">
                  {item.puntajeInicial} pts → {item.puntajeActual} pts
                  ({item.porcentajeMejora}% mejor)
                </Text>
              </View>
            </ProgressItem>
          )}
        />
      </Card>
      
      {/* Área que requiere atención */}
      {areaAtencion && (
        <AlertCard type="warning">
          <Icon name="alert-circle" size={32} color="warning" />
          <View>
            <Text variant="subtitle">⚠️ Área Requiere Atención</Text>
            <Text>{areaAtencion.nombre}</Text>
            <Text color="gray">
              Sin mejora significativa en las últimas evaluaciones
            </Text>
            <Button
              mode="text"
              onPress={() => navigateToRegionDetail(areaAtencion)}
            >
              Ver Detalles
            </Button>
          </View>
        </AlertCard>
      )}
      
      {/* Días para próxima evaluación */}
      <Card>
        <Text variant="h6">📅 Próxima Evaluación</Text>
        <View style={{ flexDirection: 'row', alignItems: 'center' }}>
          <CountdownCircle days={diasParaProximaEvaluacion} />
          <View style={{ marginLeft: 16 }}>
            <Text variant="h4">{diasParaProximaEvaluacion} días</Text>
            <Text color="gray">
              Fecha estimada: {format(proximaEvaluacion, 'dd/MM/yyyy')}
            </Text>
          </View>
        </View>
      </Card>
      
      {/* Progreso hacia meta personalizada */}
      <Card>
        <Text variant="h6">🎯 Progreso Hacia Meta</Text>
        <Text color="gray">{metaPersonalizada.descripcion}</Text>
        
        <GoalProgress>
          <CircularProgress
            value={progresoMeta}
            maxValue={100}
            radius={80}
            activeStrokeColor={COLORS.primary}
            inActiveStrokeColor={COLORS.border}
            title={`${progresoMeta}%`}
            titleStyle={{ fontSize: 24, fontWeight: 'bold' }}
          />
          
          <View style={{ marginTop: 16 }}>
            <Text>Meta: {metaPersonalizada.objetivo}</Text>
            <Text>Actual: {puntajeActual} pts</Text>
            <Text>Falta: {puntajeParaMeta} pts</Text>
          </View>
        </GoalProgress>
      </Card>
      
      {/* Comentarios del médico (último) */}
      {ultimoComentarioMedico && (
        <Card>
          <Text variant="h6">💬 Comentario del Dr. {nombreMedico}</Text>
          <Text color="gray" size="small">
            {format(ultimoComentarioMedico.fecha, 'dd/MM/yyyy HH:mm')}
          </Text>
          
          <CommentBox>
            <Markdown>{ultimoComentarioMedico.texto}</Markdown>
          </CommentBox>
          
          <Button
            mode="text"
            onPress={() => navigateToAllComments()}
          >
            Ver Todos los Comentarios
          </Button>
        </Card>
      )}
    </>
  ) : (
    <EmptyState>
      <Icon name="clipboard-text-outline" size={80} color="gray" />
      <Text variant="h6">Sin Tratamiento Activo</Text>
      <Text color="gray">
        Tu médico aún no ha iniciado un tratamiento.
        Contacta con tu médico para más información.
      </Text>
    </EmptyState>
  )}
</ScrollView>
```

#### MisTratamientosScreen.js (Vista Paciente)
```javascript
<SafeAreaView>
  <Header>
    <Text variant="h5">Mis Tratamientos</Text>
  </Header>
  
  <FlatList
    data={misTratamientos}
    renderItem={({ item }) => (
      <TratamientoCard
        numero={item.numero}
        medico={item.medico.nombre}
        estado={item.estado}
        fechaInicio={item.fechaInicio}
        evaluaciones={item.cuestionarios.length}
        ultimaEvaluacion={item.cuestionarios[item.cuestionarios.length - 1]}
        progresoGeneral={calcularProgresoGeneral(item)}
        onPress={() => navigateToDetalleTratamiento(item.id)}
      />
    )}
    ListEmptyComponent={
      <EmptyState>
        <Text>No tienes tratamientos registrados</Text>
      </EmptyState>
    }
  />
</SafeAreaView>

// Diseño TratamientoCard para paciente:
┌──────────────────────────────────────┐
│ TRATAMIENTO #1                       │
│ Dr. María García López              │
│                                      │
│ 🟢 ACTIVO                            │
│ Inicio: 15 Ene 2024                 │
│                                      │
│ 📊 5 evaluaciones realizadas        │
│ 📈 Mejoría: 35% vs inicial          │
│ 🎯 Última: 18/54 pts (10 Ene 2024) │
│                                      │
│          [Ingresar →]                │
└──────────────────────────────────────┘
```

---

### 7. DATOS MOCK - TRATAMIENTOS Y CUESTIONARIOS

```javascript
// mockData/tratamientos.js
export const TRATAMIENTOS_MOCK = [
  {
    id: 'trat-001',
    numero: 1,
    pacienteId: 'pac-001',
    medicoId: 'med-001',
    estado: 'ACTIVO', // ACTIVO | TERMINADO | CANCELADO
    nombre: 'Tratamiento Rehabilitación Neurológica',
    objetivo: 'Reducir temblor en extremidades superiores y mejorar marcha',
    fechaInicio: '2024-01-15T09:00:00Z',
    fechaFin: null,
    metaPersonalizada: {
      puntajeObjetivo: 10,
      descripcion: 'Reducir puntaje S-FMDRS a menos de 10 puntos'
    },
    cuestionarios: [
      {
        id: 'cues-001',
        numero: 1,
        fecha: '2024-01-15T10:30:00Z',
        tipo: 'INICIAL',
        puntajeTotal: 32,
        tiempoProtocolo: 850, // segundos
        respuestas: [
          {
            regionId: 'cara-lengua',
            nombre: 'Cara y Lengua',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Blefaroespasmo leve ocasional'
          },
          {
            regionId: 'cabeza-cuello',
            nombre: 'Cabeza y Cuello',
            severidad: 2,
            duracion: 2,
            total: 4,
            notas: 'Temblor cefálico intermitente'
          },
          {
            regionId: 'ext-sup-izq',
            nombre: 'Extremidad Superior Izquierda',
            severidad: 3,
            duracion: 3,
            total: 6,
            notas: 'Temblor severo constante, interfiere con AVD'
          },
          {
            regionId: 'ext-sup-der',
            nombre: 'Extremidad Superior Derecha',
            severidad: 2,
            duracion: 3,
            total: 5,
            notas: 'Temblor moderado constante'
          },
          {
            regionId: 'tronco-abdomen',
            nombre: 'Tronco y Abdomen',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Leve escoliosis funcional'
          },
          {
            regionId: 'ext-inf-izq',
            nombre: 'Extremidad Inferior Izquierda',
            severidad: 2,
            duracion: 2,
            total: 4,
            notas: 'Temblor al estar de pie'
          },
          {
            regionId: 'ext-inf-der',
            nombre: 'Extremidad Inferior Derecha',
            severidad: 1,
            duracion: 2,
            total: 3,
            notas: 'Temblor leve frecuente'
          },
          {
            regionId: 'marcha',
            nombre: 'Marcha',
            severidad: 2,
            duracion: 2,
            total: 4,
            notas: 'Patrón inconsistente, mejora con distracción'
          },
          {
            regionId: 'habla',
            nombre: 'Habla',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Disartria leve ocasional'
          }
        ],
        analisisIA: {
          modelo: 'claude',
          timestamp: '2024-01-15T10:45:00Z',
          contenido: `## Análisis Clínico S-FMDRS
**Generado con: Claude (Anthropic)**

### Resumen Ejecutivo
El paciente presenta un puntaje S-FMDRS total de **32/54 puntos**, correspondiente a una severidad **MODERADA**.

### Hallazgos Principales
1. **Extremidad Superior Izquierda**: 6 puntos
   - Severidad: 3/3 (Severo)
   - Duración: 3/3 (Constante)
   - Interpretación: Temblor severo constante que interfiere significativamente con actividades de vida diaria

2. **Extremidad Superior Derecha**: 5 puntos
   - Severidad: 2/3 (Moderado)
   - Duración: 3/3 (Constante)
   - Interpretación: Temblor moderado pero persistente

3. **Cabeza y Cuello**: 4 puntos
   - Severidad: 2/3 (Moderado)
   - Duración: 2/3 (Frecuente)
   - Interpretación: Temblor cefálico intermitente

### Patrones Observados
- **Distribución de síntomas**: Predominancia en extremidades superiores (34% del puntaje total), especialmente hemisferio izquierdo
- **Lateralidad**: Asimetría significativa con mayor afectación izquierda
- **Fenómeno de distracción**: Presente en marcha (documentado)

### Recomendaciones
1. Iniciar fisioterapia especializada enfocada en extremidades superiores, particularmente izquierda
2. Evaluar intervenciones de neurorehabilitación con enfoque cognitivo-conductual
3. Considerar terapia ocupacional para estrategias compensatorias en AVD

### Próximos Pasos
- Seguimiento recomendado en 14 días
- Considerar evaluación con fisioterapeuta especializado
- Documentar progreso fotográfico/videográfico

---
*Nota: Este análisis es generado por IA y debe ser revisado por el médico tratante.*`
        },
        comentarioMedico: 'Paciente motivado, buen candidato para rehabilitación intensiva. Iniciaremos protocolo de fisioterapia 3x/semana.',
        observacionesGenerales: 'Paciente cooperador, signos compatibles con TNF. Ausencia de banderas rojas neurológicas.'
      },
      {
        id: 'cues-002',
        numero: 2,
        fecha: '2024-01-29T11:00:00Z',
        tipo: 'SEGUIMIENTO',
        puntajeTotal: 24,
        tiempoProtocolo: 780,
        respuestas: [
          {
            regionId: 'cara-lengua',
            nombre: 'Cara y Lengua',
            severidad: 0,
            duracion: 0,
            total: 0,
            notas: 'Sin movimientos anormales'
          },
          {
            regionId: 'cabeza-cuello',
            nombre: 'Cabeza y Cuello',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Mejoría notable, temblor ocasional'
          },
          {
            regionId: 'ext-sup-izq',
            nombre: 'Extremidad Superior Izquierda',
            severidad: 2,
            duracion: 2,
            total: 4,
            notas: 'Mejoría significativa, temblor moderado intermitente'
          },
          {
            regionId: 'ext-sup-der',
            nombre: 'Extremidad Superior Derecha',
            severidad: 2,
            duracion: 2,
            total: 4,
            notas: 'Similar evolución que lado izquierdo'
          },
          {
            regionId: 'tronco-abdomen',
            nombre: 'Tronco y Abdomen',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Estable'
          },
          {
            regionId: 'ext-inf-izq',
            nombre: 'Extremidad Inferior Izquierda',
            severidad: 1,
            duracion: 2,
            total: 3,
            notas: 'Mejoría leve'
          },
          {
            regionId: 'ext-inf-der',
            nombre: 'Extremidad Inferior Derecha',
            severidad: 1,
            duracion: 2,
            total: 3,
            notas: 'Estable'
          },
          {
            regionId: 'marcha',
            nombre: 'Marcha',
            severidad: 2,
            duracion: 2,
            total: 4,
            notas: 'Patrón más consistente'
          },
          {
            regionId: 'habla',
            nombre: 'Habla',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Sin cambios'
          }
        ],
        analisisIA: {
          modelo: 'gemini',
          timestamp: '2024-01-29T11:20:00Z',
          contenido: `## Análisis de Evolución Clínica
**Generado con: Gemini (Google)**

### Resumen de Progreso
El paciente muestra una **mejoría del 25%** en el puntaje total S-FMDRS, pasando de 32 a 24 puntos en 14 días.

### Cambios Significativos
- **Extremidad Superior Izquierda**: Reducción de 33% (6→4 pts)
- **Extremidad Superior Derecha**: Reducción de 20% (5→4 pts)  
- **Cara y Lengua**: Resolución completa (2→0 pts)
- **Cabeza y Cuello**: Mejoría del 50% (4→2 pts)

### Análisis de Tendencia
La distribución de mejoría sugiere respuesta favorable a la fisioterapia iniciada. La mayor mejoría en extremidades superiores (objetivo principal) indica adherencia al tratamiento.

### Recomendaciones
1. Continuar fisioterapia actual, excelente evolución
2. Mantener seguimiento cada 14 días
3. Considerar aumentar intensidad gradualmente

*Análisis generado por IA - Validación médica requerida*`
        },
        comentarioMedico: 'Excelente evolución. Paciente muy comprometido con ejercicios en casa. Continuaremos plan actual.',
        observacionesGenerales: 'Progreso notable, especialmente en EESS. Paciente refiere mayor confianza en AVD.'
      },
      {
        id: 'cues-003',
        numero: 3,
        fecha: '2024-02-12T10:00:00Z',
        tipo: 'SEGUIMIENTO',
        puntajeTotal: 18,
        tiempoProtocolo: 720,
        respuestas: [
          {
            regionId: 'cara-lengua',
            nombre: 'Cara y Lengua',
            severidad: 0,
            duracion: 0,
            total: 0,
            notas: ''
          },
          {
            regionId: 'cabeza-cuello',
            nombre: 'Cabeza y Cuello',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Estable'
          },
          {
            regionId: 'ext-sup-izq',
            nombre: 'Extremidad Superior Izquierda',
            severidad: 1,
            duracion: 2,
            total: 3,
            notas: 'Continúa mejorando'
          },
          {
            regionId: 'ext-sup-der',
            nombre: 'Extremidad Superior Derecha',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Mejoría sostenida'
          },
          {
            regionId: 'tronco-abdomen',
            nombre: 'Tronco y Abdomen',
            severidad: 0,
            duracion: 0,
            total: 0,
            notas: 'Resuelto'
          },
          {
            regionId: 'ext-inf-izq',
            nombre: 'Extremidad Inferior Izquierda',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Mejoría progresiva'
          },
          {
            regionId: 'ext-inf-der',
            nombre: 'Extremidad Inferior Derecha',
            severidad: 1,
            duracion: 1,
            total: 2,
            notas: 'Mejoría progresiva'
          },
          {
            regionId: 'marcha',
            nombre: 'Marcha',
            severidad: 2,
            duracion: 2,
            total: 4,
            notas: 'Área que aún requiere atención'
          },
          {
            regionId: 'habla',
            nombre: 'Habla',
            severidad: 1,
            duracion: 2,
            total: 3,
            notas: 'Leve empeoramiento'
          }
        ],
        analisisIA: {
          modelo: 'claude',
          timestamp: '2024-02-12T10:25:00Z',
          contenido: `## Análisis de Evolución - Evaluación #3

### Resumen de Progreso Global
**Mejoría del 44% vs evaluación inicial** (32→18 pts)
**Mejoría del 25% vs evaluación anterior** (24→18 pts)

### Hallazgos Destacados
✅ **Regiones con mejoría sostenida:**
- Extremidades superiores: reducción del 71% vs inicial
- Tronco: resolución completa

⚠️ **Áreas que requieren atención:**
- Marcha: sin cambios (4 pts en las últimas 2 evaluaciones)
- Habla: ligero empeoramiento (2→3 pts)

### Interpretación Clínica
El paciente muestra excelente respuesta al tratamiento en extremidades, alcanzando casi la meta establecida (10 pts). Sin embargo, la persistencia de alteraciones en marcha sugiere necesidad de enfoques complementarios.

### Recomendaciones Actualizadas
1. Incorporar entrenamiento específico de marcha
2. Evaluación fonoaudiológica para disartria
3. Considerar alta de fisioterapia para EESS si mejoría se mantiene

*Análisis IA - Revisión médica obligatoria*`
        },
        comentarioMedico: 'Progreso excelente. Referiremos a fonoaudiología. Marcha será nuevo foco terapéutico.',
        observacionesGenerales: 'Paciente muy satisfecho con evolución. Alta funcionalidad en EESS recuperada.'
      }
    ]
  },
  // ... más tratamientos mock
];
```

---

### 8. KPIs MÉDICOS - DASHBOARD ANALÍTICO

```javascript
// Dashboard opcional para médicos (analítica agregada)
<ScrollView>
  <Text variant="h5">Dashboard Analítico</Text>
  
  {/* KPI 1.1.1: Puntaje S-FMDRS Promedio */}
  <MetricCard>
    <Text variant="h6">Puntaje S-FMDRS Promedio</Text>
    <Text variant="h3">{puntajePromedio}/54</Text>
    <Text color="gray">De {totalPacientes} pacientes activos</Text>
    <TrendIndicator value={tendenciaMensual} />
  </MetricCard>
  
  {/* KPI 1.1.2: Distribución de Severidad */}
  <Card>
    <Text variant="h6">Distribución por Severidad</Text>
    <VictoryPie
      data={[
        { x: "Leve (0-18)", y: porcentajeLeve, fill: COLORS.severity1 },
        { x: "Moderado (19-36)", y: porcentajeModerado, fill: COLORS.severity2 },
        { x: "Severo (37-54)", y: porcentajeSevero, fill: COLORS.severity3 }
      ]}
      colorScale={[COLORS.severity1, COLORS.severity2, COLORS.severity3]}
    />
  </Card>
  
  {/* KPI 1.2.1: Región Más Afectada */}
  <Card>
    <Text variant="h6">Regiones Más Afectadas (Promedio)</Text>
    <VictoryChart horizontal>
      <VictoryBar
        data={regionesPromedio}
        x="nombre"
        y="promedio"
      />
    </VictoryChart>
  </Card>
</ScrollView>
```

---

## FUNCIONALIDADES TÉCNICAS AVANZADAS

### AsyncStorage para Persistencia
```javascript
// utils/storage.js
import AsyncStorage from '@react-native-async-storage/async-storage';

export const StorageKeys = {
  USER_SESSION: '@sfmdrs:userSession',
  USUARIOS: '@sfmdrs:usuarios',
  TRATAMIENTOS: '@sfmdrs:tratamientos',
  DRAFT_EVALUATION: '@sfmdrs:draftEvaluation'
};

export async function saveUserSession(user) {
  try {
    await AsyncStorage.setItem(StorageKeys.USER_SESSION, JSON.stringify(user));
  } catch (error) {
    console.error('Error saving session:', error);
  }
}

export async function getUserSession() {
  try {
    const jsonValue = await AsyncStorage.getItem(StorageKeys.USER_SESSION);
    return jsonValue != null ? JSON.parse(jsonValue) : null;
  } catch (error) {
    console.error('Error reading session:', error);
    return null;
  }
}

export async function clearUserSession() {
  try {
    await AsyncStorage.removeItem(StorageKeys.USER_SESSION);
  } catch (error) {
    console.error('Error clearing session:', error);
  }
}

// Inicializar datos mock en AsyncStorage al primer uso
export async function initializeMockData() {
  try {
    const existingData = await AsyncStorage.getItem(StorageKeys.USUARIOS);
    if (!existingData) {
      await AsyncStorage.setItem(
        StorageKeys.USUARIOS,
        JSON.stringify(USUARIOS_MOCK)
      );
      await AsyncStorage.setItem(
        StorageKeys.TRATAMIENTOS,
        JSON.stringify(TRATAMIENTOS_MOCK)
      );
    }
  } catch (error) {
    console.error('Error initializing data:', error);
  }
}
```

### Context API para Estado Global
```javascript
// contexts/AuthContext.js
import React, { createContext, useState, useEffect, useContext } from 'react';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    checkUserSession();
  }, []);

  async function checkUserSession() {
    const session = await getUserSession();
    setUser(session);
    setLoading(false);
  }

  async function login(email, password) {
    // Buscar usuario en AsyncStorage
    const usuariosJSON = await AsyncStorage.getItem(StorageKeys.USUARIOS);
    const usuarios = JSON.parse(usuariosJSON);
    
    // Buscar en todos los roles
    const allUsers = [
      ...usuarios.administradores,
      ...usuarios.medicos,
      ...usuarios.pacientes
    ];
    
    const foundUser = allUsers.find(
      u => u.email === email && u.password === password
    );
    
    if (!foundUser) {
      throw new Error('Credenciales inválidas');
    }
    
    await saveUserSession(foundUser);
    setUser(foundUser);
    
    return foundUser;
  }

  async function logout() {
    await clearUserSession();
    setUser(null);
  }

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

---

## ESTRUCTURA DE ARCHIVOS SUGERIDA

```
src/
├── screens/
│   ├── auth/
│   │   └── LoginScreen.js
│   ├── admin/
│   │   ├── AdminHomeScreen.js
│   │   ├── CrearMedicoScreen.js
│   │   ├── CrearPacienteScreen.js
│   │   └── ListasScreen.js
│   ├── medico/
│   │   ├── ListaPacientesScreen.js
│   │   ├── DetallePacienteScreen.js
│   │   ├── CrearTratamientoScreen.js
│   │   ├── EvaluacionS_FMDRSScreen.js
│   │   ├── ProtocoloStepScreen.js
│   │   ├── PuntuacionRegionScreen.js
│   │   └── ResultadosEvaluacionScreen.js
│   └── paciente/
│       ├── PacienteDashboardScreen.js
│       ├── MisTratamientosScreen.js
│       └── DetalleTratamientoScreen.js
├── components/
│   ├── common/
│   │   ├── Header.js
│   │   ├── Button.js
│   │   ├── Card.js
│   │   ├── ProgressBar.js
│   │   └── EmptyState.js
│   ├── auth/
│   │   └── LoginForm.js
│   ├── evaluation/
│   │   ├── Timer.js
│   │   ├── ProtocolStep.js
│   │   ├── RegionForm.js
│   │   └── ResultsSummary.js
│   └── dashboard/
│       ├── MetricCard.js
│       ├── HeatmapBody.js
│       ├── EvolutionChart.js
│       └── ProgressGoal.js
├── navigation/
│   ├── AppNavigator.js
│   ├── AuthNavigator.js
│   ├── AdminNavigator.js
│   ├── MedicoNavigator.js
│   └── PacienteNavigator.js
├── contexts/
│   ├── AuthContext.js
│   └── EvaluationContext.js
├── data/
│   ├── mockUsuarios.js
│   ├── mockTratamientos.js
│   ├── protocoloSFMDRS.js
│   └── regionesSFMDRS.js
├── utils/
│   ├── storage.js
│   ├── calculations.js
│   ├── validators.js
│   ├── dateHelpers.js
│   └── passwordGenerator.js
├── constants/
│   ├── colors.js
│   └── routes.js
└── App.js
```

---

## PRIORIDADES DE DESARROLLO

### FASE 1 (Esencial)
1. ✅ Autenticación y roles
2. ✅ Navegación por roles
3. ✅ CRUD Administrador (crear médicos/pacientes)
4. ✅ Protocolo de 10 pasos con cronómetro
5. ✅ Formulario de 9 regiones

### FASE 2 (Importante)
6. ✅ Gestión de tratamientos
7. ✅ Resultados y análisis mock IA
8. ✅ Dashboard paciente con indicadores básicos
9. ✅ Historial de evaluaciones
10. ✅ Comentarios médico

### FASE 3 (Complementaria)
11. ✅ Gráficos avanzados (evolución temporal)
12. ✅ Mapa de calor corporal
13. ✅ KPIs médicos agregados
14. ✅ Exportar PDF (opcional)

---

## NOTAS FINALES

### Para Antigravity:
- Priorizar funcionalidad sobre animaciones complejas
- Usar React Native Paper para UI consistente
- Implementar validaciones en formularios
- Guardar progreso en AsyncStorage cada paso importante
- Mock de análisis IA debe parecer realista (con delay de 2-3 seg)

### Datos Mock:
- Incluir al menos 2 médicos, 4 pacientes
- 2-3 tratamientos con historial de evaluaciones
- Variedad de puntajes para demostrar evolución
- Comentarios médicos realistas

### UX Móvil:
- Botones táctiles grandes (mínimo 44x44pt)
- Textos legibles (mínimo 16sp)
- Navegación intuitiva con bottom tabs
- Confirmaciones para acciones destructivas
- Indicadores de carga para operaciones async

**OBJETIVO**: Aplicación móvil profesional, completa y funcional con datos mock que simule todo el flujo de trabajo real del sistema S-FMDRS, lista para que Copilot implemente el backend real posteriormente.
