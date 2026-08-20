# Instituto medocp---Base-datos-3
## 1. Definición del Dominio del Problema
El proyecto aborda la centralización y optimización del acceso a historias clínicas y la gestión de turnos en un centro medico, sea una clinica, instituto privado u otro. En situaciones de guardia médica o consultas programadas, la velocidad de lectura del historial completo del paciente es un factor crítico.

Los esquemas relacionales tradicionales suelen requerir múltiples combinaciones (*joins*) costosas entre tablas de pacientes, consultas, diagnósticos y coberturas. La solución propuesta utiliza una base de datos orientada a documentos (MongoDB) para consolidar la información médica en lecturas atómicas de alta velocidad, garantizando la flexibilidad requerida para registrar distintas especialidades médicas sin alterar la estructura general.

---

## 2. Esquema de Colecciones (Modelado Conceptual)
A continuación, se detalla la estructura propuesta para los documentos de las colecciones principales del sistema:

### Colección: `pacientes`
```json
{
  "_id": { "$oid": "64f1a2b3c4d5e6f7a8b9c0d1" },
  "dni": "38123456",
  "nombre": "Carlos Mendoza",
  "fecha_nacimiento": "1994-05-12",
  "obra_social": {
    "nombre": "OSDE",
    "plan": "210",
    "numero_afiliado": "1-38123456-0"
  },
  "antecedentes": [
    "Asma",
    "Alergia a la Penicilina"
  ],
  "historia_clinica": [
    {
      "fecha": "2026-08-10T14:30:00Z",
      "medico_id": { "$oid": "64f1a2b3c4d5e6f7a8b9c0d2" },
      "especialidad": "Neumonología",
      "diagnostico": "Bronquitis aguda",
      "tratamiento": "Salbutamol spray cada 8hs",
      "observaciones": "Control evolutivo en 7 días"
    }
  ]
}
```

### Colección: `medicos`
```json
{
  "_id": { "$oid": "64f1a2b3c4d5e6f7a8b9c0d2" },
  "matricula": "MN-85241",
  "nombre": "Dra. Elena Gómez",
  "especialidades": [
    "Neumonología",
    "Pediatría"
  ],
  "telefono_contacto": "+542614556677",
  "dias_atencion": [
    "Lunes",
    "Miércoles"
  ]
}
```

### Colección: `turnos`
```json
{
  "_id": { "$oid": "64f1a2b3c4d5e6f7a8b9c0d3" },
  "paciente_id": { "$oid": "64f1a2b3c4d5e6f7a8b9c0d1" },
  "medico_id": { "$oid": "64f1a2b3c4d5e6f7a8b9c0d2" },
  "fecha_hora": "2026-08-25T10:00:00Z",
  "estado": "Reservado",


## 3. Fundamentación de la Lógica No Relacional

La arquitectura de datos se diseñó en función de los patrones de acceso más frecuentes en el ámbito de salud:

### 1. Documentos Anidados (*Embedded Data*)
* **Historia Clínica en `pacientes`:** El caso de uso más crítico es la consulta médica. Anidar las atenciones dentro del documento del paciente permite recuperar en una única consulta por DNI o ID toda la información clínica, antecedentes y obra social, evitando *lookups* y reduciendo drásticamente la latencia.
* **Obra Social en `pacientes`:** Los datos de cobertura son propios e inherentes al paciente en cada consulta; no justifican una colección separada.

### 2. Referencias (*Normalized Data*)
* **Médicos (`medicos`):** Los profesionales atienden a múltiples pacientes. Si se embebiera toda la información del médico en cada historia clínica o turno, se generaría una duplicación masiva e inconsistencias al momento de actualizar datos de contacto o matrícula. Por ello, solo se guarda `medico_id`.
* **Turnos (`turnos`):** Los turnos son entidades altamente transaccionales y con un ciclo de vida propio (cambian de estado, se cancelan, se reprograman). Mantenerlos como colección independiente evita sobrepasar el límite de 16 MB por documento en MongoDB y permite indexar consultas eficientes por rangos de fecha y estado de ocupación.
  "motivo_consulta": "Control neumonológico"
}
```
