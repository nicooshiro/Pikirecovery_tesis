# Piki Recovery — experiencia Unity (Web + VR)

La experiencia recorre el **trabajo invisible**: lo que pasa después de entrenar o competir. El usuario elige su deporte, aparece en la cancha y atraviesa un recorrido lineal:

**Recuperación física → Recuperación nutricional → Vuelta a la calma → Final**

La experiencia es la misma en Web y en VR: narrativa, escenarios, situaciones, objetivos y resultados son idénticos. Solo cambia cómo se interactúa.

| Acción | Web (WebGL) | VR (OpenXR / Meta Quest) |
|---|---|---|
| Apuntar | mouse | rayo del control derecho |
| Elegir / atrapar | clic | gatillo |
| Mantener (aplicar tratamiento, inhalar) | mantener el clic o la barra espaciadora | mantener el gatillo |

## Puesta en marcha (5 minutos)

1. Creá un proyecto **3D** en Unity **2022.3 LTS o Unity 6**. Funciona con *Built-in* o *URP*.
2. Copiá la carpeta `Assets/PikiRecovery` dentro de `Assets/` de tu proyecto.
3. En el menú superior elegí **Piki Recovery → Crear escena**. Eso crea la escena, los materiales base y la agrega a Build Settings.
4. Dale **Play**. Sin visor conectado, corre en modo Web (mouse).

Si usás el paquete *Input System*, en **Project Settings → Player → Active Input Handling** elegí **Both**. El código también funciona con "Input System Package" solo, pero así evitás advertencias.

No hace falta ningún asset externo. Canchas, deportista, alimentos, textos, luces, música y sonidos se generan por código. Todo se puede reemplazar por modelos y audios propios (ver más abajo).

## Compilar

### Web (WebGL)

1. **Piki Recovery → Configurar para WebGL**.
2. **File → Build Settings → Build**.
3. Subí la carpeta generada a cualquier hosting estático (itch.io, Netlify, GitHub Pages).

### VR (Meta Quest u otro visor OpenXR)

1. **Window → Package Manager**: instalá **XR Plugin Management** y **OpenXR**.
2. **Project Settings → XR Plug-in Management**:
   - Activá **OpenXR** en la pestaña Android (Quest) y/o en Windows (PC VR / Quest Link).
   - En *OpenXR → Interaction Profiles* agregá **Oculus Touch Controller Profile**.
3. **Piki Recovery → Configurar para Meta Quest (Android)**.
4. **Build And Run** con el visor conectado por USB. También podés probar sin compilar con Quest Link y Play en el editor.

El juego detecta solo si hay un visor activo. Para probar en el editor sin visor, marcá **Force Web** en el componente `GameManager`.

No usa XR Interaction Toolkit: lee cabeza y controles directamente, así hay menos paquetes que configurar. Si más adelante querés usarlo, solo hay que cambiar `PointerInput`.

## Cómo está armado

```
Assets/PikiRecovery/
├── Scripts/Core/
│   ├── PikiData.cs        Deportes, situaciones corporales, reglas frío/calor/masajes, alimentos, resultado
│   └── GameManager.cs     Arranque, rig Web o VR, flujo lineal entre etapas
├── Scripts/Platform/
│   └── PointerInput.cs    Entrada unificada mouse/controlador + rig VR
├── Scripts/World/
│   ├── Environment.cs     Vestuario, cancha por deporte, estación de recarga, luz de cada etapa
│   ├── BodyFigure.cs      Silueta con zonas marcables y efecto de cada tratamiento
│   ├── PikiAudio.cs       Música y efectos procedurales por etapa
│   └── Mats.cs            Materiales (Built-in/URP) y paleta de Pikirecovery
├── Scripts/UI/Ui3D.cs     Textos, paneles, botones e indicadores en 3D (sirven igual en Web y VR)
├── Scripts/Stages/        Una clase por etapa del recorrido
└── Editor/PikiSetup.cs    Menú "Piki Recovery"
```

Todo lo que se toca es un objeto 3D con collider. Por eso los mismos botones funcionan con el mouse y con el rayo del control, y no hay dos interfaces distintas que mantener.

### Etapas

1. **Elección del deporte** (`SportSelectStage`). Fútbol, básquet, vóley, tenis, hockey o rugby. Define la cancha y las zonas del cuerpo más probables.
2. **Aparición en la cancha** (`ArrivalStage`). Marcador en "FINAL" y el mensaje "El esfuerzo terminó. La recuperación recién comienza."
3. **Recuperación física** (`PhysicalStage`):
   - Aparecen 3 situaciones al azar, cada una con tipo (dolor, molestia, fatiga o rigidez), zona e intensidad. Cambian en cada partida.
   - Ante la pregunta *¿Qué necesita esta zona?* el usuario elige entre Frío, Calor o Masajes.
   - Si acierta, ve "Respuesta correcta.", la explicación, y mantiene presionado para aplicar el tratamiento.
   - Si no acierta, ve "No es la mejor opción para esta situación.", una pista, y vuelve a intentar. Los errores bajan el resultado final.
4. **Recuperación nutricional** (`NutritionStage`):
   - Cambia el ambiente: aparece una estación de recarga y la música toma más pulso.
   - Los alimentos pasan por una cinta y el usuario atrapa los que su cuerpo necesita.
   - Hay tres indicadores (Energía, Hidratación y Reparación) que tienen que quedar en la franja verde, sin pasarse.
   - Los alimentos no prioritarios (gaseosa, papas fritas, golosinas, energizante) aportan poco y restan hidratación.
   - Con el tiempo aumenta la velocidad y aparecen más elementos.
5. **Vuelta a la calma** (`CalmStage`):
   - Baja la luz, la música y el movimiento, y el usuario vuelve a la cancha.
   - Una esfera luminosa se expande (INHALÁ, 4 s) y se contrae (EXHALÁ, 6 s) durante 5 ciclos.
   - Se mide la sincronía, pero nunca se "pierde".
6. **Final** (`SummaryStage`):
   - La cancha aparece tranquila, al atardecer, con el cartel "RECUPERACIÓN COMPLETADA".
   - Resumen de las tres etapas: cada una marcada Completada, con su detalle.
   - Botones para jugar otra vez o cambiar de deporte.

Las opciones del componente `GameManager` (situaciones por partida, duración del minijuego y ciclos de respiración) se ajustan desde el Inspector.

### Reglas de la etapa física

Son una **guía educativa general, no una indicación médica**:

| Situación | Mejor opción |
|---|---|
| Dolor | Frío |
| Rigidez | Calor |
| Fatiga | Masajes |
| Molestia alta | Frío |
| Molestia leve o moderada | Masajes |

Ante un dolor alto, el feedback sugiere consultar a un profesional.

Las reglas están en `RecoveryRules` (`PikiData.cs`), para que las revise y ajuste un kinesiólogo.

## Reemplazar la versión con primitivas

Todo el aspecto visual y sonoro está hecho con primitivas y audio generado. Para pasar a assets reales:

- **Deportista:** reemplazá `BodyFigure.Build()` por un modelo riggeado y ubicá los puntos de cada zona en `anchors`.
- **Canchas:** en `PikiEnvironment.BuildCourt()`, instanciá un prefab por deporte.
- **Alimentos:** agregá un campo `GameObject Prefab` a `FoodItem` y usalo en `NutritionStage.Spawn()`.
- **Audio:** cambiá los clips procedurales de `PikiAudio` por `AudioClip` asignados desde el Inspector.
- **Textos:** usá TextMeshPro en `Ui3D.Label()` si querés tipografías propias.

## Estado

El código se escribió y revisó a mano. Este entorno no tiene Unity ni compilador de C#, así que **no se compiló ni se probó en Unity todavía**. Si al importarlo aparece algún error de compilación, pasame el mensaje de la consola y lo corrijo.
