# grep Cheatsheet · Hoja de referencia de comandos grep

> 🌍 [中文](README.md) · [Português](README.pt.md)

Una **hoja de referencia de una sola página** con 30 usos reales de `grep`: las opciones de búsqueda de cada día; recursión y filtros de archivos; los tres sabores de regex y cuánto de la línea debe coincidir; contexto y formato de salida; y el comportamiento en tuberías del que dependen los scripts (códigos de salida, archivos binarios, registros NUL, flujos en vivo). Cada entrada responde solo tres preguntas: **qué hace, un comando listo para copiar y pegar, y etiquetas de uso**. Sin semántica inventada: las 30 entradas se **ejecutaron de verdad**, una por una, contra un árbol de archivos de prueba (dos logs, notas, fuentes en C y Python, un binario, un archivo con separador NUL, enlaces simbólicos, un nombre con espacios) en GNU grep 3.11, y la semántica se verificó contra el manual de GNU Grep. Las diferencias entre GNU y BSD están señaladas al final de la hoja.

## Para quién es

Ingenieros de operaciones (SRE/DevOps), desarrolladores backend y cualquiera que tenga un log de millones de líneas y solo quiera saber cuáles fallaron, o que necesite saber exactamente qué devuelve `grep` dentro de un script. Si tus búsquedas siempre llegan tarde —la recursión se mete donde no debe, un binario solo responde «matches», la salida de una tubería no aparece—, esta hoja es para ti.

## Cómo usarla

1. Ve directo a **[CHEATSHEET.md](CHEATSHEET.md)**: el cuerpo de la referencia; los comandos en inglés los entiende cualquier desarrollador.
2. Identifica primero tu grep: `grep --version` dice GNU grep y su versión en la primera línea; macOS trae el grep de BSD, que no tiene `-P`. Las 30 entradas están verificadas en GNU grep 3.11 y las notas de implementación del final son lectura obligada.
3. Sigue el orden de la hoja —conceptos básicos → patrones → contexto y salida → tuberías y scripts— y prueba cada comando a mano sobre archivos de práctica. En una hora los dominas.

## Pruébalo en 30 segundos

```bash
grep --version | head -1               # confirma la implementación: GNU grep y versión
mkdir -p /tmp/proj && printf 'INFO ok\nERROR disk full\n' > /tmp/proj/app.log
grep -c 'ERROR' /tmp/proj/app.log      # 1: el conteo de líneas que coinciden, el gesto clásico de grep
```

## La combinación de oro, lista para usar

```bash
grep -rn --include='*.log' 'ERROR' proj
```

Recursión + números de línea + solo archivos de log: todas las líneas ERROR de todos los logs del árbol, con archivo y línea, sin tocar fuentes ni binarios. El primer comando de cualquier postmortem suele ser este.

## Qué incluye

| Categoría | Contenido |
|---|---|
| 🧱 Conceptos básicos | búsqueda simple / -i sin distinción de mayúsculas / -v invertir / -c contar / -n números de línea / -l y -L listas de archivos / -r recursión / -R siguiendo enlaces / --include, --exclude y --exclude-dir (9) |
| 🎯 Patrones | -E regex extendida y la trampa de las barras en BRE / -F cadena literal / -w palabras completas / -x línea completa / -e varios patrones / -f archivo de patrones / anclas ^ y $ (7) |
| 🖍️ Contexto y salida | -A después / -B antes / -C ambos y la fusión de grupos / --color / -o solo la parte coincidente / -H y -h el prefijo de archivo (6) |
| 🔧 Tuberías y scripts | -q y el contrato de salida 0/1/2 / -s silenciar errores / -m parar pronto / el trío binario (aviso, -a, -I) / -z registros NUL y coincidencias entre líneas / --line-buffered / -P PCRE con lookarounds / patrones que empiezan por - (8) |

La documentación oficial enlazada es solo https://www.gnu.org/software/grep/manual/; todos los comandos se verificaron contra el manual de GNU Grep y se ejecutaron con éxito en GNU grep 3.11; los ejemplos operan únicamente sobre archivos de prueba creados al momento, sin datos reales.

## Por qué estas 30

- **El contrato de códigos de salida tenía que estar**: 0 = hay coincidencia, 1 = no hay, 2 = error. La mitad de los accidentes con grep en scripts viene de tratar el 1 como si fuera un error, o de no esperar que `-c` imprima 0 saliendo con código 1. La hoja lo fija por escrito y lo mide.
- **-r y -R difieren en una letra y mucho en comportamiento**: `-r` solo sigue los enlaces simbólicos nombrados en la línea de comandos y salta los que encuentra dentro del árbol; `-R` los sigue todos. Sin saberlo, una búsqueda recursiva omite archivos en silencio.
- **El «falso negativo» de los binarios, señalado**: la línea coincidente se suprime y solo queda un aviso en stderr (`binary file matches`); quien solo mira stdout cree que no había nada. `-a` fuerza la línea e `-I` salta el archivo: las dos salidas correctas están dadas.
- **-z tiene un requisito invisible para cruzar líneas**: el salto de línea del patrón debe ser real (comillas `$'…'` de bash); un `\n` entre comillas normales no hace nada. Medido antes de escribirlo.

## Se aceptan PR

¿Un comando incorrecto, una opción obsoleta, una traducción mejorable? Los Pull Requests son bienvenidos. Mantén el formato de las entradas (una línea de explicación / bloque bash / etiquetas) y **solo comandos reales y verificados; ante la duda, déjalo fuera**.

## 📚 Más hojas de referencia

Otras hojas del mismo autor, también en chino / español / portugués:

- [Hoja de SSH](https://github.com/Ezra-Zhao/ssh-cheatsheet) — 28 comandos de conexión, claves, túneles y endurecimiento
- [Hoja de FFmpeg](https://github.com/Ezra-Zhao/ffmpeg-cheatsheet) — 28 comandos de audio y vídeo
- [Hoja de Git](https://github.com/Ezra-Zhao/git-cheatsheet) — 30 comandos diarios y de rescate
- [Hoja de Docker](https://github.com/Ezra-Zhao/docker-cheatsheet) — 28 comandos de contenedores
- [Hoja de curl](https://github.com/Ezra-Zhao/curl-cheatsheet) — 29 comandos para depurar APIs
- [Hoja de expresiones regulares](https://github.com/Ezra-Zhao/regex-cheatsheet) — 30 patrones prácticos
- [Hoja de jq](https://github.com/Ezra-Zhao/jq-cheatsheet) — 30 filtros para JSON
- [Hoja de superpoderes de terminal](https://github.com/Ezra-Zhao/terminal-superpowers-cheatsheet) — 14 herramientas de productividad
- [Hoja de find + xargs](https://github.com/Ezra-Zhao/find-xargs-cheatsheet) — 30 comandos para buscar archivos y actuar en lote
- [Hoja de awk](https://github.com/Ezra-Zhao/awk-cheatsheet) — 30 comandos de campos, totales y unión de archivos
- [Hoja de kubectl](https://github.com/Ezra-Zhao/kubectl-cheatsheet) — comandos habituales de Kubernetes
- [Hoja de SQL](https://github.com/Ezra-Zhao/sql-cheatsheet) — consultas y funciones habituales
- [Hoja de Vim](https://github.com/Ezra-Zhao/vim-cheatsheet) — 30 comandos de edición, búsqueda y macros
- [Hoja de tmux](https://github.com/Ezra-Zhao/tmux-cheatsheet) — 30 comandos de sesiones, ventanas y paneles
- [Hoja de systemd](https://github.com/Ezra-Zhao/systemd-cheatsheet) — 30 comandos de servicios y registros

## Licencia

MIT © 2026 Guangyi Zhao. Véase [LICENSE](LICENSE).
