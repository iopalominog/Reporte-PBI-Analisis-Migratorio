# Reportes

```sql
let
    Origen = Csv.Document(Web.Contents("https://raw.githubusercontent.com/iopalominog/Reportes/refs/heads/main/dataset_migraciones.csv"),[Delimiter=",", Columns=33, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    
    Encabezados = Table.PromoteHeaders(Origen),

    // 
    AsignacionDeTipos = 
        Table.TransformColumnTypes(
            Encabezados,
            {
                {"fecha_migracion", type date},
                {"fecha_nacimiento", type date},
                {"edad_al_migrar", Int64.Type},
                {"años_experiencia", Int64.Type},
                {"numero_hijos", Int64.Type},
                {"numero_hijos_acompañan", Int64.Type},
                {"numero_familiares_en_destino", Int64.Type},
                {"ingresos_pais_origen", Currency.Type},
                {"ingresos_pais_destino", Currency.Type},
                {"porcentaje_incremento_ingresos", Int64.Type},
                {"porcentaje_ingreso_remesas", Int64.Type},
                {"años_planeados_en_exterior", Int64.Type},
                {"satisfaccion_migracion", Int64.Type}
            }
        ),

    // Variación de ingresos origen y destino
    VariacionIngresos =
        Table.TransformColumnTypes(
            Table.AddColumn(
                AsignacionDeTipos,
                "variacion_ingresos_origen_destino",
                each [ingresos_pais_destino] - [ingresos_pais_origen]
            ),
            {"variacion_ingresos_origen_destino", Int64.Type}
        ),

    // Periodo de migracion
    PeriodoMigracion = 
        Table.TransformColumnTypes(
            Table.AddColumn(
                VariacionIngresos,
                "periodo_migracion",
                each 
                    let
                        Anio = Date.Year([fecha_migracion]),
                        Mes = Date.Month([fecha_migracion]),
                        Periodo = #date(Anio,Mes,1)
                    in
                        Periodo
            ),
            {"periodo_migracion", type date}
        ),

    // Personas totales que viajan
    PersonasTotales = 
        Table.TransformColumnTypes(
            Table.AddColumn(
                PeriodoMigracion,
                "personas_totales_que_migran",
                each [numero_hijos_acompañan] + 1
            ),
            {"personas_totales_que_migran", Int64.Type}
        ),

    // Etiqueta de satisfacción
    EtiquetaSatisfaccion = 
        Table.AddColumn(
            PersonasTotales,
            "etiqueta_satisfaccion_migracion",
            each 
                if 
                    [satisfaccion_migracion] < 3 then "Muy Insatisfecho"
                else if 
                    [satisfaccion_migracion] < 5 then "Insatisfecho"
                else if
                    [satisfaccion_migracion] < 7 then "Neutral"
                else if
                    [satisfaccion_migracion] < 9 then "Buena"
                else
                    "Excelente"
        )
in
    EtiquetaSatisfaccion
```
