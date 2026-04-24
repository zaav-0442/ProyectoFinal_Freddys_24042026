¡Excelente elección de stack! Flutter y Firestore son una combinación potente para aplicaciones en tiempo real. 

He notado una pequeña discrepancia: mencionaste primero una app de **comida (pizzas)** y luego un **proyecto veterinario**. Para mantener la coherencia con tu instrucción final, configuraremos la estructura base para el **Proyecto Veterinario**, pero aplicaremos la lógica de CRUD que solicitaste.

---

## 1. Configuración del Agente Global `.agents`

Para que tu IDE (especialmente con extensiones de IA o automatización) reconozca las capacidades de este entorno, estructuramos la carpeta raíz:

```text
.agents/
├── SKILL.md          # Definición de habilidades (Diseño, Código, Scraping)
├── scripts/          # Automatizaciones de despliegue/limpieza
├── ejemplos/         # Snippets de UI y Modelos
└── resources/        # Assets y configuraciones JSON de Firebase
```

### Contenido de `.agents/SKILL.md`
> **Habilidades del Agente:**
> * **Skill de Diseño:** Implementación de Atomic Design en Flutter y Material 3.
> * **Skill de Código:** Generación de CRUDs asíncronos con `cloud_firestore`.
> * **Skill de Scraping:** Automatización para poblar el catálogo de productos/servicios veterinarios.

---

## 2. Prerrequisitos y Verificación del Entorno

Antes de escribir código, asegúrate de que tu terminal responda correctamente:

1.  **Firebase CLI:** Si no lo tienes, instálalo con `npm install -g firebase-tools`.
2.  **Login:** Ejecuta `firebase login` en tu terminal.
3.  **Flutterfire:** Para vincular Flutter con Firebase de forma automática:
    ```bash
    dart pub global activate flutterfire_cli
    flutterfire configure
    ```
    *Selecciona tu proyecto "proyectofreddys" (o el nombre que asignaste en Console Firebase).*

---

## 3. Estructura del Proyecto: `proyectofreddys`

Ejecuta en tu terminal: `flutter create proyectofreddys`. 
Dentro de `lib`, organizaremos el código por capas:

### Configuración de `pubspec.yaml`
Añade estas dependencias para que el CRUD funcione:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
  cupertino_icons: ^1.0.6
```

---

## 4. Desarrollo del CRUD (Módulo: Mascotas/Servicios)



### A. Modelo de Datos (`lib/models/pet_model.dart`)
```dart
class Pet {
  String id;
  String nombre;
  String especie;
  double precioConsulta;

  Pet({required this.id, required this.nombre, required this.especie, required this.precioConsulta});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "especie": especie,
    "precio": precioConsulta,
  };
}
```

### B. Servicio Firestore (`lib/services/firebase_service.dart`)
```dart
import 'package:cloud_firestore/cloud_firestore.dart';

FirebaseFirestore db = FirebaseFirestore.instance;

// CREATE
Future<void> addPet(String nombre, String especie, double precio) async {
  await db.collection('pacientes').add({"nombre": nombre, "especie": especie, "precio": precio});
}

// READ
Stream<QuerySnapshot> getPets() {
  return db.collection('pacientes').snapshots();
}

// UPDATE
Future<void> updatePet(String id, String nuevoNombre) async {
  await db.collection('pacientes').doc(id).update({"nombre": nuevoNombre});
}

// DELETE
Future<void> deletePet(String id) async {
  await db.collection('pacientes').doc(id).delete();
}
```

---

## 5. Interfaz de Usuario (UI)

### Pantalla de Inicio y Listado (`lib/ui/home_page.dart`)
Aquí visualizamos el CRUD en tiempo real usando un `StreamBuilder`.

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Veterinaria Freddy - Pacientes')),
      body: StreamBuilder(
        stream: getPets(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return const Center(child: CircularProgressIndicator());
          var docs = snapshot.data!.docs;
          return ListView.builder(
            itemCount: docs.length,
            itemBuilder: (context, index) {
              return ListTile(
                title: Text(docs[index]['nombre']),
                subtitle: Text(docs[index]['especie']),
                trailing: IconButton(
                  icon: const Icon(Icons.delete, color: Colors.red),
                  onPressed: () => deletePet(docs[index].id),
                ),
                onTap: () {
                  // Lógica para actualizar (Update)
                  updatePet(docs[index].id, "${docs[index]['nombre']} (Chequeado)");
                },
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _displayDialog(context),
        child: const Icon(Icons.add),
      ),
    );
  }

  // Diálogo simple para añadir (Create)
  _displayDialog(BuildContext context) {
    addPet("Nuevo Paciente", "Canino", 25.0);
  }
}
```

### Configuración Final en `main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'ui/home_page.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicializa Firebase
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Vet App',
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
      home: HomePage(),
    );
  }
}
```

---

## Pasos Siguientes para tu Agente Global
1.  **Refinar el Skill de Scraping:** ¿Tienes una fuente de datos (precios de alimentos o servicios) que desees integrar automáticamente a la base de datos?
2.  **Validación:** Ejecuta `flutter run` en tu IDE Antigravity o VS Code y verifica que al añadir un dato en la App, este aparezca instantáneamente en la **Console Firebase**.

¿Deseas que profundicemos en el código de la lógica de "scraping" para el catálogo veterinario?
