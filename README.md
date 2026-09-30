[README.md](https://github.com/user-attachments/files/32880053/README.md)
# sQuAd1 · Automation Exercise (Web UI)

Proyecto de automatización de pruebas end-to-end para el sitio [automationexercise.com](https://automationexercise.com), desarrollado en **Java 21** con **Selenium WebDriver** y **Cucumber (BDD)**, con escenarios escritos en **español**. Los resultados se publican con **Allure Report** y la suite corre automáticamente en **GitHub Actions**.

## Alcance de la automatización

46 escenarios distribuidos en 6 módulos funcionales más un smoke test de navegación:

| Módulo | Feature | Tags principales | Escenarios |
|---|---|---|---|
| Registro de usuarios | `01_registro_usuario.feature` | `@Registro` | 8 |
| Autenticación y sesión | `02_autenticación_y_gestión_de_sesión.feature` | `@Autenticacion` | 8 |
| Productos y catálogo | `03_productos_y_catalogo.feature` | `@Catalogo` `@Busqueda` `@Filtros` | 14 |
| Carrito de compras | `04_carrito_de_compras.feature` | `@Carrito` | 5 |
| Checkout y pago | `05_proceso_de_checkout_y_pago.feature` | `@Pago` `@Checkout` | 6 |
| Formulario de contacto | `06_formulario_de_contacto.feature` | `@Contacto` | 4 |
| Smoke | `smoke_test.feature` | - | 1 |

Cada escenario lleva un tag de trazabilidad (`@TC-01`, `@TC-02`, ...) y tags de tipo (`@Positivo`, `@Negativo`, `@Smoke`, `@ValidacionCampos`, etc.). El listado funcional de casos está en `src/test/resources/features/listado de casos`.

## Stack tecnológico

| Herramienta | Versión | Uso |
|---|---|---|
| Java | 21 | Lenguaje |
| Maven | 3.9+ | Build y dependencias |
| Selenium WebDriver | 4.35.0 | Automatización del navegador |
| Cucumber JVM | 7.18.0 | BDD / Gherkin |
| JUnit 5 (Jupiter + Platform Suite) | 5.10.2 / 1.10.2 | Motor de ejecución |
| Allure (cucumber7-jvm) | 2.29.0 | Reportes |
| AspectJ Weaver | 1.9.22.1 | Instrumentación de Allure |
| SLF4J Simple | 2.0.13 | Logging |

## Requisitos previos

- **JDK 21** (`java -version`)
- **Maven 3.9+** (`mvn -version`)
- **Google Chrome** o **Microsoft Edge** instalado
- Conexión a internet

Selenium 4 incluye Selenium Manager, así que no hace falta descargar el driver del navegador a mano.

## Instalación

```bash
git clone https://github.com/jonalmlp/sQuAd1.git
cd sQuAd1
mvn clean install -DskipTests
```

## Ejecución

La ejecución pasa por una única clase, `runner.TestRunner`. Surefire está configurado para correr solo `**/TestRunner.java` y así evitar escenarios duplicados.

```bash
# Ejecutar la suite (Chrome, con ventana visible)
mvn clean test

# Elegir navegador: chrome (por defecto) o edge
mvn clean test -Dbrowser=edge

# Modo headless
mvn clean test -Dheadless=true
```

> En entornos de CI (variable `CI` definida, como en GitHub Actions) el navegador corre en headless automáticamente.

### Filtrar por tags

Los tags a ejecutar se definen en `src/test/java/runner/TestRunner.java`, en el parámetro `FILTER_TAGS_PROPERTY_NAME`. Por defecto corre todos los módulos funcionales:

```
@Pago or @Carrito or @Autenticacion or @Registro or @Catalogo or @Contacto
```

Para correr solo un subconjunto, editá esa expresión. Ejemplos:

```
@Smoke                      # solo los casos smoke (TC-01, TC-09, TC-32)
@Carrito and @Positivo      # casos positivos del carrito
@Catalogo and not @Filtros  # catálogo sin filtros
```

## Reportes

Durante la ejecución se generan:

- **Allure (resultados crudos):** `target/allure-results`
- **Cucumber HTML / JSON:** `target/cucumber-reports/cucumber.html` y `cucumber.json`
- **Evidencia de fallos:** capturas de pantalla y HTML de la página en `target/evidencia-fallos/` (se guardan automáticamente cuando un paso falla)

Para ver el reporte de Allure:

```bash
# Genera y abre el reporte en el navegador
mvn allure:serve

# Solo genera el reporte estático
mvn allure:report
```

## Estructura del proyecto

El proyecto sigue el patrón **Page Object Model**:

```
sQuAd1/
├── .github/workflows/
│   └── maven-tests.yml          # Pipeline de CI
├── .allure/                     # Allure CLI 2.29.0
├── src/
│   ├── main/java/pages/         # Page Objects
│   │   ├── BasePage.java        # Acciones comunes (clicks, esperas, selects)
│   │   ├── CommonPage.java
│   │   ├── RegisterPage.java
│   │   ├── LoginPage.java
│   │   ├── ProductosPage.java
│   │   ├── ProductoDetallePage.java
│   │   ├── CarritoPage.java
│   │   ├── CheckoutPage.java
│   │   └── ContactoPage.java
│   └── test/
│       ├── java/
│       │   ├── runner/TestRunner.java   # Suite JUnit Platform + config de Cucumber
│       │   ├── hooks/Hooks.java         # Before/After y evidencia ante fallos
│       │   ├── steps/                   # Step definitions por módulo
│       │   └── utils/DriverManager.java # Creación y cierre del WebDriver
│       └── resources/
│           ├── features/                # Escenarios Gherkin (.feature)
│           ├── allure.properties
│           └── junit-platform.properties
└── pom.xml
```

## Integración continua

El workflow `.github/workflows/maven-tests.yml` ("Automation Tests Pipeline") se dispara en:

- `push` a `main`, `dev`, `rama_integracion` y ramas `feature/**`
- `pull_request` hacia `main`, `dev` y `rama_integracion`
- ejecución manual (`workflow_dispatch`)

Pasos: checkout, configuración de JDK 21 (Temurin) con caché de Maven, `mvn clean test`, publicación del reporte JUnit en la interfaz de GitHub y subida de `target/allure-results` como artefacto (retención de 7 días).

Historial de ejecuciones: [pestaña Actions](https://github.com/jonalmlp/sQuAd1/actions).

## Notas

- Algunos escenarios (login, checkout) dependen de usuarios que ya existen en automationexercise.com.
- El escenario de registro usa un email fijo: si ese usuario ya fue creado en una ejecución anterior, hay que cambiarlo en el `.feature` antes de volver a correrlo.
- La ejecución en paralelo está deshabilitada (`cucumber.execution.parallel.enabled=false`).

## Contribuir

1. Hacé un fork del repositorio.
2. Creá una rama: `git checkout -b feature/nombre-de-la-mejora`
3. Commiteá tus cambios: `git commit -m "feat: descripción"`
4. Subí la rama: `git push origin feature/nombre-de-la-mejora`
5. Abrí un Pull Request.

## Autor

**jonalmlp** · [GitHub](https://github.com/jonalmlp)
