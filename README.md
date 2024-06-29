# INDICE-DE-LA-HETEROGENEIDAD-DE-LOS-USOS-DEL-SUELO
En este repositorio se incluyen los códigos para el cálculo del índice de heterogeniedad de los usos del suelo para los municpios de Don Benito - Villanueva de la Serena, Badajoz.

-*- coding: utf-8 -*-

File: CÓDIGO PYTHON_TFM_DIEGO_MURILLO_VILLAR.py
Author: Diego Murillo Villar

Date: 2 de julio de 2024

Description: Códigos sobre el cálculo de índice de heterogeneidad en los municipios de Don Benito - Villanueva de la Serena.

## Importación de librerias

        import geopandas as gpd
        import matplotlib.pyplot as plt
        import numpy as np
        import pandas as pd
        import os
## Rutas de los archivos shapefile

        don_benito_path = r"D:\TFM\CC\donbenito.shp"
        villanueva_path = r"D:\TFM\CC\villanueva.shp"

## Leer los archivos shapefile

        gdf_don_benito = gpd.read_file(don_benito_path)
        gdf_villanueva = gpd.read_file(villanueva_path)

## Mapeo de valores de 'USO' a nombres completos

        uso_map = {
        'IND': 'Industrial', 'EQUIP': 'Equipamientos', 'RES': 'Residencial',
        'ED_SING': 'Edificaciones singulares', 'COM_OFI': 'Comercio y oficinas',
        'OC_RES': 'Ocio y restauración', 'SIN_EDI': 'Sin Edificación', 'ZON_VER': 'Zonas Verdes'
    }

## Definir una función para mapear los códigos de uso del suelo a nombres completos

        def map_uso(uso):
                return uso_map.get(uso, 'Desconocido')

## Filtrar por municipio y contar el número de parcelas para Don Benito

        total_parcelas_don_benito = len(gdf_don_benito['ref_catast'].unique())

## Contar el número de parcelas por cada categoría de uso del suelo en Don Benito

        uso_parcelas_don_benito = (gdf_don_benito.groupby('USO').size()
                                   .reset_index(name='numero_parcelas'))
        uso_parcelas_don_benito['USO'] = uso_parcelas_don_benito['USO'].apply(map_uso)

## Filtrar por municipio y contar el número de parcelas para Villanueva de la Serena

        total_parcelas_villanueva = len(gdf_villanueva['ref_catast'].unique())

## Contar el número de parcelas por cada categoría de uso del suelo en Villanueva de la Serena

        uso_parcelas_villanueva = (gdf_villanueva.groupby('USO').size()
                                   .reset_index(name='numero_parcelas'))
        uso_parcelas_villanueva['USO'] = uso_parcelas_villanueva['USO'].apply(map_uso)

## Imprimir los resultados

        print("Información para Don Benito:")
        print("Número total de parcelas:", total_parcelas_don_benito)
        print("\nNúmero de parcelas por categoría de uso del suelo:")
        print(uso_parcelas_don_benito)
        
        print("\nInformación para Villanueva de la Serena:")
        print("Número total de parcelas:", total_parcelas_villanueva)
        print("\nNúmero de parcelas por categoría de uso del suelo:")
        print(uso_parcelas_villanueva)


# GRÁFICO DE BARRAS

## Configurar el tamaño del gráfico

        plt.figure(figsize=(10, 6))

## Configurar el ancho de las barras

        bar_width = 0.35
        index = np.arange(len(uso_parcelas_don_benito))

## Crear gráfico para Don Benito

        plt.bar(index, uso_parcelas_don_benito['numero_parcelas'], bar_width,
                color='skyblue', label='Don Benito')

## Crear gráfico para Villanueva de la Serena

        plt.bar(index + bar_width, uso_parcelas_villanueva['numero_parcelas'], bar_width,
                color='lightgreen', label='Villanueva de la Serena')

## Configurar etiquetas y título

        plt.xlabel('Categoría de Uso del Suelo')
        plt.ylabel('Número de Parcelas')
        plt.title('Número de Parcelas por Categoría de Uso del Suelo')
        plt.xticks(index + bar_width / 2, uso_parcelas_don_benito['USO'], rotation=45)
        plt.legend()

## Configurar escala logarítmica en el eje y

        plt.yscale('log')

## Configurar marcas de graduación en el eje y

        plt.yticks([1, 10, 100, 1000, 10000], ['1', '10', '100', '1000', '10000'])

## Configurar formato de los números en el eje y
        plt.gca().yaxis.set_major_formatter(plt.FuncFormatter(lambda x, _: '{:.0f}'.format(x)))

## Mostrar el gráfico

        plt.tight_layout()
        plt.show()

# GRÁFICO DE SECTORES

## Rutas de los archivos shapefile de los municipios y buffers

        don_benito_buffer_path = r"D:\TFM\CC\donbenito_buffer.shp"
        villanueva_buffer_path = r"D:\TFM\CC\villanueva_buffer.shp"

## Leer los archivos shapefile de los buffers
        
        buffer_don_benito = gpd.read_file(don_benito_buffer_path)
        buffer_villanueva = gpd.read_file(villanueva_buffer_path)

## Intersectar parcelas y buffers

        intersect_don_benito = gpd.overlay(gdf_don_benito, buffer_don_benito, how='intersection')
        intersect_villanueva = gpd.overlay(gdf_villanueva, buffer_villanueva, how='intersection')

## Calcular el área de cada intersección

        intersect_don_benito['area_interseccion'] = intersect_don_benito.geometry.area
        intersect_villanueva['area_interseccion'] = intersect_villanueva.geometry.area

## Calcular el área total de cada buffer

        area_buffer_don_benito = buffer_don_benito.geometry.area.iloc[0]
        area_buffer_villanueva = buffer_villanueva.geometry.area.iloc[0]

## Calcular el área total por uso del suelo en cada municipio

        area_total_don_benito = intersect_don_benito.groupby('USO')['area_interseccion'].sum()
        area_total_villanueva = intersect_villanueva.groupby('USO')['area_interseccion'].sum()

## Calcular el porcentaje de área por uso del suelo en cada municipio

        porcentaje_area_don_benito = (area_total_don_benito / area_buffer_don_benito) * 100
        porcentaje_area_villanueva = (area_total_villanueva / area_buffer_villanueva) * 100

## Reemplazar NaN con "SIN_EDIF" para representar "Sin Edificación"

        porcentaje_area_don_benito = porcentaje_area_don_benito.fillna('SIN_EDIF')
        porcentaje_area_villanueva = porcentaje_area_villanueva.fillna('SIN_EDIF')

## Renombrar índices usando el mapeo

        porcentaje_area_don_benito.index = porcentaje_area_don_benito.index.map(lambda x: uso_map.get(x, x))
        porcentaje_area_villanueva.index = porcentaje_area_villanueva.index.map(lambda x: uso_map.get(x, x))

## Crear el gráfico de pastel para Don Benito

        plt.figure(figsize=(12, 6))
        
        plt.subplot(1, 2, 1)
        plt.pie(porcentaje_area_don_benito, labels=porcentaje_area_don_benito.index, autopct='%1.1f%%', startangle=140, pctdistance=0.85)
        plt.title('Don Benito')

## Crear el gráfico de pastel para Villanueva de la Serena

        plt.subplot(1, 2, 2)
        plt.pie(porcentaje_area_villanueva, labels=porcentaje_area_villanueva.index, autopct='%1.1f%%', startangle=140, pctdistance=0.85)
        plt.title('Villanueva de la Serena')

## Ajustar layout para evitar superposiciones

        plt.tight_layout()

## Mostrar el gráfico

        plt.show()

# Creación de Buffers para Parcelas

## Rutas de los archivos shapefile
        
        buffer_parcelas_fp = "D:/TFM/buffer_parcelas.shp"
        clasificacion_fp = "D:/TFM/CC_donbenito/clasificación_donbenito.shp"

## Tamaño del buffer en metros

        buffer_size = 300

## Cargar los GeoDataFrames

        buffer_parcelas = gpd.read_file(buffer_parcelas_fp)
        clasificacion = gpd.read_file(clasificacion_fp)

## Asegurar que ambos GeoDataFrames tienen el mismo sistema de referencia

        if buffer_parcelas.crs != clasificacion.crs:
            clasificacion = clasificacion.to_crs(buffer_parcelas.crs)

## Realizar la intersección espacial

        interseccion = gpd.overlay(buffer_parcelas, clasificacion, how='intersection')

## Verificar la presencia de la columna 'USO'

        if 'USO' not in interseccion.columns:
            raise KeyError("La columna 'USO' no está presente en el GeoDataFrame resultante de la intersección.")

## Calcular el número de parcelas dentro del buffer

        num_parcelas = interseccion.shape[0]

## Agrupar por uso y contar el número de parcelas por uso

        uso_counts = interseccion.groupby('USO').size()

## Calcular el área total por uso en m²

        interseccion['area_m2'] = interseccion['geometry'].area
        area_por_uso = interseccion.groupby('USO')['area_m2'].sum()

## Crear un DataFrame con los resultados

        resultados = pd.DataFrame({
            'Número de Parcelas': uso_counts,
            'Área Total (m²)': area_por_uso
        })

# Cálculo de Áreas

## Mostrar la tabla de resultados en formato de texto
        
        print(resultados.to_string())


## Función para calcular el área de intersección y área total del buffer

        def calcular_areas(gdf_municipio, gdf_buffer):
            # Intersectar parcelas y buffers
            intersect = gpd.overlay(gdf_municipio, gdf_buffer, how='intersection')
        
            # Calcular el área de cada intersección
            intersect['area_interseccion'] = intersect.geometry.area
        
            # Calcular el área total de cada buffer
            area_buffer = gdf_buffer.geometry.area.iloc[0]
        
            return intersect, area_buffer

## Rutas de los archivos shapefile de los municipios y buffers

        don_benito_path = r"D:\TFM\CC\donbenito.shp"
        villanueva_path = r"D:\TFM\CC\villanueva.shp"
        don_benito_buffer_path = r"D:\TFM\CC\donbenito_buffer.shp"
        villanueva_buffer_path = r"D:\TFM\CC\villanueva_buffer.shp"

## Leer los archivos shapefile de los municipios y buffers

        gdf_don_benito = gpd.read_file(don_benito_path)
        gdf_villanueva = gpd.read_file(villanueva_path)
        buffer_don_benito = gpd.read_file(don_benito_buffer_path)
        buffer_villanueva = gpd.read_file(villanueva_buffer_path)

## Calcular áreas para Don Benito

        intersect_don_benito, area_buffer_don_benito = calcular_areas(gdf_don_benito, buffer_don_benito)
        area_total_don_benito = intersect_don_benito.groupby('USO')['area_interseccion'].sum()

## Calcular áreas para Villanueva de la Serena

        intersect_villanueva, area_buffer_villanueva = calcular_areas(gdf_villanueva, buffer_villanueva)
        area_total_villanueva = intersect_villanueva.groupby('USO')['area_interseccion'].sum()

# Cálculo del Índice de Heterogeneidad para Parcelas

## Función para calcular la entropía por parcela

        def calcular_entropia_por_parcela(data, uso_col):
            total_area = data['area_parcela'].sum()
            data['area'] = data['area_parcela'] / total_area
            data['entropia'] = -data['area'] * np.log(data['area'])
            max_entropia = data['entropia'].max()
            data['entropia'] = data['entropia'] / max_entropia
            return data

## Rutas de los archivos shapefile de los municipios

        don_benito_path = r"D:\TFM\CC\donbenito.shp"
        villanueva_path = r"D:\TFM\CC\villanueva.shp"

## Leer los archivos shapefile de los municipios

        gdf_don_benito = gpd.read_file(don_benito_path)
        gdf_villanueva = gpd.read_file(villanueva_path)

# Asegurarse de que ambos GeoDataFrames tengan el mismo CRS

        crs = "EPSG:25830"  # Este es el CRS ETRS89
        gdf_don_benito = gdf_don_benito.to_crs(crs)
        gdf_villanueva = gdf_villanueva.to_crs(crs)

## Asumiendo que el nombre de la columna de uso es 'USO', actualízalo si es diferente

        uso_col = 'USO'  # Actualiza esto con el nombre correcto de la columna de uso

## Calcular el área de cada parcela

        gdf_don_benito['area_parcela'] = gdf_don_benito.geometry.area
        gdf_villanueva['area_parcela'] = gdf_villanueva.geometry.area

## Calcular la entropía para Don Benito

        gdf_don_benito = calcular_entropia_por_parcela(gdf_don_benito, uso_col)

## Calcular la entropía para Villanueva de la Serena

        gdf_villanueva = calcular_entropia_por_parcela(gdf_villanueva, uso_col)

## Mostrar los resultados de la entropía 

        print("Índices de Entropía para Don Benito:")
        print(gdf_don_benito[[uso_col, 'entropia']])
        
        print("\nÍndices de Entropía para Villanueva de la Serena:")
        print(gdf_villanueva[[uso_col, 'entropia']])

## Guardar los resultados en archivos shapefile

        output_folder = r"C:\Users\dmurillov\Documents\GEO"
        output_filename_don_benito = os.path.join(output_folder, "entropia_don_benito.shp")
        output_filename_villanueva = os.path.join(output_folder, "entropia_villanueva.shp")
        
        gdf_don_benito.to_file(output_filename_don_benito)
        gdf_villanueva.to_file(output_filename_villanueva)
        
        print("Archivos shapefile guardados correctamente en la carpeta:", output_folder)

# Cálculo del Índice de Heterogeneidad para Manzanas

## Función para calcular la entropía por manzana

        def calcular_entropia(row):
            prob_area = row['area_parce'] / manzanas_agrupadas['area_parce'].sum()
            entropia = -(prob_area * np.log(prob_area)) if prob_area > 0 else 0
            return entropia

## Ruta del archivo Shapefile de entrada y salida

        ruta_entrada = r"D:\TFM\CARTOCIUDAD_CALLEJERO_BADAJOZ\CARTOCIUDAD_CALLEJERO_BADAJOZ\manzanas.shp"
        ruta_salida = r"D:\TFM\CARTOCIUDAD_CALLEJERO_BADAJOZ\CARTOCIUDAD_CALLEJERO_BADAJOZ\manzanas_con_entropia.shp"

## Cargar el GeoDataFrame con las manzanas

        manzanas_agrupadas = gpd.read_file(ruta_entrada)

## Calcular la entropía para cada manzana

        manzanas_agrupadas['entropia_uso'] = manzanas_agrupadas.apply(calcular_entropia, axis=1)

## Guardar el GeoDataFrame con la información de entropía 

        manzanas_agrupadas.to_file(ruta_salida)
        
        print("El archivo Shapefile con la entropía se ha guardado correctamente.")
