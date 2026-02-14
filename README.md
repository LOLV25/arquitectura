# arquitectura

# Diagramas del Sistema - Instituto Liceo Cristiano

## 1. Arquitectura General del Sistema

```mermaid
graph TB
    subgraph "Frontend - React + Vite"
        A[Interfaz Web] --> B[Módulo Asistencia]
        A --> C[Módulo Rendimiento]
        A --> D[Módulo Justificaciones]
        A --> E[Dashboard Administrativo]
    end
    
    subgraph "Backend - Python/Flask o FastAPI"
        F[API REST] --> G[Servicio Asistencia]
        F --> H[Servicio Rendimiento]
        F --> I[Servicio Justificaciones]
        F --> J[Servicio Autenticación]
        
        G --> K[Reconocimiento Facial]
        H --> L[ML - Predicción Académica]
        I --> M[Servicio Email]
    end
    
    subgraph "Base de Datos - MySQL"
        N[(Estudiantes)]
        O[(Asistencias)]
        P[(Calificaciones)]
        Q[(Justificaciones)]
        R[(Usuarios)]
        S[(Cursos)]
    end
    
    subgraph "Servicios Externos"
        T[Almacenamiento S3/Local]
        U[Servidor SMTP]
        V[OpenCV/Face Recognition]
    end
    
    B --> G
    C --> H
    D --> I
    E --> J
    
    G --> N
    G --> O
    G --> S
    H --> P
    H --> N
    I --> Q
    I --> N
    J --> R
    
    K --> V
    K --> T
    M --> U
    L --> P
    
    style A fill:#61dafb
    style F fill:#3776ab
    style N fill:#4479a1
    style K fill:#ff6b6b
    style L fill:#51cf66
```

## 2. Diagrama de Base de Datos (Modelo Entidad-Relación)

```mermaid
erDiagram
    USUARIOS ||--o{ ESTUDIANTES : "gestiona"
    USUARIOS {
        int id PK
        varchar email UK
        varchar password_hash
        varchar nombre
        varchar apellido
        enum rol
        datetime created_at
        datetime updated_at
    }
    
    CURSOS ||--o{ ESTUDIANTES : "pertenecen"
    CURSOS {
        int id PK
        varchar nombre
        varchar codigo UK
        int grado
        varchar seccion
        int año_academico
        datetime created_at
    }
    
    ESTUDIANTES ||--o{ ASISTENCIAS : "tiene"
    ESTUDIANTES ||--o{ CALIFICACIONES : "obtiene"
    ESTUDIANTES ||--o{ JUSTIFICACIONES : "solicita"
    ESTUDIANTES ||--|| ROSTROS : "tiene"
    ESTUDIANTES {
        int id PK
        varchar rut UK
        varchar nombre
        varchar apellido
        date fecha_nacimiento
        varchar email
        varchar telefono
        int curso_id FK
        varchar foto_perfil
        enum estado
        datetime created_at
        datetime updated_at
    }
    
    ASISTENCIAS ||--|| ESTUDIANTES : "registra"
    ASISTENCIAS {
        int id PK
        int estudiante_id FK
        date fecha
        time hora_entrada
        enum estado
        varchar foto_asistencia
        float confianza_reconocimiento
        int registrado_por FK
        datetime created_at
    }
    
    MATERIAS ||--o{ CALIFICACIONES : "contiene"
    MATERIAS {
        int id PK
        varchar nombre
        varchar codigo UK
        int curso_id FK
        int creditos
        datetime created_at
    }
    
    CALIFICACIONES ||--|| ESTUDIANTES : "pertenece"
    CALIFICACIONES ||--|| MATERIAS : "de"
    CALIFICACIONES {
        int id PK
        int estudiante_id FK
        int materia_id FK
        decimal nota
        int periodo
        enum tipo_evaluacion
        date fecha_evaluacion
        text observaciones
        datetime created_at
    }
    
    JUSTIFICACIONES ||--|| ESTUDIANTES : "de"
    JUSTIFICACIONES ||--o{ USUARIOS : "aprueba"
    JUSTIFICACIONES {
        int id PK
        int estudiante_id FK
        date fecha_inicio
        date fecha_fin
        text motivo
        varchar certificado_url
        enum estado
        int revisado_por FK
        datetime fecha_revision
        text comentario_revision
        datetime created_at
        datetime updated_at
    }
    
    ROSTROS {
        int id PK
        int estudiante_id FK UK
        text encoding_facial
        varchar foto_original
        datetime fecha_registro
        datetime updated_at
    }
    
    PREDICCIONES_ACADEMICAS ||--|| ESTUDIANTES : "para"
    PREDICCIONES_ACADEMICAS ||--|| MATERIAS : "en"
    PREDICCIONES_ACADEMICAS {
        int id PK
        int estudiante_id FK
        int materia_id FK
        float probabilidad_aprobacion
        enum prediccion
        json factores_riesgo
        date fecha_prediccion
        varchar modelo_version
        datetime created_at
    }
    
    PERIODOS_ACADEMICOS ||--o{ CALIFICACIONES : "contiene"
    PERIODOS_ACADEMICOS {
        int id PK
        varchar nombre
        date fecha_inicio
        date fecha_fin
        int año
        enum estado
    }
```

## 3. Diagrama de Flujo - Módulo de Asistencia con Reconocimiento Facial

```mermaid
flowchart TD
    Start([Inicio - Tomar Asistencia]) --> Upload[Profesor sube foto del curso]
    Upload --> Process[Backend recibe imagen]
    Process --> Detect[Detectar rostros en la imagen]
    
    Detect --> ExtractFaces[Extraer cada rostro detectado]
    ExtractFaces --> LoadDB[Cargar encodings faciales de BD]
    
    LoadDB --> Compare[Comparar cada rostro con BD]
    Compare --> Match{¿Coincidencia?}
    
    Match -->|Sí| CheckConfidence{Confianza > 60%?}
    Match -->|No| NotFound[Marcar como no identificado]
    
    CheckConfidence -->|Sí| MarkPresent[Marcar asistencia PRESENTE]
    CheckConfidence -->|No| ManualReview[Requiere revisión manual]
    
    MarkPresent --> SaveDB[Guardar en BD]
    NotFound --> SaveDB
    ManualReview --> SaveDB
    
    SaveDB --> MoreFaces{¿Más rostros?}
    MoreFaces -->|Sí| Compare
    MoreFaces -->|No| GenerateReport[Generar reporte de asistencia]
    
    GenerateReport --> CheckAbsent[Marcar ausentes los no detectados]
    CheckAbsent --> SendNotification[Enviar notificaciones]
    SendNotification --> End([Fin])
    
    style Upload fill:#e3f2fd
    style MarkPresent fill:#c8e6c9
    style NotFound fill:#ffcdd2
    style ManualReview fill:#fff9c4
```

## 4. Diagrama de Flujo - Predicción de Rendimiento Académico

```mermaid
flowchart TD
    Start([Inicio - Predicción Académica]) --> GetStudent[Obtener datos del estudiante]
    GetStudent --> CollectData[Recolectar características]
    
    CollectData --> Features[Preparar features:<br/>- Promedio histórico<br/>- Asistencia %<br/>- Notas parciales<br/>- Tendencia notas<br/>- Participación]
    
    Features --> LoadModel[Cargar modelo ML entrenado]
    LoadModel --> Predict[Realizar predicción]
    
    Predict --> Probability[Calcular probabilidad de aprobación]
    Probability --> Classify{Probabilidad}
    
    Classify -->|>= 70%| Alto[RIESGO BAJO<br/>Alta probabilidad aprobación]
    Classify -->|50-69%| Medio[RIESGO MEDIO<br/>Necesita apoyo]
    Classify -->|< 50%| Bajo[RIESGO ALTO<br/>Intervención urgente]
    
    Alto --> SavePrediction[Guardar predicción en BD]
    Medio --> SavePrediction
    Bajo --> SavePrediction
    
    SavePrediction --> GenerateFactors[Generar factores de riesgo]
    GenerateFactors --> CheckRisk{¿Riesgo Alto/Medio?}
    
    CheckRisk -->|Sí| Alert[Generar alerta para tutores]
    CheckRisk -->|No| NoAlert[Sin alertas]
    
    Alert --> SendEmail[Enviar email a coordinador]
    NoAlert --> Dashboard[Actualizar dashboard]
    SendEmail --> Dashboard
    
    Dashboard --> End([Fin])
    
    style Alto fill:#c8e6c9
    style Medio fill:#fff9c4
    style Bajo fill:#ffcdd2
    style Alert fill:#ff9800
```

## 5. Diagrama de Flujo - Justificación de Faltas

```mermaid
flowchart TD
    Start([Inicio - Justificación]) --> StudentForm[Estudiante llena formulario web]
    StudentForm --> Upload[Sube certificado médico/documento]
    
    Upload --> FillData[Completa:<br/>- Fecha inicio/fin<br/>- Motivo<br/>- Descripción]
    
    FillData --> Submit[Enviar solicitud]
    Submit --> SaveRequest[Guardar en BD<br/>Estado: PENDIENTE]
    
    SaveRequest --> UploadFile[Subir archivo a servidor]
    UploadFile --> GenerateURL[Generar URL de revisión]
    
    GenerateURL --> PrepareEmail[Preparar email con:<br/>- Datos estudiante<br/>- Motivo<br/>- Link revisión]
    
    PrepareEmail --> SendEmail[Enviar a directivos]
    SendEmail --> WaitReview[Esperar revisión]
    
    WaitReview --> DirectorClick[Directivo abre link]
    DirectorClick --> ViewRequest[Ver solicitud completa]
    
    ViewRequest --> ReviewDoc[Revisar certificado]
    ReviewDoc --> Decision{Decisión}
    
    Decision -->|Aprobar| Approve[Marcar APROBADO]
    Decision -->|Rechazar| Reject[Marcar RECHAZADO]
    
    Approve --> AddComment[Agregar comentario opcional]
    Reject --> AddComment
    
    AddComment --> UpdateDB[Actualizar estado en BD]
    UpdateDB --> NotifyStudent[Notificar al estudiante]
    
    NotifyStudent --> UpdateAttendance{¿Aprobado?}
    UpdateAttendance -->|Sí| MarkJustified[Marcar faltas como JUSTIFICADAS]
    UpdateAttendance -->|No| KeepAbsent[Mantener como AUSENTE]
    
    MarkJustified --> End([Fin])
    KeepAbsent --> End
    
    style Approve fill:#c8e6c9
    style Reject fill:#ffcdd2
    style SendEmail fill:#bbdefb
```

## 6. Estructura del Backend (Python)

```mermaid
graph TB
    subgraph "app/"
        A[main.py] --> B[config.py]
        A --> C[database.py]
        
        subgraph "api/"
            D[__init__.py]
            E[routes/]
            F[dependencies.py]
        end
        
        subgraph "models/"
            G[user.py]
            H[student.py]
            I[attendance.py]
            J[grade.py]
            K[justification.py]
            L[course.py]
        end
        
        subgraph "schemas/"
            M[user_schema.py]
            N[student_schema.py]
            O[attendance_schema.py]
            P[grade_schema.py]
            Q[justification_schema.py]
        end
        
        subgraph "services/"
            R[auth_service.py]
            S[face_recognition_service.py]
            T[ml_prediction_service.py]
            U[email_service.py]
            V[file_service.py]
        end
        
        subgraph "routes/"
            W[auth.py]
            X[students.py]
            Y[attendance.py]
            Z[grades.py]
            AA[justifications.py]
        end
        
        subgraph "utils/"
            AB[security.py]
            AC[validators.py]
            AD[helpers.py]
        end
        
        subgraph "ml/"
            AE[model_training.py]
            AF[predictor.py]
            AG[feature_engineering.py]
        end
    end
    
    A --> D
    D --> E
    E --> W
    E --> X
    E --> Y
    E --> Z
    E --> AA
    
    W --> R
    Y --> S
    Z --> T
    AA --> U
    
    style A fill:#3776ab
    style S fill:#ff6b6b
    style T fill:#51cf66
```

## 7. Estructura del Frontend (React + Vite)

```mermaid
graph TB
    subgraph "src/"
        A[main.jsx] --> B[App.jsx]
        
        subgraph "components/"
            C[common/]
            D[attendance/]
            E[grades/]
            F[justifications/]
            G[dashboard/]
        end
        
        subgraph "pages/"
            H[Login.jsx]
            I[Dashboard.jsx]
            J[AttendancePage.jsx]
            K[GradesPage.jsx]
            L[JustificationsPage.jsx]
            M[StudentProfile.jsx]
        end
        
        subgraph "services/"
            N[api.js]
            O[authService.js]
            P[attendanceService.js]
            Q[gradeService.js]
            R[justificationService.js]
        end
        
        subgraph "store/"
            S[authStore.js]
            T[studentStore.js]
            U[attendanceStore.js]
        end
        
        subgraph "hooks/"
            V[useAuth.js]
            W[useAttendance.js]
            X[usePrediction.js]
        end
        
        subgraph "utils/"
            Y[constants.js]
            Z[validators.js]
            AA[formatters.js]
        end
        
        subgraph "assets/"
            AB[images/]
            AC[styles/]
        end
    end
    
    B --> H
    B --> I
    I --> J
    I --> K
    I --> L
    
    J --> D
    K --> E
    L --> F
    
    D --> P
    E --> Q
    F --> R
    
    style B fill:#61dafb
    style N fill:#0088cc
```

## 8. Diagrama de Secuencia - Reconocimiento Facial

```mermaid
sequenceDiagram
    actor Profesor
    participant Frontend
    participant API
    participant FaceService
    participant DB
    participant Storage
    
    Profesor->>Frontend: Sube foto del curso
    Frontend->>API: POST /api/attendance/recognize
    API->>Storage: Guardar imagen original
    Storage-->>API: URL de imagen
    
    API->>FaceService: Procesar imagen
    FaceService->>FaceService: Detectar rostros
    FaceService->>DB: Obtener encodings de estudiantes
    DB-->>FaceService: Lista de encodings
    
    loop Para cada rostro detectado
        FaceService->>FaceService: Comparar con BD
        FaceService->>FaceService: Calcular confianza
        alt Confianza > 60%
            FaceService->>DB: Registrar asistencia PRESENTE
        else Confianza baja
            FaceService->>DB: Marcar para revisión manual
        end
    end
    
    FaceService-->>API: Resultado de reconocimiento
    API->>DB: Marcar ausentes
    API-->>Frontend: Lista de asistencia
    Frontend-->>Profesor: Mostrar resultados
```

## 9. Diagrama de Secuencia - Predicción Académica

```mermaid
sequenceDiagram
    actor Usuario
    participant Frontend
    participant API
    participant MLService
    participant DB
    participant EmailService
    
    Usuario->>Frontend: Solicita predicción
    Frontend->>API: GET /api/predictions/student/{id}
    
    API->>DB: Obtener historial del estudiante
    DB-->>API: Calificaciones, asistencias
    
    API->>MLService: Predecir rendimiento
    MLService->>MLService: Preparar features
    MLService->>MLService: Aplicar modelo ML
    MLService->>MLService: Calcular probabilidad
    MLService-->>API: Predicción + factores riesgo
    
    API->>DB: Guardar predicción
    
    alt Riesgo Alto o Medio
        API->>EmailService: Enviar alerta
        EmailService-->>EmailService: Generar email
        EmailService->>EmailService: Enviar a coordinador
    end
    
    API-->>Frontend: Resultado predicción
    Frontend-->>Usuario: Mostrar análisis
```

## 10. Diagrama de Componentes React

```mermaid
graph TB
    subgraph "Aplicación Principal"
        A[App.jsx]
        B[Router]
    end
    
    subgraph "Módulo Asistencia"
        C[AttendancePage]
        D[AttendanceUpload]
        E[AttendanceList]
        F[FaceRecognitionViewer]
        G[ManualReview]
    end
    
    subgraph "Módulo Rendimiento"
        H[GradesPage]
        I[GradesList]
        J[PredictionCard]
        K[RiskIndicator]
        L[ProgressChart]
    end
    
    subgraph "Módulo Justificaciones"
        M[JustificationsPage]
        N[JustificationForm]
        O[JustificationsList]
        P[ReviewModal]
    end
    
    subgraph "Componentes Comunes"
        Q[Header]
        R[Sidebar]
        S[LoadingSpinner]
        T[ErrorBoundary]
        U[ConfirmDialog]
    end
    
    A --> B
    B --> C
    B --> H
    B --> M
    
    C --> D
    C --> E
    E --> F
    E --> G
    
    H --> I
    H --> J
    J --> K
    J --> L
    
    M --> N
    M --> O
    O --> P
    
    C --> Q
    C --> R
    H --> Q
    H --> R
    M --> Q
    M --> R
    
    style A fill:#61dafb
    style C fill:#e3f2fd
    style H fill:#f3e5f5
    style M fill:#e8f5e9
```
```
