# Estructura de la base de datos de Koha (IAE)

Fuente: Carlos Lasso (IT), 2026-11-18, "estructura_bd_koha.pdf". Solo el esquema, sin datos reales.

## Catálogo
| Tabla | Clave | Campos principales | Qué es |
|---|---|---|---|
| biblio | biblionumber | title, author, notes | La obra |
| biblioitems | biblioitemnumber | biblionumber, isbn, publishercode, publicationyear, pages, itemtype | La edición |
| items | itemnumber | biblionumber, biblioitemnumber, barcode, itemcallnumber, homebranch, notforloan | Cada copia física (código de barras y ubicación) |

## Usuarios
| Tabla | Clave | Campos principales | Qué es |
|---|---|---|---|
| borrowers | borrowernumber | cardnumber, surname, firstname, email, categorycode, branchcode, dateenrolled, dateexpiry | Lectores (datos personales, carné) |
| categories | categorycode | description, enrolmentperiod | Tipos de usuario (Primaria, Secundaria, Docente) |

## Préstamos
| Tabla | Clave | Campos principales | Qué es |
|---|---|---|---|
| issues | issue_id | borrowernumber, itemnumber, issuedate, date_due, renewals_count | Préstamos activos |
| old_issues | issue_id | + returndate | Historial devuelto |
| reserves (opcional) | reserve_id | borrowernumber, biblionumber, reservedate, found | Reservas |

## Configuración
| Tabla | Clave | Campos | Qué es |
|---|---|---|---|
| itemtypes | itemtype | description | Tipo de material |
| branches | branchcode | branchname | Biblioteca o sede |

## Relaciones
biblio 1—N biblioitems 1—N items · borrowers N—1 categories · issues/old_issues: borrower + item · reserves: borrower + biblio.

## Para la app nueva
- Identificador interno de la copia = `items.itemnumber`. Lo que se escanea = `items.barcode` (Code 39/128), no el ISBN. `barcode` puede venir vacío: esas copias se importan igual y se marcan para etiquetar.
- Ubicación = `items.itemcallnumber` + `location` (estante, si está) + `holdingbranch` (sede donde está; `homebranch` es la dueña).
- Estado: no disponible si `notforloan`, `itemlost` o `withdrawn`; prestado si hay fila en `issues`. Reservas: `reserves.item_level_hold` dice si se pidió una copia concreta o el título; `reserves.itemnumber` es la copia pedida o asignada (puede estar vacío); solo `found` dice si ya está apartada o lista para retiro. Una reserva de título sin copia asignada no marca todas las copias. `found` dice si está pendiente, en tránsito, en proceso o esperando retiro.
- `borrowers` es dato protegido: no se importa hasta tener autorización del colegio.
- Pedir a IT confirmar que el export incluya `items.itemnumber`, `location`, `holdingbranch`, `itemlost`, `withdrawn`, `reserves.itemnumber`, `reserves.item_level_hold` y `reserves.found`.

## Lo que llegó en el export (2026-11-18)
Los originales están en la carpeta de Drive que compartió IT. Las copias locales del catálogo están en `datos-koha/` en la Mac de Marco, fuera de Git; nunca se suben. Los de usuarios y préstamos no se descargaron.
- `items`: itemnumber, biblionumber, biblioitemnumber, barcode, itemcallnumber, notforloan.
  - Casi todos los códigos son numéricos de 14 dígitos; hay algunos de otras longitudes y algunas copias sin código. No hay códigos repetidos.
  - `itemcallnumber` viene vacío en todas las copias: la ubicación no está ahí.
  - Faltan `location`, `holdingbranch`, `homebranch`, `itemlost`, `withdrawn`.
- `biblioitems`: la mayoría tiene ISBN; `itemtype` casi siempre vacío.
- `categories` salió igual que `itemtypes` (export equivocado).
- No vino `reserves` ni `branches`.
