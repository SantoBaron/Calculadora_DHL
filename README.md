# Calculadora DHL — Prototipo JSON

Desarrollo experimental de un cotizador HTML/JavaScript sin backend ni persistencia, usando las tarifas originales de DHL como fuente.

## Tarifas de trabajo
- DHL Export · cuenta 301913599
- DHL Import · cuenta 958289452
- DHL Heavy Export · cuenta 313041246

## Arquitectura prevista
- Datos JSON estáticos, normalizados por perfil tarifario.
- Zonificación local/internacional y matriz independientes por tarifa.
- Variantes según sobre, despacho aduanero y límites de peso.
- Precios explícitos, reglas incrementales ancladas y suplementos separados.
- Registro de procedencia de cada dato para comprobación contra Excel.
- Preparado para migrar el almacenamiento a tablas SQL.

La especificación técnica está en [docs/modelo-json-v1.md](docs/modelo-json-v1.md).

**Estado:** especificación inicial. Los JSON completos y el motor de cálculo aún no están implementados; no deben mostrarse cotizaciones hasta su validación con los Excel.
