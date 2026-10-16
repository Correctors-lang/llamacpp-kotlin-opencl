# llamacpp-kotlin-opencl

`llama.cpp` pour Android, empaqueté en `.aar` pour Kotlin, avec deux choses que
les versions publiées n'avaient pas :

1. **Le backend OpenCL** (GPU Adreno), que `llama.cpp` doit à Qualcomm. Une
   septième variante, `rnllama_v8_2_dotprod_i8mm_opencl`, est construite avec ;
   les six variantes processeur sont inchangées, et le chargeur retombe dessus
   quand le pilote OpenCL manque. Vulkan, lui, rendait un
   `vk::Queue::submit: ErrorDeviceLost` sur le téléphone d'essai.
2. **Le préchauffage** (`warmup` de `common/common.cpp`), qui n'était exposé ni
   au JNI ni au Kotlin : le premier lancement du modèle était très long, une
   bonne partie du bénéfice de `use_mmap=true` étant perdue.

Construit pour le [clavier AZERTY](https://github.com/Correctors-lang), qui s'en
sert pour l'IA locale (reformuler, corriger, écrire), hors ligne.

## Prendre la bibliothèque

L'archive `.aar` est jointe à chaque publication (onglet *Releases*) :

```kotlin
dependencies {
    implementation(files("libs/llamacpp-kotlin-opencl-0.5.0.aar"))
}
```

## La reconstruire

Les sources sont dans les forks de l'organisation, branche par branche :

- [`Correctors-lang/kotlinllamacpp`, branche `opencl-warmup`](https://github.com/Correctors-lang/kotlinllamacpp/tree/opencl-warmup)
  — l'empaquetage Kotlin, le JNI et le CMake qui ajoute la variante OpenCL ;
- [`Correctors-lang/cui-llama.rn`, branche `ndk-27.2`](https://github.com/Correctors-lang/cui-llama.rn/tree/ndk-27.2)
  — d'où viennent les sources de `llama.cpp` et le script
  `scripts/build-opencl.sh`, qui construit `libOpenCL.so` depuis
  l'ICD Khronos.

```bash
./scripts/build-opencl.sh          # libOpenCL.so (OpenCL-ICD-Loader, Khronos)
./gradlew :llamaCpp:assembleRelease
```

Il faut le NDK Android 27.2 ; le script nomme la version qu'il attend.

## D'où ça vient, et sous quelle licence

| Projet | Licence | Ce qu'on en a pris |
|---|---|---|
| [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) | MIT | le moteur, et son backend OpenCL |
| [mybigday/llama.rn](https://github.com/mybigday/llama.rn) | MIT | les liaisons Android |
| [Vali-98/cui-llama.rn](https://github.com/Vali-98/cui-llama.rn) | MIT | les variantes par jeu d'instructions, le script OpenCL |
| [ljcamargo/kotlinllamacpp](https://github.com/ljcamargo/kotlinllamacpp) | MIT | l'empaquetage Kotlin et le JNI |
| [KhronosGroup/OpenCL-ICD-Loader](https://github.com/KhronosGroup/OpenCL-ICD-Loader) | Apache 2.0 | `libOpenCL.so`, construit à la demande |

Tout est permissif et utilisable commercialement, moyennant citation. Nos
propres modifications sont sous licence MIT (voir `LICENSE`).
