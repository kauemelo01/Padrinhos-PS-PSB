# 🎁 Sponsorships — Pastoral Social Jardim Colombo

A mobile app for registering **sponsors** (*padrinhos*) for the children served by
the Pastoral Social (a parish community charity), built with **Streamlit**. It reads
and writes the `cadastro.xlsx` spreadsheet hosted on **Google Drive**, which remains
the source of truth.

> **Spin-off of [Cadastro-PS-PSB](https://github.com/kauemelo01/Cadastro-PS-PSB).**
> This app is a spin-off of the Pastoral Social's main registry app: it uses the same
> spreadsheet, the same Drive service account and the same *Secrets* setup, but it
> does a single job — linking each child to a sponsor. Both apps can run at the same
> time, writing to the same file (see [Saving and data integrity](#-saving-and-data-integrity)).

The app's interface is in Portuguese; button labels, sheet names and column headers
are quoted below exactly as they appear in the app and in the spreadsheet.

---

## 🔍 Search

| Feature | Description |
|---|---|
| **Search by record / membership-card number** | Shows every child linked to that family |
| **Exact number match** | Leading zeros and decimals are ignored: "015", "15" and "15.0" find the same record |
| **Family summary** | Above the cards: number, how many children and how many already have a sponsor |

---

## 🧒 Child card

| Element | Description |
|---|---|
| **Name, prominent** | With sex and age just below |
| **Beneficiary** | Name of the person who receives the food basket, in smaller type |
| **Notes** | Highlighted in yellow, when present |
| **Sponsor** | Name and phone in green when registered; "Ainda sem padrinho" (no sponsor yet) otherwise |
| **Border colour** | Blue for boys, pink for girls |
| **Icon** | By sex and age group (table below) |

| Age group | Boy | Girl |
|---|---|---|
| 0 to 3 years | 👶 🧢 | 👶 🎀 |
| 4 to 8 years | 👦 | 👧 |
| 9 years or older | 🙋‍♂️ | 🙋‍♀️ |

Ages given in months ("3 meses") count as 0 years.

---

## ✏️ Register / edit a sponsor

| Feature | Description |
|---|---|
| **Popup per child** | **➕ Cadastrar Padrinho** button (or **✏️ Editar Padrinho** when there already is one) |
| **Required fields** | Sponsor and Phone; the phone needs at least 9 digits (area code optional) |
| **Automatic clean-up** | Extra spaces in the name are removed before saving |
| **Nothing changed, nothing saved** | If the values equal the current ones, the popup closes without going to Drive |
| **Confirmation** | A success message at the top once Drive confirms the save |

---

## 📊 Statistics

In the footer, always over the full list (not just the family searched for):

- **sponsored**, **% complete**, **total** and **how many are left**;
- a progress bar per age group (0 to 3, 4 to 8, 9 or older);
- a note on how many children have no age recorded — they count only in the total.

---

## 💾 Saving and data integrity

Every change goes to the `.xlsx` on Google Drive through a service account. Because
each save re-uploads the **whole file**, and the registry app writes to the same
file, the app uses these safeguards:

- **Serialized writes**: a single lock in the process covers the whole
  download → edit → upload cycle. Without it, two volunteers saving at the same time
  would have the slower upload silently erase the faster one.
- **Version check on Drive**: the file's version is checked just before uploading. If
  another process (for example, the registry app) wrote in the meantime, the cycle
  starts over with the new file, up to 3 attempts. If the file keeps changing, nothing
  is written and the user is asked to try again.
- **Optimistic concurrency per child**: the popup remembers the sponsor and phone it
  showed when it opened. If someone else changed that child while the form was open,
  the save is refused and the app shows who is registered now.
- **Content-based addressing**: the row is found by **number + child's name**, and
  the columns by header name, always in the freshly downloaded file. Reordering the
  spreadsheet or inserting columns won't send the sponsor to the wrong child.
- **Refusal when ambiguous**: if the child isn't found, or two children with the same
  name share a number, nothing is changed and the reason is shown.
- **Write before display**: the card shows the new sponsor only after Drive confirms,
  so it never shows as saved something that failed. The change appears immediately,
  even before the cache reloads the file.
- **Double timestamp**: each save records, in the columns `Salvo no App` and
  `Gravado na Planilha`, when the button was clicked and when it reached the
  spreadsheet; the gap reveals network or lock waits. Both columns are created
  automatically on the first save (with the header's style) and reused if they
  already exist.

---

## 🗂️ Spreadsheet layout

The app always uses the **`Lista Ordenada`** tab of `cadastro.xlsx` (other tabs,
including the registry app's, are ignored). The tab is found regardless of accents,
capitalization or extra spaces. Header in row 1:

| Header | Note |
|---|---|
| `Nº Família / Carteirinha` | required — family / membership-card number |
| `Beneficiário (Cesta)` | basket beneficiary |
| `Nome da criança` | required — child's name |
| `Sexo` | `Masculino` / `Feminino` |
| `Idade` | e.g. `6 anos`, `3 meses` |
| `Observações` | notes |
| `Padrinho` | sponsor — created automatically on the first save |
| `Telefone Padrinho` | sponsor's phone — created automatically on the first save |
| `Salvo no App` | timestamp of the click — created automatically on the first save |
| `Gravado na Planilha` | timestamp of the write — created automatically on the first save |

Notes:
- Column order doesn't matter; each column is found by its header name (ignoring
  accents and capitalization).
- Rows without a number or a child's name are ignored.
- Timestamps use the `dd/mm/yyyy hh:mm:ss` format, in Brasília time regardless of
  where the server runs.

---

## 🛠️ Technical notes

- Data is cached for 5 minutes; the cache is cleared after each save.
- The download handles the confirmation page Google Drive shows for large files.
- When looking for the child's row, the app reads only the filled cells, so it doesn't
  scan thousands of empty rows when the sheet has formatting down to the bottom.
- The write lock applies within one process. Across different processes (two apps, or
  several instances), the Drive version check is what protects the data.
- Mobile-first layout, with its own dark-mode palette.
