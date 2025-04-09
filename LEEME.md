---

# 👞😭 Proyecto de Análisis de Despidos Globales 🔍

*Donde comenzó la decadencia de los trabajos en TI...*

---

## Tabla de Contenidos

- [Introducción](#introducción)
- [Proceso de Limpieza de Datos](#proceso-de-limpieza-de-datos)
- [Análisis Exploratorio de Datos (EDA)](#análisis-exploratorio-de-datos-eda)
- [Conclusiones y Recomendaciones](#conclusiones-y-recomendaciones)

---

## Introducción

![image](https://github.com/user-attachments/assets/85fecb70-58de-495c-abb0-ed8cda3c9123)

En este proyecto, se procesan datos en crudo sobre despidos globales a través de un sólido flujo de trabajo de limpieza de datos. Luego de la limpieza, se realiza un análisis exploratorio de datos (EDA) para revelar ideas clave sobre tendencias, distribución de despidos por empresa, industria, país e intervalos de tiempo. El proyecto simula una canalización de datos del mundo real, desde la ingesta y preparación de datos en bruto hasta una etapa de informes analíticos refinados.

![image](https://github.com/user-attachments/assets/9138e9d0-c38f-4d73-be7b-1e5dd0fc9076)

---

## Proceso de Limpieza de Datos

🧹 Esta sección resume los pasos seguidos para limpiar y estandarizar el conjunto de datos:

1. **📦 Configuración de Base de Datos y Tablas:**  
   - Inicializar MySQL Workbench y crear un nuevo esquema (llamado `world_layoffs`).
   - Usar el asistente de importación de datos para cargar el dataset en una tabla de trabajo.
   - Crear una tabla staging duplicada para trabajar sin modificar los datos originales.

   ![image](https://github.com/user-attachments/assets/b7bab74b-988b-4533-ad88-47876c1f9027)

2. **🚫 Eliminación de Duplicados:**  
   - Ejecutar consultas utilizando `ROW_NUMBER() OVER (...)` particionando por columnas clave (empresa, ubicación, etc.) para identificar registros duplicados.

   ![image](https://github.com/user-attachments/assets/0dd87dc0-efcd-45e9-a856-32c35c7a06af)
   *(Detección de duplicados)*

   - Aplicar una Expresión de Tabla Común (CTE) para detectar duplicados y luego eliminar las filas con `row_num > 1`.

   *Primero creamos una segunda tabla staging:*
   ```sql
   CREATE TABLE `layoffs_staging2` (
    `company` text,
    `location` text,
    `industry` text,
    `total_laid_off` int DEFAULT NULL,
    `percentage_laid_off` text,
    `date` text,
    `stage` text,
    `country` text,
    `funds_raised_millions` int DEFAULT NULL,
    `row_num` int
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
   ```

   *Luego copiamos los datos con el número de fila:*
   ```sql
   INSERT INTO layoffs_staging2
   SELECT *,
   ROW_NUMBER() OVER(
   PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
   ) AS row_num
   FROM layoffs_staging;
   ```

   *Y finalmente, eliminamos los duplicados:*
   ```sql
   DELETE FROM layoffs_staging2
   WHERE row_num > 1;
   ```

3. **💾 Estandarización de Datos:**
   - Recortar espacios y corregir inconsistencias en los campos de texto (por ejemplo, estandarizar versiones de "Crypto", corregir signos de puntuación en nombres de países).

   ```sql
   UPDATE layoffs_staging2
   SET company = TRIM(company);
   ```

   ```sql
   UPDATE layoffs_staging2
   SET industry = 'Crypto'
   WHERE industry LIKE 'Crypto%';
   ```

   ```sql
   UPDATE layoffs_staging2
   SET country = TRIM(TRAILING '.' FROM country)
   WHERE country LIKE 'United States%';
   ```

   - Convertir cadenas de texto de fechas a un formato de fecha adecuado utilizando `STR_TO_DATE`.

   ```sql
   SELECT `date`,
   STR_TO_DATE(`date`, '%d/%m/%Y')
   FROM layoffs_staging2;
   ```

   ```sql
   UPDATE layoffs_staging2
   SET `date` = STR_TO_DATE(`date`, '%d/%m/%Y');
   ```

5. **💬 Manejo de Valores Nulos y Vacíos:**

   - Identificar valores NULL o vacíos en columnas clave.

   ```sql
   SELECT *
   FROM layoffs_staging2
   WHERE total_laid_off IS NULL;
   ```

   ```sql
   SELECT DISTINCT industry
   FROM layoffs_staging2;
   ```

   ```sql
   SELECT *
   FROM layoffs_staging2
   WHERE industry IS NULL OR industry = '';
   ```

   ```sql
   SELECT *
   FROM layoffs_staging2
   WHERE company = 'Airbnb';
   ```

   - Convertir valores vacíos en NULL para su manipulación.

   ```sql
   UPDATE layoffs_staging2
   SET industry = NULL
   WHERE industry = '';
   ```

   - Completar registros faltantes usando valores de registros duplicados.

   ```sql
   UPDATE layoffs_staging2 t1
   JOIN layoffs_staging2 t2
   ON t1.company = t2.company
   SET t1.industry = t2.industry
   WHERE (t1.industry IS NULL OR t1.industry = '')
   AND t2.industry IS NOT NULL;
   ```

   - Eliminar registros sin valor informativo (por ejemplo, si `total_laid_off` y `percentage_laid_off` son NULL).

   ```sql
   DELETE
   FROM layoffs_staging2
   WHERE total_laid_off IS NULL
   AND percentage_laid_off IS NULL;
   ```

   *Así quedó la tabla tras la limpieza:*
   ![image](https://github.com/user-attachments/assets/4a344406-0188-4022-a303-6bc79ef4f8c6)

7. **🗑️ Eliminación de Columnas Innecesarias:**  
   - Eliminar columnas auxiliares como `row_num` para simplificar el dataset final.

   ```sql
   ALTER TABLE layoffs_staging2
   DROP COLUMN row_num;
   ```

   - Verificar el conjunto de datos limpio con una consulta final:

   ```sql
   SELECT * FROM layoffs_staging2;
   ```

   ![image](https://github.com/user-attachments/assets/e86d0e3e-0aca-4505-93f5-1008ba830441)

---

## Análisis Exploratorio de Datos (EDA)

🔎 Con el conjunto de datos limpio, se procede al análisis exploratorio para extraer información útil:

1. **👓 Examen Inicial de Datos:**

   ```sql
   SELECT * FROM layoffs_staging2;
   ```

   ```sql
   SELECT MAX(total_laid_off), MAX(percentage_laid_off)
   FROM layoffs_staging2;
   ```

   ![image](https://github.com/user-attachments/assets/0f136a01-aa41-46e2-b8d7-cea9e4097e78)

   ```sql
   SELECT *
   FROM layoffs_staging2
   WHERE percentage_laid_off = 1
   ORDER BY total_laid_off DESC;
   ```

   ![image](https://github.com/user-attachments/assets/f25dec4a-3ce4-454e-b658-e248c599c13b)

   ```sql
   SELECT *
   FROM layoffs_staging2
   WHERE percentage_laid_off = 1
   ORDER BY funds_raised_millions DESC;
   ```

   ![image](https://github.com/user-attachments/assets/3ab03cda-081b-4f74-b7ea-7d2c5f59d324)

2. **🏢 Análisis por Empresa:**

   ```sql
   SELECT company, SUM(total_laid_off)
   FROM layoffs_staging2
   GROUP BY company
   ORDER BY 2 DESC;
   ```

   ![image](https://github.com/user-attachments/assets/36e1323f-6189-4729-be00-55db90b8d1bd)

   ```sql
   SELECT company, percentage_laid_off, total_laid_off
   FROM layoffs_staging2
   WHERE percentage_laid_off = 1
   ORDER BY total_laid_off DESC;
   ```

   ![image](https://github.com/user-attachments/assets/6fe3cf8d-e366-4fd4-aa22-dc27820babaf)

3. **🏭 Análisis por Industria:**

   ```sql
   SELECT industry, SUM(total_laid_off)
   FROM layoffs_staging2
   GROUP BY industry
   ORDER BY 2 DESC;
   ```

   ![image](https://github.com/user-attachments/assets/be47dbc3-91e4-4382-8426-b25382688015)

4. **🌎 Análisis por País:**

   ```sql
   SELECT country, SUM(total_laid_off)
   FROM layoffs_staging2
   GROUP BY country
   ORDER BY 2 DESC;
   ```

   ![image](https://github.com/user-attachments/assets/e2460e9f-7d4e-4852-8faa-20e285e54304)

5. **📆 Análisis Temporal:**

   ```sql
   SELECT YEAR(`date`), SUM(total_laid_off)
   FROM layoffs_staging2
   WHERE YEAR(`date`) IS NOT NULL
   GROUP BY YEAR(`date`)
   ORDER BY 1 DESC;
   ```

   ![image](https://github.com/user-attachments/assets/f66fc7e8-ca90-4f33-8255-1b3b712dc5b9)

   ```sql
   WITH Rolling_Total AS (
    SELECT SUBSTRING(`date`, 1, 7) AS `MONTH`, SUM(total_laid_off) AS total_off
    FROM layoffs_staging2
    WHERE SUBSTRING(`date`, 1, 7) IS NOT NULL
    GROUP BY `MONTH`
    ORDER BY 1 ASC
   )
   SELECT `MONTH`, total_off, SUM(total_off) OVER(ORDER BY `MONTH`) AS rolling_total 
   FROM Rolling_Total;
   ```

   ![image](https://github.com/user-attachments/assets/937ade30-e290-490f-9cc7-4066f7370197)

6. **🛠️ Análisis Avanzado:**

   ```sql
   WITH Company_Year (company, years, total_laid_off) AS (
    SELECT company, YEAR(`date`), SUM(total_laid_off)
    FROM layoffs_staging2
    GROUP BY company, YEAR(`date`)
   )
   SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS Ranking
   FROM Company_Year
   WHERE years IS NOT NULL
   ORDER BY Ranking ASC;
   ```

   ![image](https://github.com/user-attachments/assets/ee26e7ca-4450-4b42-8ee1-48cc467a2a03)

   ```sql
   WITH Company_Year (company, years, total_laid_off) AS (
    SELECT company, YEAR(`date`), SUM(total_laid_off)
    FROM layoffs_staging2
    GROUP BY company, YEAR(`date`)
   ),
   Company_Year_Rank AS (
    SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS Ranking
    FROM Company_Year
    WHERE years IS NOT NULL
   )
   SELECT *
   FROM Company_Year_Rank
   WHERE Ranking <= 5;
   ```

   ![image](https://github.com/user-attachments/assets/3b07b606-1058-4e61-bb0e-5832209e726a)

---

## Conclusiones y Recomendaciones

📝 Después de completar el análisis exploratorio, el proyecto revela ideas clave como:

- La fluctuación de los despidos en el tiempo y la identificación de períodos críticos.
- Las empresas e industrias más afectadas por la reducción de personal.
- Tendencias geográficas que pueden informar investigaciones económicas más amplias.

**Siguientes pasos recomendados:**

- Integrar herramientas de visualización para comprender mejor las tendencias.
- Realizar segmentaciones más detalladas (por ejemplo, por tamaño de empresa o sector de mercado).
- Aplicar análisis estadístico profundo o modelos predictivos para una toma de decisiones proactiva.

--- 
