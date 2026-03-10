# 🎙️ MediVoz
> *Mientras otros médicos escriben, los tuyos solo hablan.*

**MediVoz** es un sistema de captura de expedientes médicos que permite a los médicos registrar información clínica mediante dictado por voz con IA, reduciendo el tiempo de captura sin sacrificar estructura ni cumplimiento normativo.

Desarrollado para el concurso **Deploy Me** — WeWolf + Vinculación FMAT (UADY).

---

## 📋 Tabla de contenidos
- [El problema](#-el-problema)
- [La solución](#-la-solución)
- [Demo](#-demo)
- [Arquitectura](#-arquitectura)
- [Tecnologías](#-tecnologías)
- [Instalación](#-instalación)
- [Uso](#-uso)
- [Modelo de datos](#-modelo-de-datos)
- [Cumplimiento normativo](#-cumplimiento-normativo-nom-004-ssa3-2012)
- [Seguridad](#-seguridad)
- [Equipo](#-equipo)

---

## 🩺 El problema

Los médicos en consulta dedican tiempo valioso a escribir manualmente en el expediente clínico. En entornos de alta demanda — con 20 o más pacientes al día — esto genera:

- ⏱️ Retrasos en la atención al paciente
- 📋 Datos incompletos o anotados después de la consulta
- 😓 Fatiga administrativa acumulada
- ❌ Errores por abreviaciones o escritura apresurada

---

## 💡 La solución

MediVoz combina **dictado por voz con IA** y **formularios inteligentes** para que el médico capture el expediente de forma natural, rápida y sin interrumpir el flujo de la consulta.

### Funcionalidades principales

| Función | Descripción |
|---|---|
| 🎙️ Dictado por voz | El médico habla y Whisper transcribe automáticamente |
| 📝 Formularios inteligentes | Autocompletado de diagnósticos con catálogo CIE-10 |
| 🔒 Autenticación segura | Login con JWT, sesiones protegidas |
| 📁 Expediente estructurado | Campos NOM-004 compliant |
| 🕵️ Trazabilidad | Registro de quién creó y modificó cada expediente |

---

## 🎬 Demo

> 📹 [Ver video de demostración](#) ← *(enlace al video)*

**Flujo principal:**
1. El médico inicia sesión
2. Busca o crea el expediente del paciente
3. Presiona el botón de dictado y habla durante la consulta
4. MediVoz transcribe y organiza la información en los campos correspondientes
5. El médico revisa, ajusta si es necesario y guarda
6. El expediente queda firmado, fechado y trazable

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────┐
│                  FRONTEND                        │
│              Streamlit UI                        │
│   Login │ Pacientes │ Expediente │ Dictado       │
└──────────────────┬──────────────────────────────┘
                   │ HTTP (REST)
┌──────────────────▼──────────────────────────────┐
│                  BACKEND                         │
│               FastAPI                            │
│  /auth  │  /pacientes  │  /expedientes  │ /voz   │
└──────────────────┬──────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
┌───────▼──────┐    ┌─────────▼────────┐
│   SQLite DB  │    │   Whisper (IA)    │
│  SQLAlchemy  │    │  Speech-to-Text   │
└──────────────┘    └──────────────────┘
```

**Decisiones de diseño:**
- **FastAPI** por su velocidad de desarrollo y documentación automática (Swagger)
- **Streamlit** para prototipado rápido de UI sin sacrificar funcionalidad
- **SQLite** para simplicidad en el MVP, reemplazable por PostgreSQL en producción
- **Whisper** por su precisión en español médico sin necesidad de internet (modelo local)

---

## 🛠️ Tecnologías

| Capa | Tecnología | Versión |
|---|---|---|
| Backend | FastAPI | 0.110+ |
| Frontend | Streamlit | 1.32+ |
| Base de datos | SQLite + SQLAlchemy | 2.0+ |
| IA / Voz | OpenAI Whisper | large-v3 |
| Autenticación | JWT (python-jose) | 3.3+ |
| Cifrado | bcrypt | 4.0+ |
| Lenguaje | Python | 3.11+ |

---

## 🚀 Instalación

### Prerequisitos
- Python 3.11 o superior
- pip
- ffmpeg (requerido por Whisper)

```bash
# Instalar ffmpeg en Ubuntu/Debian
sudo apt install ffmpeg

# Instalar ffmpeg en Mac
brew install ffmpeg
```

### Pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/EmiSolis11/MediVoz.git
cd MediVoz

# 2. Crear entorno virtual
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Configurar variables de entorno
cp .env.example .env
# Editar .env con tus configuraciones

# 5. Inicializar la base de datos
python scripts/init_db.py

# 6. Correr el backend
uvicorn app.main:app --reload --port 8000

# 7. En otra terminal, correr el frontend
streamlit run frontend/app.py
```

### Acceso
- **Frontend:** http://localhost:8501
- **API docs:** http://localhost:8000/docs
- **Usuario demo:** `medico@demo.com` / `demo1234`

---

## 🗃️ Modelo de datos

### Paciente
```python
- id
- nombre_completo
- sexo
- fecha_nacimiento
- edad (calculada)
- domicilio
- created_at
- updated_at
```

### Expediente Clínico
```python
- id
- paciente_id
- medico_id (FK → Usuario)

# Historia clínica
- ficha_identificacion
- antecedentes_heredofamiliares
- antecedentes_patologicos
- antecedentes_no_patologicos
- padecimiento_actual

# Exploración física
- signos_vitales (JSON: frecuencia_cardiaca, frecuencia_respiratoria, temperatura)
- peso_kg
- talla_cm
- tension_arterial

# Diagnóstico
- diagnosticos (JSON: [{codigo_cie10, descripcion}])
- pronostico
- indicacion_terapeutica

# Notas
- notas_evolucion
- resultados_estudios

# Trazabilidad
- created_by
- updated_by
- created_at
- updated_at
- firma_medico
```

---

## 📜 Cumplimiento normativo: NOM-004-SSA3-2012

MediVoz fue diseñado considerando los requisitos del expediente clínico electrónico establecidos en la **NOM-004-SSA3-2012**.

### ✅ Campos contemplados

| Requisito NOM-004 | Implementado en MediVoz |
|---|---|
| Datos del paciente (nombre, sexo, edad, domicilio) | ✅ Modelo `Paciente` |
| Datos del establecimiento | ✅ Configuración del sistema |
| Interrogatorio completo | ✅ Sección historia clínica |
| Exploración física (signos vitales, peso, talla, TA) | ✅ Sección exploración física |
| Diagnósticos con clasificación | ✅ Catálogo CIE-10 integrado |
| Pronóstico e indicación terapéutica | ✅ Campos dedicados |
| Notas de evolución | ✅ Sin abreviaturas ambiguas |
| Firma del médico | ✅ Firma digital básica |
| Trazabilidad de autoría | ✅ created_by / updated_by |
| Confidencialidad | ✅ Cifrado + autenticación |

### 📌 Estándares adicionales considerados
- **CIE-10** (Clasificación Internacional de Enfermedades): catálogo de diagnósticos integrado para autocompletado
- **HL7 FHIR**: arquitectura orientada a futura interoperabilidad con sistemas como Agemed

### ⚠️ Alcance del MVP
Este prototipo no constituye una implementación certificada. Demuestra el conocimiento del marco regulatorio y sienta las bases arquitectónicas para una implementación completa.

---

## 🔐 Seguridad

| Mecanismo | Implementación |
|---|---|
| Autenticación | JWT con expiración de sesión |
| Contraseñas | Hash con bcrypt |
| Cifrado de datos sensibles | Campos críticos cifrados en BD |
| Trazabilidad | Log de creación y modificación por usuario |
| Consentimiento informado | Campo documentado por paciente |

---

## 👥 Equipo

| Nombre | Rol | Carrera |
|---|---|---|
| [Nombre] | Backend (FastAPI + BD) | Ciencias de la Computación |
| [Nombre] | IA y captura de voz (Whisper) | Ciencias de la Computación |
| [Nombre] | Frontend + Documentación | Ciencias de la Computación |
| [Nombre] | Dominio médico + Validación clínica | Medicina |

---

## 📄 Licencia

MIT License — libre para uso educativo y de demostración.

---

*Desarrollado para Deploy Me 2025 — WeWolf + Vinculación FMAT, UADY*
*#DeployMeUADY*
