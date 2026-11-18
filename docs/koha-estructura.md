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
- Copia = `items.barcode` (Code 39/128), no el ISBN.
- Ubicación = `items.itemcallnumber` + `homebranch`.
- Estado: `notforloan` + si hay fila en `issues` (prestado) o `reserves`.
- `borrowers` es dato protegido: no se importa hasta tener autorización del colegio.
