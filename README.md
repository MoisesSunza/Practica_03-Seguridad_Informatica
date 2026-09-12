# Práctica 3: Algoritmo Híbrido

---

## 1. Explicación del funcionamiento del algoritmo híbrido
El sistema implementa un esquema híbrido que aprovecha la velocidad del cifrado simétrico y la seguridad en la distribución de claves del cifrado asimétrico. El proceso se divide en dos fases:

* **Emisión (Cifrado):** Se genera dinámicamente una clave simétrica AES de sesión (32 bytes) y un Vector de Inicialización (IV). El mensaje original se cifra con AES. Para enviar de forma segura la clave simétrica y el IV al receptor, ambos valores se concatenan (empaquetan) y se cifran utilizando la clave pública RSA del receptor.
* **Recepción (Descifrado):** El receptor utiliza su clave privada RSA para descifrar el "sobre digital" y extraer la clave AES y el IV. Con estos elementos recuperados, descifra el mensaje original en AES, obteniendo el texto en claro. Ambos procesos utilizan codificación Base64 para garantizar que los datos binarios cifrados puedan almacenarse y transmitirse sin pérdida de información.

## 2. Justificación de elecciones de diseño
* **Tamaño de claves:** Se eligió AES de 256 bits (clave de 32 bytes) porque representa el estándar de seguridad más alto actual para algoritmos simétricos. Para RSA, se usa un tamaño de 2048 bits, que es el mínimo recomendado por el NIST para garantizar la inviabilidad de la factorización de números primos a corto/medio plazo.
* **Modo de operación AES (CBC):** Se eligió el modo Cipher Block Chaining (CBC) porque oculta los patrones de datos al mezclar el texto en claro con el bloque cifrado anterior.
* **Tamaño del IV (Vector de Inicialización):** Aunque la especificación indicaba un IV de 32 bytes, el diseño se ajustó a 16 bytes. Esta decisión técnica obedece a que el algoritmo AES tiene un tamaño de bloque fijo y estandarizado de 128 bits (16 bytes). El modo CBC exige estrictamente que el IV tenga el mismo tamaño que el bloque criptográfico, por lo que usar 32 bytes provocaría un error de ejecución en la librería.
* **Esquema de Relleno RSA (PKCS1_OAEP):** En lugar de usar RSA básico, se implementó OAEP con hash SHA-256. Esto añade aleatoriedad al cifrado asimétrico, evitando que el mismo mensaje genere siempre el mismo texto cifrado, lo cual protege contra ataques de oráculo de padding.

## 3. Análisis de seguridad del sistema implementado
El sistema híbrido diseñado mitiga las principales vulnerabilidades de los algoritmos aislados:
* **Mitigación del problema de distribución de claves:** Si usáramos solo AES, compartir la clave de 32 bytes a través de internet sería un riesgo crítico. Al envolverla en RSA, solo el dueño de la clave privada puede descubrir cómo descifrar el mensaje AES.
* **Ataques de Intermediario (Man-in-the-Middle):** Si un atacante intercepta el archivo `mensaje_oculto.json`, solo verá cadenas en Base64. Sin la clave privada RSA de 2048 bits del receptor, descifrar el sobre digital tomaría millones de años con el poder computacional actual.
* **Ataques de Texto Cifrado Escogido (CPA):** La implementación de `PKCS1_OAEP` y el relleno `PKCS7` evitan que los atacantes manipulen los bytes del mensaje para intentar deducir la clave o forzar comportamientos predecibles en la aplicación.
