# SKILL: precio-fijo

## Description
Actualiza los precios en los archivos HTML del directorio `publicaciones/` usando los valores definidos en `tabla-precios.xml`.

## Source data
- **XML**: `precios/skill/precio-fijo/tabla-precios.xml` (mismo directorio que este SKILL)
- **Target HTML files**: `publicaciones/*.html` (directorio raiz del proyecto)

## Mapping
El contenido de `<nombre>` en el XML corresponde al nombre del archivo HTML **sin la extensión `.html`**.

Ejemplo:
```
XML:   <nombre>el-interes-superior-del-nino</nombre>
HTML:  publicaciones/el-interes-superior-del-nino.html
```

## Price tag format in HTML
```html
<p class="price" style="color: #04a04e;">Precio pre-compra: $3500.00.-</p>
```

The price value is always in the format `$XXXX.XX.-` (dollar sign, digits, dot, two decimals, dot, dash).

## Steps

### 1. Parse `tabla-precios.xml`
Read the XML file and build a dictionary where:
- **Key**: `<nombre>` value
- **Value**: `<precio>` value (e.g. `$3500.00.-` or `-`)

Skip entries where `<precio>` equals `-` (no price to apply).

### 2. Iterate HTML files in `publicaciones/`
For each `.html` file:
1. Get the filename without `.html` extension (BaseName)
2. Look up the BaseName in the dictionary from step 1
3. If found, read the HTML file content
4. Replace the price inside `<p class="price">` using regex

### 3. Regex replacement
```
Find:    (<p class="price"[^>]*>Precio pre-compra: )\$[\d,]+\.\d{2}\.\-(</p>)
Replace: $1{new_price}$2
```
Where `{new_price}` is the value from the XML dictionary.

### 4. Write updated HTML
Overwrite the HTML file with the modified content. Use UTF8 encoding.

## PowerShell implementation reference
```powershell
$xmlPath = "precios/skill/precio-fijo/tabla-precios.xml"
$htmlDir = "publicaciones"

[xml]$xml = Get-Content $xmlPath -Encoding UTF8
$prices = @{}
$xml.libros.libro | ForEach-Object {
    if ($_.precio -ne '-') {
        $prices[$_.nombre] = $_.precio
    }
}

Get-ChildItem "$htmlDir/*.html" | ForEach-Object {
    $name = $_.BaseName
    if ($prices.ContainsKey($name)) {
        $content = Get-Content $_.FullName -Raw -Encoding UTF8
        $newPrice = $prices[$name]
        $pattern = '(<p class="price"[^>]*>Precio pre-compra: )\$[\d,]+\.\d{2}\.\-(</p>)'
        $replacement = "`$1$newPrice`$2"
        $content = $content -replace $pattern, $replacement
        Set-Content $_.FullName -Value $content -Encoding UTF8 -NoNewline
    }
}
```

## Edge cases
- If `<precio>` is `-` in XML, the corresponding HTML file is not modified
- If an HTML file has no matching entry in XML, it is left unchanged
- If the `<p class="price">` tag is empty in the HTML, it is left unchanged (regex won't match)
