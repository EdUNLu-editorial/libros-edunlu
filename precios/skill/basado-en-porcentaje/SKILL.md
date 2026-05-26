# SKILL: basado-en-porcentaje

## Description
Aumenta automaticamente todos los precios en los archivos HTML del directorio `publicaciones/` aplicando un porcentaje configurable definido por el usuario.

## Configuracion
Antes de ejecutar, reemplaza el valor numerico con el porcentaje deseado:

**Porcentaje de aumento:** `___`

> El valor debe ser un numero entero o decimal positivo, sin el signo `%`.
> Ejemplo: `15` significa aumentar un 15% (factor 1.15).
> `7.5` significa aumentar un 7.5% (factor 1.075).

## Source and target
- **Target HTML files**: `publicaciones/*.html`
- Solo se modifican archivos cuyo tag `<p class="price">` contenga un precio (no vacio).

## Price tag format in HTML
```html
<p class="price" style="color: #04a04e;">Precio pre-compra online con 50% de descuento: $10000.00.-</p>
```

El valor monetario siempre tiene el formato `$XXXX.XX.-` (signo dolar, digitos, punto, dos decimales, punto, guion).

## Steps

### 1. Configurar el porcentaje
Leer el valor de `## Configuracion` en este mismo archivo `SKILL.md` y asignarlo a una variable `$porcentaje`.

### 2. Iterar HTML files en `publicaciones/`
Para cada archivo `.html`:
1. Leer el contenido del archivo (UTF8)
2. Buscar `<p class="price"` y extraer el valor monetario actual con regex
3. Si el tag esta vacio (sin precio), saltar el archivo
4. Convertir el valor a numero
5. Calcular `nuevo_valor = valor_actual * (1 + $porcentaje / 100)`
6. Redondear a 2 decimales
7. Formatear como `$XXXX.XX.-`
8. Reemplazar en el tag HTML
9. Sobreescribir el archivo

### 3. Regex para extraer el precio actual
```
Pattern: \$([\d,]+\.\d{2})\.\-
```
Captura el grupo numerico (ej: `10000.00`) desde `$10000.00.-`.

### 4. Regex para reemplazar el precio
```
Find:    (<p class="price"[^>]*>Precio pre-compra online con 50% de descuento: )\$[\d,]+\.\d{2}\.\-(</p>)
Replace: $1{new_price}$2
```

### 5. Write updated HTML
Sobreescribir el archivo HTML con el contenido modificado. Encoding UTF8 sin BOM.

## PowerShell implementation reference

```powershell
# === CONFIGURACION: reemplazar ___ con el porcentaje ===
$porcentaje = ___

$htmlDir = "publicaciones"

Get-ChildItem "$htmlDir/*.html" | ForEach-Object {
    $content = Get-Content $_.FullName -Raw -Encoding UTF8

    # Extract current price
    if ($content -match '<p class="price"[^>]*>Precio pre-compra online con 50% de descuento: \$([\d,]+\.\d{2})\.\-</p>') {
        $currentValue = [decimal]($Matches[1] -replace ',', '')
        $factor = 1 + ($porcentaje / 100)
        $newValue = [math]::Round($currentValue * $factor, 2)
        $newPrice = '$' + $newValue.ToString('F2', [System.Globalization.CultureInfo]::InvariantCulture) + '.-'

        $pattern = '(<p class="price"[^>]*>Precio pre-compra online con 50% de descuento: )\$[\d,]+\.\d{2}\.\-(</p>)'
        $replacement = "`$1$newPrice`$2"
        $content = $content -replace $pattern, $replacement

        Set-Content $_.FullName -Value $content -Encoding UTF8 -NoNewline
    }
}
```

## Edge cases
- Si el tag `<p class="price">` esta vacio, el archivo no se modifica (el regex no matchea)
- Si el porcentaje es `0`, los precios no cambian
- Si el resultado tiene mas de 2 decimales, se redondea con `[math]::Round(x, 2)`
- Los archivos se procesan uno por uno; si falla uno, los demas continuan
