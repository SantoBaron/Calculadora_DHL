# Modelo JSON v1 — contrato inicial

Estado: **diseño preliminar** pendiente de validación íntegra frente a los dos archivos Excel.

## Fuentes

- `EXPORT 301913599 - IMPORT 958289452.xlsx`: 8 hojas (Export, Import, zonas por dirección, matriz, servicios y suplementos, condiciones y suplemento por demanda).
- `HEAVY–313041246-EXPORT.xlsx`: 6 hojas (Export, zonas, matriz, servicios y suplementos, condiciones y suplemento por demanda).

## Entidades / futuros archivos

| Entidad SQL prevista | JSON | Clave |
|---|---|---|
| `tariff_profiles` | `data/tariffs.json` | `tariffId` |
| `tariff_geographic_zones` | `data/zones.json` | `tariffId + direction + scope + countryCode + qualifier` |
| `tariff_zone_matrix` | `data/zone-matrices.json` | `tariffId + localZoneCode + internationalZoneCode` |
| `tariff_product_variants` | `data/product-variants.json` | `tariffId + variantId` |
| `tariff_base_rates` | `data/rates.json` | `tariffId + variantId + priceZone + weightUpperKg` |
| `tariff_increment_rules` | `data/weight-rules.json` | `tariffId + variantId + priceZone + fromWeightExclusiveKg` |
| `tariff_surcharges` | `data/surcharges.json` | `tariffId + surchargeCode + conditions` |

Todos los documentos tendrán `schemaVersion: 1`. Todos los registros importados conservarán `sourceFile`, `sourceSheet` y `sourceRow` o `sourceRange`.

## Perfil de ejemplo (metadatos, no tarifa validada)
```json
{
  "schemaVersion": 1,
  "tariffId": "DHL_ES_301913599_EXPORT_2026",
  "carrier": "DHL",
  "accountNumber": "301913599",
  "service": "EXPRESS_WORLDWIDE",
  "direction": "export",
  "currency": "EUR",
  "validFrom": null,
  "validTo": null,
  "status": "pending_validation"
}
```

Las fechas de vigencia se fijarán tras verificar el texto literal del documento.

## Campos críticos

- Zonas: `countryCode`, `scope` (`local`/`international`), `zoneCode`, `qualifier`. El calificador conserva excepciones territoriales o contractuales sin fusionarlas en un ISO2.
- Matriz: `localZoneCode`, `internationalZoneCode`, `priceZone`.
- Variante: `variantId`, `productType`, `customsTreatment`, `minWeightKg`, `maxWeightKg`. No asignar una condición aduanera si el Excel no la demuestra.
- Precio base: `variantId`, `priceZone`, `weightUpperKg`, `priceCents` (entero).
- Incrementos: `fromWeightExclusiveKg`, `toWeightInclusiveKg`, `stepKg`, `incrementCents`, `anchorWeightKg`, `anchorPriceCents`. No interpolar sin regla comprobada.
- Suplemento: `calculationType`, `conditions`, `value`, `status`. Valores externos variables: `external_rate_required`.

## Secuencia del motor

1. Elegir perfil según cuenta, dirección, servicio y vigencia.
2. Resolver las dos zonas geográficas y el cruce de la matriz específica de esa tarifa.
3. Determinar variante según tipo de envío y tratamiento aduanero.
4. Calcular peso facturable con las condiciones documentadas.
5. Localizar precio explícito o regla incremental válida con ancla verificada.
6. Devolver precio base y explicación; suplementos por separado.

Si falta información devolver estado `unquotable`, sin inventar precio.

## Criterios de aceptación de la conversión

1. Cobertura de las tres tarifas y de sus mapas, matrices y variantes.
2. Ausencia de duplicados o cruces ambiguos sin calificador.
3. Tarifas y recargos separados; euros expresados en céntimos enteros.
4. Reproducción exacta de precios publicados y de transiciones entre tramos.
5. Informar de inconsistencias con procedencia exacta (archivo, hoja, fila).
6. No habilitar el cotizador hasta aprobar las comprobaciones anteriores.

## Hallazgos importantes

Ambos libros distinguen «Envíos sin trámite aduanero hasta 2.0 KG» y «Envíos con trámite aduanero desde 0.5 KG & Envíos sin trámite aduanero desde 2.5 KG». Las tablas de exportación incluyen un bloque específico «Sobre hasta 300 g sólo». Los mapeos geográficos y las matrices no son intercambiables entre cuentas.

Los JSON finales se producirán por extracción verificable desde Excel; este documento **no supone que la conversión ya esté realizada**.
