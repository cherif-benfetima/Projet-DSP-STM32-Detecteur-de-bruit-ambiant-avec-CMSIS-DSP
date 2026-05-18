# Projet DSP STM32 – Détecteur de bruit avec CMSIS-DSP

## 1. Objectif du projet

Le but du projet est de réaliser un détecteur de bruit ambiant à l’aide d’une carte STM32 et de la bibliothèque CMSIS-DSP.

Le système :

- acquiert un signal audio depuis un microphone analogique,
- convertit ce signal via l’ADC du STM32,
- calcule la valeur RMS du signal grâce à la fonction `arm_rms_f32()`,
- détecte si le niveau sonore dépasse un seuil,
- allume une LED lorsqu’un environnement devient trop bruyant.

---

# 2. Contexte du projet

Ce système peut être utilisé dans :

- une salle de classe,
- une bibliothèque,
- un open space,
- un laboratoire,
- un couloir.

L’objectif est de surveiller automatiquement le niveau sonore.

Le RMS (Root Mean Square) représente l’énergie moyenne du signal audio. Plus le signal est fort, plus la valeur RMS est élevée.

---

# 3. Technologies utilisées

## Carte utilisée

- STM32 Nucleo-F446RE

## IDE

- STM32CubeIDE

## Bibliothèque DSP

- CMSIS-DSP

## Fonction DSP utilisée

```c
arm_rms_f32()
```

---

# 4. Qu’est-ce que CMSIS ?

CMSIS signifie :

> Cortex Microcontroller Software Interface Standard

CMSIS est une bibliothèque développée par ARM pour les microcontrôleurs Cortex-M.

Elle fournit :

- des fonctions DSP optimisées,
- des fonctions mathématiques,
- un accès standard au cœur ARM.

Dans ce projet, la partie utilisée est :

- CMSIS-DSP

Cette bibliothèque contient des fonctions optimisées comme :

- FIR,
- FFT,
- RMS,
- statistiques,
- opérations vectorielles.

---

# 5. Ce qui a été réalisé dans le TP FIR

Avant le projet final RMS, un TP FIR a été réalisé afin de comprendre l’environnement STM32 et CMSIS-DSP.

## Fonction utilisée dans le TP

```c
arm_fir_f32()
```

## Pipeline FIR

```text
Signal test
↓
Filtre FIR CMSIS-DSP
↓
Signal filtré
↓
DAC
↓
Sortie analogique
```

## Éléments configurés

- HAL
- CMSIS-DSP
- FIR
- DAC
- DMA
- TIM7
- SWV
- GPIO
- interruptions

---

# 6. Structure d’un projet STM32

## Dossiers importants

```text
Core/
Drivers/
Debug/
DSP2026.ioc
```

## Où coder

Principalement dans :

```text
Core/Src/main.c
```

---

# 7. Fonctionnement STM32CubeIDE

Le fichier `.ioc` permet de configurer graphiquement :

- GPIO,
- ADC,
- DAC,
- DMA,
- timers,
- UART.

Ensuite CubeIDE génère automatiquement le code.

Le développeur ajoute ensuite son propre traitement dans les zones :

```c
/* USER CODE BEGIN */
```

et

```c
/* USER CODE END */
```

---

# 8. Architecture finale du projet RMS

## Chaîne de traitement

```text
Microphone analogique
↓
ADC STM32
↓
Buffer d’échantillons
↓
Conversion float
↓
arm_rms_f32()
↓
Comparaison seuil
↓
LED
```

---

# 9. Matériel nécessaire

## Carte

- STM32 Nucleo-F446RE

## Microphone analogique recommandé

- MAX4466
- KY-037
- KY-038

Le microphone doit fournir une sortie analogique compatible 0V → 3.3V.

---

# 10. Configuration CubeMX (.ioc)

## Activer ADC1

Exemple :

- PA0 → ADC1_IN0

## Paramètres ADC

### Resolution

```text
12 bits
```

### Continuous Conversion

```text
ENABLE
```

### DMA Continuous Requests

```text
ENABLE
```

---

# 11. Configuration DMA ADC

Dans ADC1 :

- Add DMA

Mode DMA :

```text
Circular
```

---

# 12. Variables globales

Dans :

```c
/* USER CODE BEGIN PV */
```

Ajouter :

```c
#define BLOCK_SIZE 64

uint16_t adcBuffer[BLOCK_SIZE];

float32_t floatBuffer[BLOCK_SIZE];

float32_t rmsValue;
```

---

# 13. Démarrage ADC DMA

Dans :

```c
/* USER CODE BEGIN 2 */
```

Ajouter :

```c
HAL_ADC_Start_DMA(
    &hadc1,
    (uint32_t*)adcBuffer,
    BLOCK_SIZE
);
```

---

# 14. Calcul RMS

Dans :

```c
/* USER CODE BEGIN WHILE */
```

Ajouter :

```c
for(int i=0; i<BLOCK_SIZE; i++)
{
    floatBuffer[i] =
        ((float32_t)adcBuffer[i] - 2048.0f) / 2048.0f;
}

arm_rms_f32(
    floatBuffer,
    BLOCK_SIZE,
    &rmsValue
);

if(rmsValue > 0.05f)
{
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
}
else
{
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
}
```

---

# 15. Pourquoi convertir les données ADC ?

L’ADC STM32 fournit des valeurs :

```text
0 → 4095
```

Mais les fonctions DSP audio travaillent généralement sur :

```text
-1 → +1
```

La formule :

```c
(adcBuffer[i] - 2048) / 2048
```

permet :

- de recentrer le signal,
- de normaliser les données,
- d’obtenir un signal compatible DSP.

---

# 16. Détection de bruit

## Silence

- RMS faible
- LED éteinte

## Bruit important

- RMS élevé
- LED allumée

---

# 17. Utilisation du SWV

Le SWV (Serial Wire Viewer) permet de visualiser des variables en temps réel.

Variable intéressante à surveiller :

```c
rmsValue
```

Cela permet de voir évoluer le niveau sonore sans utiliser `printf`.

---

# 18. Résultat final attendu

Le système doit :

- acquérir un signal réel depuis un microphone,
- calculer la valeur RMS en temps réel,
- détecter un niveau sonore élevé,
- piloter une LED d’alerte.

---

# 19. Conclusion

Ce projet montre l’utilisation de la bibliothèque CMSIS-DSP sur microcontrôleur STM32.

La fonction `arm_rms_f32()` permet d’estimer efficacement le niveau sonore d’un environnement à partir d’échantillons numériques obtenus via un ADC.

Le projet combine :

- acquisition analogique,
- traitement numérique du signal,
- DSP embarqué,
- HAL STM32,
- CMSIS-DSP,
- décision temps réel.

Il constitue un exemple simple et cohérent de traitement DSP embarqué sur architecture ARM Cortex-M4.

